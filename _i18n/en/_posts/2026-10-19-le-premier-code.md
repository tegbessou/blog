---
layout: article
title: "The First Code, and What I Decided Not to Fix"
date:   2026-10-19 00:00:00
categories: agojie-logbook
lang: en
resume: "Second episode of the series on developing Agojie. The first two slices arrive: a game skeleton and a dice-rolling machine. An ADR also takes the hero out of the game. Plus two flaws spotted during review, given a deadline and postponed rather than fixed in a hurry."
permalink: /agojie-logbook/2026-10-19
---
**Agojie** automates **Dice Throne**, a turn-based fighting board game played with dice. In the previous episode, after three days, I still hadn’t written a single line of code. Four decisions had come out of those days, though: the shape of the model, the heroes’ defense, how the work was split into six slices, and the stack, in that order.

## A Skeleton That Walks

The work is split into six **vertical slices**: pieces of the game delivered one at a time, each of which works end to end. The first one can do almost nothing. Two players start at 50 health points, they take turns, and whoever drops to zero loses. Attacks don’t exist yet: damage is a fixed, hard-coded value that the real attack will replace later.

This bare minimum is deliberate. It’s the **Walking Skeleton** from the previous episode: the thinnest path that goes through the whole system and works. From there, each slice thickens that path instead of opening a new one.

Unlike the fixed damage, what this skeleton sets up is here to stay: the game and its identity, the two fighters, their health points. Those health points are not an integer. They are a **Value Object**, in *Eric Evans*’s sense: an object defined by its value rather than by an identity, and which protects its own rule. This one refuses to go below zero or above the starting maximum. Its code appears further down, with the first flaw spotted during review.

### Checking a Game Without an Interface

The previous episode raised the question without answering it in detail: how do you check all this when there is nothing on screen? The acceptance criteria of the skeleton are written as small games, with numbers:

- B is at 5 health points and the turn’s damage is 5. A plays. B drops to zero, the game ends, A wins.
- B is at 3 health points and the damage is 5. A plays. B is at zero, not at −2.
- The game is over. Someone tries to play one more turn. The action is refused.

These three sentences don’t describe code, they describe a game. That form is what makes it possible to deliver a slice without an interface: what gets checked is the game, not the implementation behind it.

The third scenario is worth a closer look. Refusing to play after the game is over adds nothing to a player’s experience, since nobody clicks on a dead board. It’s a rule for the model, not for the screen. It says that a finished game is a final state, not a state the rest of the code is kindly asked to leave alone.

## The Hero Doesn’t Live in the Game

Before touching the dice, one question was still open: where do a hero’s abilities live, along with their dice combinations and damage tables?

The answer became an **ADR**, an *Architecture Decision Record*: a short note that records an architecture decision, its context and its consequences at the time it is made. *Michael Nygard* defined the format. The previous episode announced its content; here it is.

The reasoning rests on one observation. A hero’s definition never changes during a game, whereas the state of that game changes with every hit taken.

The decision: the hero is immutable catalog data, **outside the game**, referred to by an identifier. All the game knows about its fighters is that identifier. When a rule needs the definition, the application loads it and passes it to the game as a parameter. This is what *Vaughn Vernon* recommends in *Effective Aggregate Design*: refer to what lives outside by its identifier, and pass in the data that is needed instead of letting the game fetch it.

![The boundary between definition and state](/assets/images/2026-10-19/frontiere-en.png)

Three options were ruled out. Hard-coding the abilities: the Barbarian would be a class with its own methods, and every new hero would require writing engine code. Locking the hero inside the game: every saved game would carry a copy of the definition, and rebalancing a hero would mean fixing all of them. Injecting the catalog into the game: the game would fetch its own data and depend on an outside source, which is exactly what **Hexagonal Architecture**, chosen in the previous episode, aims to avoid.

One point matters as much as the decision itself: the data carries its behavior. It is not an inert structure that something else interprets. It is the hero that knows which ability a dice combination triggers, and how much damage that ability deals. Without this, a third object would have had to do that work instead, and the game logic would have ended up far from the data it concerns.

The decision has a cost. Every time a rule needs the hero, the application has to load it and then hand it to the game. That coordination step would not exist if the hero were locked inside the game.

The catalog’s storage format, however, is still open. ***PHP*** objects, a file, a database: that is a question of wiring, not of modeling. For now, the heroes are ***PHP*** objects built by hand: they are data, not hero classes.

## A Dice-Rolling Machine

Next comes the offensive roll. Five dice, free rerolls, three rolls at most. No ability is triggered yet, no damage is dealt: all you see is the final result of the dice.

![The three rolls of a turn](/assets/images/2026-10-19/jets-en.png)

Two rules govern this roll. At most three rolls per turn. And between rolls, the player rerolls exactly the dice they want, including a die they kept on the previous roll. That second rule is why the kept dice can’t be represented as a fixed set.

On the application side, two **Commands** instead of one: roll and reroll. A **Command**, here, is the message that carries a player’s intent: “I roll my dice.” *Greg Young* made it the write side of CQRS, which separates the messages that change state from those that read it. It is received by an object that handles it, loads what it needs, and moves the game forward.

I could have written a single **Command**, with a “which dice to reroll” parameter left empty on the first roll. I preferred two distinct intents. The first roll leaves no choice: all five dice go. The reroll, on the other hand, is a choice, and in fact the only one in the offensive phase. Merging them into a single message would have forced every reader to work out which of the two situations they were looking at. Two messages, two player actions, two names. Only the reroll carries the positions of the dice:

{% highlight php linenos %}
final readonly class RollOffensiveDiceCommand
{
    public function __construct(
        public string $battleId,
    ) {}
}

final readonly class RerollOffensiveDiceCommand
{
    public function __construct(
        public string $battleId,
        public array $positions,
    ) {}
}
{% endhighlight %}

This also keeps the rules readable: “at most three rolls” is counted across both **Commands**, while “free rerolls” only concerns the second one.

Chance goes through the dice **Port** introduced in the previous episode. In this slice, that **Port** becomes concrete. In the game, one implementation draws random faces on the server. In tests, another returns exactly the faces the scenario imposes. Here is the object that handles the roll **Command**. It loads the game, asks the **Port** for five faces, hands them to the game, then saves it:

{% highlight php linenos %}
public function handle(RollOffensiveDiceCommand $command): void
{
    $battle = $this->repository->find(BattleId::fromString($command->battleId));
    $outcome = DiceOutcome::fromValues($this->diceRoller->roll(self::DICE_COUNT));
    $battle->rollOffense($outcome);
    $this->repository->save($battle);
}
{% endhighlight %}

Neither this object nor the game knows which of the two implementations answered.

### The Events I Don’t Write

A **Domain Event** is a past fact of the domain that the business considers important enough to be given a name, and that another component of the system can listen to. The previous episode set a rule about them: no **Domain Event** without a subscriber. The skeleton and the dice-rolling machine give that rule its first chance to apply.

There was no shortage of candidates. “The game is over” in the skeleton, “the dice were rolled” and “the dice were rerolled” in the dice-rolling machine. Three game facts, nameable, datable, exactly the expected shape.

I wrote none of them. Nobody listens to them yet: no interface to refresh, no statistics to feed, no game history. An event without a subscriber adds nothing to the model. All it leaves behind is an object to maintain, and a question for the next reader about who uses it.

They are written down on each slice as deferred candidates. When the interface arrives, in the second stage of the project, it will show which ones deserve to exist. This time, the answer won’t be an intuition: it will be a subscriber.

## The Two Flaws I Didn’t Fix Right Away

The reviews of these two slices caught two flaws. I fixed neither of them at the time.

The first one concerns health points. The object does check its bounds when it is created, but the operation that applies damage doesn’t go through that check again:

{% highlight php linenos %}
final readonly class HitPoints
{
    private const int MAX = 50;

    public static function fromValue(int $value): self
    {
        if ($value < 0 || $value > self::MAX) {
            throw new \InvalidArgumentException('Hit points out of bounds.');
        }

        return new self($value);
    }

    public function takeDamage(int $amount): self
    {
        return new self(max(0, $this->value - $amount));
    }
}
{% endhighlight %}

***takeDamage*** builds its result directly, without going through ***fromValue***. A negative amount would therefore push health points above 50. Nothing broke back then, because nothing added health points yet. The gap was bound to open when healing arrived. So I recorded the flaw on the slice for the defensive roll and healing, the first one that really exercises the upper bound.

The second one concerns the rolling phase. Nothing prevented replaying the roll **Command** in the middle of a turn. The roll count then started over at one, and the three-roll limit could be bypassed. I recorded it on the next slice, the one that reads the dice combination. The deadline is written there in black and white, but as a condition rather than a slice number: “as soon as the turn sequence includes the real offense.” That condition was met when damage arrived, and that is where the flaw was closed. As for the first flaw, it was closed in two steps, with damage and then with healing.

Postponing a known flaw is hard to defend. What makes it acceptable here is precisely what is usually missing: the flaw is recorded on a named slice, along with the reason for putting it there. It isn’t on a wish list, or only in my memory. And both were closed when their condition was met.

A third review comment, however, was fixed right away. To reroll, the player points to the dice by their position in the roll: “the first and the fourth.” The game accepted a list of positions and a list of new faces of different lengths, and produced a technical error further down instead of a clear refusal. That wasn’t a debt to schedule, it was wrong behavior.

## What This First Code Produced

At this point, the game still can’t be played: turns alternate, the dice roll, and nobody attacks yet. What is in place is a boundary between definition and state, and two game rules that hold on their own in the code.

In the next episode, the real heroes enter the code: the dice point to an ability, and the attack finally deals its damage. The previous episode showed how a detail found on a card corrected the specification. This time, it isn’t the specification that moves but the model: an **Invariant** had been living in the wrong place since the dice-rolling machine, and a hero sheet moved it.

Feel free to comment on this article, whether about the modeling or about the way I tell the project’s story.

See you soon for a new episode.
