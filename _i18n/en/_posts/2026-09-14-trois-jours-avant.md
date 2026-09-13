---
layout: article
title: "Three Days Before the First Line of Code"
date:   2026-09-14 00:00:00
categories: agojie-logbook
lang: en
resume: "First episode of the series on developing Agojie, which automates the Dice Throne board game. Three days without a single line of code, and four decisions: the shape of the model, the heroes’ defense, how the work was sliced, and the technical stack."
permalink: /agojie-logbook/2026-09-14
---
This article is about the beginning of the **Agojie** project, which automates the board game **Dice Throne**. It’s a turn-based fighting game between characters, played with dice. The Agojie were an all-female military regiment of the Fon people, in the Kingdom of Dahomey. Like all my project names, this one is linked to King Tegbessou, a figure I’m very fond of.

The first days of the project produced no code at all, only decisions. What interests me here is that none of them was made on intuition. This article follows four of them, in order: the shape of the model, the heroes’ defense, how the work was sliced, and finally the technical stack.

## What We’re Building

The first part of the development covers the core of the game in one-on-one play, and only what is played with the dice. The card layer will come later. In Dice Throne, rolling the dice lets you attack, defend, and trigger abilities. The cards are used to influence the dice or to upgrade the abilities.

The project starts with two heroes, the Barbarian and the Pyromancer. Their abilities are the simplest to model. That way I can start with the easiest part and increase the modeling difficulty as I go.

The project started on June 21 and has no planned end date. I organized it into four stages:

1. **The one-on-one core, without cards.**
2. **A playable interface for the core, and saving games.** At first, the core is developed without any interface. The interface only arrives at this stage.
3. **The card layer.** It will be easier to build once the foundations of the game are in place.
4. **Status effects.** They can prevent a hero from attacking, or make them impossible to target. They come last.

![Project roadmap](/assets/images/2026-09-14/roadmap-en.png)

For the first stage, I started by writing a specification: the game rules the core has to follow, with worked examples for each one.

## Dice Throne in a Few Rules

To follow the rest of the article, you only need four rules.

Each player starts the game with health points (HP). Whoever drops to zero loses.

On their turn, the active player rolls five dice, up to three times. Between rolls, they freely choose which dice to reroll.

The faces of the dice show symbols. A combination of symbols triggers one of the hero’s abilities, and the player can trigger only one per turn.

The attacked player answers with a defensive roll. Depending on the hero, this roll reduces the damage taken, heals the defender, or deals damage back to the attacker. That last case is called retaliation.

## Architecture Ideas

On the first day, before I had even sliced the work, three architecture ideas came to me.

The first: separate a hero’s **definition** from the **state** of a game. The Barbarian always has the same abilities, whatever the game. His health points, on the other hand, change with every hit he takes. The hero catalog never changes, while the game changes all the time.

The second: make the game, ***Match***, an **Aggregate**. *Eric Evans* defines an **Aggregate** as a group of objects that the code treats as a single unit whenever it changes data. It exists to protect **Invariants**: business rules that must hold at all times, even in the middle of a change. Every change goes through a single object, the **Aggregate Root**, which checks those rules before and after each operation.

The third: a list of candidate **Domain Events**, such as “the die was rolled,” “the ability was triggered,” or “the damage was resolved.” *Martin Fowler* gave the **Domain Event** its name. *Vaughn Vernon* (*Implementing Domain-Driven Design*) and *Mathias Verraes* describe it as a past fact of the domain that the business considers important enough to be given a name.

I applied none of them. I wrote them down so I could revisit them properly after the slicing. An intuition is not a reason. As long as no game rule justified an **Aggregate**, creating one would have meant deciding blindly.

On June 23, while preparing the first development slice, the **Aggregate** was challenged: should there be one **Aggregate** per player instead? It held, for two specific reasons. The rule “only one player plays at a time” and the rule “as soon as a player drops to zero, the game ends” both involve the two players at once. Neither of them holds if each player lives in their own **Aggregate**. That is why the whole game forms the **Aggregate**. It only changed its name: ***match*** is a reserved word in ***PHP***, so ***Match*** became ***Battle***.

The **Domain Events**, however, were not kept. No part of the system was listening to them yet, and I gave myself a rule: no **Domain Event** without a subscriber. A **Domain Event** nobody listens to only adds noise to the model.

### Future Outlook

On July 11, the separation between definition and state became an **ADR** (Architecture Decision Record). *Michael Nygard*, who set its format, describes it as a short note that records an architecture decision, its context, and its consequences at the moment it is made. What this ADR decides will be the subject of the next article.

The candidate **Domain Events** were noted on each slice, in case some part of the system needs them one day.

## Real Heroes, and a Card Detail That Corrects the Specification

On June 22, I decided to work with real heroes rather than invented ones. The price is stripping them of their status effects, which only arrive in the fourth stage. Until then, those effects are ignored.

I still had to find their numbers. They weren’t in the text of the game’s wiki, but on the card images. At that point, two things were certain: the dice faces, and the two heroes’ basic attacks. <u>Smack</u> for the Barbarian and <u>Fireball</u> for the Pyromancer are symmetrical: three, four, or five symbols deal four, six, or eight damage.

The real surprise came from the defense. I had written the specification assuming that a defense reduces the damage taken. Neither hero does that.

The Barbarian heals. His defense, <u>Thick Skin</u>, gives him back two HP per heart rolled, without dealing anything back to the attacker. The Pyromancer, on the other hand, retaliates without reducing anything. Her defense, <u>Magma Armor</u>, deals one damage per fire rolled. Even her defense is offensive.

Yet both defenses share the same calculation: one value per symbol. Only the nature of the effect changes, healing for one, damage for the other.

So I rewrote the definition of the defense in the specification, and added an edge case: a defense that reduces nothing.

## Slicing the Work, from Three to Six Slices

Once the rules were set, the work had to be split into **vertical slices**: pieces of the game small enough to be delivered one at a time, but each working end to end. After each slice, you can play a bit more of the game than before.

The first slicing, on June 22, had three slices: a game skeleton, the attack, and the defense. It didn’t last the day. The attack alone bundled the dice roll, the rerolls, reading the combination, and computing the damage. Four behaviors, each with its own test scenarios. That was too much for a single delivery, and the defense had the same problem. By the end of the day, there were six slices.

![The reworked slicing](/assets/images/2026-09-14/decoupage-en.png)

Here is what you can do after each of the six slices:

1. **Game skeleton.** Two players at 50 HP take turns and deal fixed damage to each other until one of them drops to zero. There are no dice yet. This is what *Alistair Cockburn* calls a **Walking Skeleton** in *Crystal Clear*: the thinnest version of the system that works end to end.
2. **Roll and rerolls.** The player rolls five dice, then rerolls whichever ones they want, up to twice.
3. **Reading the combination.** The dice point to an ability. For the Barbarian, faces 1, 2, and 3 are swords: a roll of [1, 2, 3, 4, 6] gives three swords, hence <u>Smack</u>.
4. **Raw damage.** <u>Smack</u> with three swords deals 4 damage: the opponent goes from 50 to 46 HP.
5. **Defensive roll and healing.** The attacked Barbarian rolls <u>Thick Skin</u> and heals. If he takes 6 damage and heals 2, he ends at 46 HP. This slice was meant to be called “mitigation”: it was the real heroes that renamed it.
6. **Retaliation and ordered resolution.** The attacked Pyromancer deals damage back to her opponent.

The sixth slice also involves a decision that goes against the official rules. The rulebook adds up all the damage of a turn before applying it, so both players can drop to zero at the same time. I chose to apply the damage one effect at a time: first the attack, then the retaliation, and defeat is checked after each effect.

The scenario that shows it is simple. The Pyromancer is at 6 HP, her opponent deals 6 damage to her, and her retaliation would be worth 50. She drops to zero, the game ends, and the retaliation never happens. This is the rule “as soon as a player drops to zero, the game ends” again, the very rule that made the game an **Aggregate**.

This choice has a cost. In this specific case, the board game and the software no longer give the same result. I accepted it, and I wrote it in the specification: “Deliberate divergence from the official rules, which add up the damage and then apply it.”

One question remains: how do you check all this without an interface? Each slice is checked by scenarios written like a small game: “B is at 50 HP, A rolls [1, 2, 3, 4, 6], then B drops to 46.” The test plays the game in the player’s place.

But a test must always give the same result, and the dice result depends on chance. So in the scenario, the dice return exactly [1, 2, 3, 4, 6]. The application asks for a dice result without knowing where it comes from. In the game, chance provides it. In a test, the scenario imposes it. This point of contact is called a **Port**. *Alistair Cockburn* defines it as an interface through which the application talks to the outside world.

## The Stack, Chosen Last

The choice of technology came after the slicing, on purpose. Slicing by observable behaviors doesn’t depend on any stack. However, the backend had to be chosen before preparing the first slice.

I started with the interface question: a rich client, or server-side rendering? That question had to decide the backend, not the other way around.

This is where **Hexagonal Architecture** comes in. *Alistair Cockburn*, who also calls it Ports & Adapters, defines it as a way of separating the application from everything around it: interface, database, external services. The dice **Port** we saw earlier is one of those points of separation. The intent of this architecture is to let the application run without any interface or database. So the choice of interface doesn’t touch the domain. I could make it based on polish and developer experience, with no architectural constraint.

Before deciding, I checked that HTML was enough for a game like this one. 3D dice in CSS are a solved problem. And there is a demo of a real-time interactive card game built with ***Symfony UX***, the *Live Memory Card Game*.

On June 23, I chose server-side rendering with ***Symfony UX***, and a ***PHP*** backend with ***Symfony***. This decision is recorded in ADR 001. Thanks to **Hexagonal Architecture**, the door stays open to a rich client if server-side rendering is no longer enough.

The project was set up right after: ***Symfony*** 8.1, ***PHP*** 8.4, ***FrankenPHP***, and ***MariaDB***. ***Doctrine*** is installed but unused in the core, since saving games only arrives in the second stage.

## What These Three Days Produced

Three days, no code, and four decisions. The shape of the model was decided by two game rules, not by an intuition. The heroes’ defense was corrected by a detail found on a card. The slicing was reworked because two slices were too heavy. The stack was chosen based on the interface.

In the next episode, the first code arrives with the first two slices, and I’ll explain what the ADR on separating the heroes from the game decides. I’ll also talk about two flaws spotted during review, which I chose to push to later slices rather than fix in a hurry.

Feel free to comment on this article, whether about the modeling or about the way I tell the project’s story.

See you soon for a new episode.
