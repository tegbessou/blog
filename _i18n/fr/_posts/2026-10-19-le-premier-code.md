---
layout: article
title: "Le premier code, et ce qu’on décide de ne pas corriger"
date:   2026-10-19 00:00:00
categories: agojie-logbook
lang: fr
resume: "Deuxième épisode de la série sur le développement d’Agojie. Les deux premières tranches arrivent : un squelette de partie et une machine à jets. Un ADR sort aussi le héros de la partie. Avec deux défauts repérés en relecture, datés et reportés plutôt que corrigés dans l’urgence."
permalink: /agojie-logbook/2026-10-19
---
**Agojie** est l’automatisation de **Dice Throne**, un jeu de société de combat au tour par tour qui se joue avec des dés. Dans l’épisode précédent, au bout de trois jours, je n’avais toujours pas écrit une ligne de code. Quatre décisions en étaient pourtant sorties : la forme du modèle, la défense des héros, le découpage en six tranches et la stack, dans cet ordre.

## Un squelette qui marche

Le travail est découpé en six **tranches verticales** : des morceaux du jeu livrés un par un, qui fonctionnent chacun de bout en bout. La première ne sait presque rien faire. Deux joueurs commencent à 50 points de vie, ils jouent chacun leur tour, et celui qui tombe à zéro perd. L’attaque n’existe pas encore : les dégâts sont une valeur fixe, écrite en dur, que l’attaque réelle remplacera plus tard.

Ce presque-rien est volontaire. C’est le **Walking Skeleton** vu dans l’épisode précédent : le chemin le plus mince qui traverse tout le système et qui fonctionne. À partir de là, chaque tranche épaissit ce chemin au lieu d’en ouvrir un nouveau.

Ce que ce squelette pose, contrairement aux dégâts fixes, va rester : la partie et son identité, les deux combattants, leurs points de vie. Ces points de vie ne sont pas un entier. C’est un **Value Object**, au sens d’*Eric Evans* : un objet défini par sa valeur et non par une identité, qui protège sa propre règle. Celui-ci refuse de descendre sous zéro et de dépasser le maximum de départ. Son code apparaît plus bas, avec le premier défaut repéré en relecture.

### Vérifier une partie sans interface

L’épisode précédent posait la question sans y répondre en détail : comment vérifie-t-on tout ça, alors qu’il n’y a rien à l’écran ? Les critères d’acceptation du squelette sont écrits comme de petites parties, avec des chiffres :

- B est à 5 points de vie et les dégâts du tour valent 5. A joue. B tombe à zéro, la partie s’arrête, A est vainqueur.
- B est à 3 points de vie et les dégâts valent 5. A joue. B est à zéro, pas à −2.
- La partie est terminée. On tente d’enchaîner un tour de plus. L’action est refusée.

Ces trois phrases ne décrivent pas du code, elles décrivent une partie. C’est cette forme-là qui permet de livrer une tranche sans interface : ce qui est vérifié, c’est le jeu, pas l’implémentation qui le porte.

Le troisième scénario mérite un mot. Refuser de jouer après la fin de partie n’ajoute rien à l’expérience d’un joueur, puisque personne ne clique sur un plateau mort. C’est une règle pour le modèle, pas pour l’écran. Elle dit que la partie terminée est un état définitif, et non un état que le reste du code est prié de ne pas déranger.

## Le héros ne vit pas dans la partie

Avant de toucher aux dés, une question restait ouverte : où vivent les capacités d’un héros, ses combinaisons de dés, ses tables de dégâts ?

La réponse est devenue un **ADR**, un *Architecture Decision Record* : une note courte qui consigne une décision d’architecture, son contexte et ses conséquences au moment où elle est prise. C’est *Michael Nygard* qui en a fixé le format. L’épisode précédent en annonçait le contenu ; le voici.

Le raisonnement tient en une observation. La définition d’un héros ne change jamais pendant une partie, alors que l’état de cette partie change à chaque coup reçu.

La décision : le héros est une donnée de catalogue immuable, **hors de la partie**, désignée par un identifiant. La partie ne connaît de ses combattants que cet identifiant. Quand une règle a besoin de la définition, c’est l’application qui la charge et la passe en paramètre à la partie. C’est ce que recommande *Vaughn Vernon* dans *Effective Aggregate Design* : désigner par son identifiant ce qui vit à l’extérieur, et passer en paramètre la donnée nécessaire au lieu de laisser la partie aller la chercher.

![La frontière entre la définition et l’état](/assets/images/2026-10-19/frontiere.png)

Trois options ont été écartées. Coder les capacités en dur : le Barbare serait une classe avec ses propres méthodes, et chaque nouveau héros demanderait d’écrire du code de moteur. Enfermer le héros dans la partie : chaque partie sauvegardée embarquerait une copie de la définition, et rééquilibrer un héros obligerait à les corriger toutes. Injecter le catalogue dans la partie : la partie irait chercher elle-même sa donnée et dépendrait d’une source extérieure, ce que l’**Architecture Hexagonale**, choisie dans l’épisode précédent, cherche justement à éviter.

Un point compte autant que la décision elle-même : la donnée porte son comportement. Ce n’est pas une structure inerte que quelqu’un d’autre interprète. C’est le héros qui sait quelle capacité une combinaison de dés active, et combien cette capacité inflige. Sans cela, il aurait fallu un objet tiers pour faire ce travail à sa place, et la logique du jeu se serait retrouvée loin de la donnée qu’elle concerne.

La décision a un coût. Chaque fois qu’une règle a besoin du héros, l’application doit le charger puis le transmettre à la partie. C’est une étape de coordination qu’un héros enfermé dans la partie aurait évitée.

Le format de stockage du catalogue, lui, n’est pas décidé. Objets ***PHP***, fichier, base de données : c’est un choix de branchement, pas de modèle. Aujourd’hui, les héros sont des objets ***PHP*** construits à la main : ce sont des données, pas des classes de héros.

## Une machine à jets

Vient ensuite le lancer offensif. Cinq dés, relance libre, trois jets au maximum. Aucune capacité n’est encore activée, aucun dégât n’est infligé : on ne voit que le résultat final des dés.

![Les trois jets d’un tour](/assets/images/2026-10-19/jets.png)

Deux règles encadrent ce lancer. Au plus trois jets par tour. Et entre deux jets, le joueur relance exactement les dés qu’il veut, y compris un dé qu’il avait gardé au jet précédent. C’est cette seconde règle qui interdit de représenter les dés gardés comme un ensemble figé.

Côté application, deux **Command** plutôt qu’une : lancer et relancer. Une **Command**, ici, est le message qui porte une intention du joueur : « je lance mes dés ». *Greg Young* en a fait le versant écriture de CQRS, qui sépare les messages qui modifient l’état de ceux qui le lisent. Elle est reçue par un objet qui la traite, charge ce qu’il faut, et fait avancer la partie.

On aurait pu n’en écrire qu’une, avec un paramètre « quels dés relancer », vide au premier jet. J’ai préféré deux intentions distinctes. Le premier jet ne laisse aucun choix : les cinq dés partent. La relance, elle, est un choix, et c’est même le seul de la phase offensive. Les fondre dans un même message aurait obligé, à chaque lecture, à identifier laquelle des deux situations on regarde. Deux messages, deux gestes du joueur, deux noms. Seule la relance porte les positions des dés :

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

C’est aussi ce qui rend la suite lisible côté règles : « au plus trois jets » se compte sur les deux **Command** réunies, alors que « relance libre » ne concerne que la seconde.

Le hasard, lui, passe par le **Port** des dés, présenté dans l’épisode précédent. Ici, ce **Port** devient concret. En jeu, une implémentation tire des faces au hasard, côté serveur. En test, une autre rend exactement les faces que le scénario impose. Voici l’objet qui traite la **Command** de lancer. Il charge la partie, demande cinq faces au **Port**, les transmet à la partie, puis l’enregistre :

{% highlight php linenos %}
public function handle(RollOffensiveDiceCommand $command): void
{
    $battle = $this->repository->find(BattleId::fromString($command->battleId));
    $outcome = DiceOutcome::fromValues($this->diceRoller->roll(self::DICE_COUNT));
    $battle->rollOffense($outcome);
    $this->repository->save($battle);
}
{% endhighlight %}

Ni cet objet ni la partie ne savent laquelle des deux implémentations a répondu.

### Les événements qu’on n’écrit pas

Un **Domain Event** est un fait passé du domaine, que le métier juge assez important pour lui donner un nom, et qu’un autre composant du système peut écouter. L’épisode précédent posait une règle à son sujet : pas de **Domain Event** sans abonné. Le squelette et la machine à jets lui donnent sa première occasion de s’appliquer.

Les candidats ne manquaient pas. « La partie est terminée » dans le squelette, « les dés ont été lancés » et « les dés ont été relancés » dans la machine à jets. Trois faits du jeu, nommables, datables, exactement la forme attendue.

Je n’en ai écrit aucun. Personne ne les écoute encore : pas d’interface à rafraîchir, pas de statistiques à alimenter, pas d’historique de partie. Un événement sans abonné n’ajoute rien au modèle : il laisse seulement un objet à maintenir et une question pour le prochain lecteur, celle de savoir qui s’en sert.

Ils sont notés sur chaque tranche, comme candidats différés. Le jour où l’interface arrivera, à la deuxième séquence du projet, elle dira lesquels méritent d’exister. Cette fois-là, la réponse ne sera pas une intuition : ce sera un abonné.

## Les deux défauts que je n’ai pas corrigés tout de suite

Les relectures de ces deux tranches ont attrapé deux défauts. Je n’ai corrigé ni l’un ni l’autre sur le moment.

Le premier touche les points de vie. L’objet valide bien ses bornes à la construction, mais l’opération qui applique les dégâts, elle, ne repasse pas par cette validation :

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

***takeDamage*** construit son résultat directement, sans passer par ***fromValue***. Un montant négatif ferait donc dépasser les 50 points de vie. Rien ne cassait alors, parce que rien n’ajoutait encore de points de vie. Le trou devait s’ouvrir avec l’arrivée du soin. J’ai donc noté le défaut sur la tranche du jet défensif et du soin, la première qui exerce vraiment la borne haute.

Le second touche la phase de jet. Rien n’empêchait de rejouer la **Command** de lancer au milieu d’un tour. Elle repartait alors d’un compteur à un, et la limite de trois jets devenait contournable. Je l’ai noté sur la tranche suivante, celle de la lecture de la combinaison. L’échéance y est écrite noir sur blanc, mais sous la forme d’une condition plutôt que d’un numéro de tranche : « dès que la séquence de tour intègre l’offensive réelle ». Cette condition s’est réalisée avec l’arrivée des dégâts, et c’est là que le défaut a été fermé. Le premier défaut, lui, a été fermé en deux temps, avec les dégâts puis avec le soin.

En général, reporter un défaut connu se défend mal. Ce qui le rend acceptable ici, c’est précisément ce qui manque d’habitude : le défaut est noté sur une tranche nommée, avec la raison de l’y mettre. Il n’est ni dans une liste de vœux, ni dans ma seule mémoire. Les deux ont effectivement été fermés quand leur condition s’est réalisée.

Une troisième remarque de relecture, elle, a été corrigée tout de suite. Pour relancer, le joueur désigne les dés par leur place dans le jet : « le premier et le quatrième ». La partie acceptait une liste de places et une liste de nouvelles faces qui n’avaient pas la même taille, et produisait plus loin une erreur technique au lieu d’un refus clair. Ce n’était pas une dette à dater, c’était un comportement faux.

## Ce que ce premier code a produit

Le jeu, à ce stade, ne se joue toujours pas : des tours alternent, des dés tombent, et personne n’attaque encore. Ce qui est posé, c’est une frontière entre la définition et l’état, et deux règles du jeu qui tiennent toutes seules dans le code.

Dans le prochain épisode, les héros réels entrent dans le code : les dés désignent une capacité, et l’attaque inflige enfin ses dégâts. L’épisode précédent racontait comment une donnée lue sur une carte avait corrigé la spécification. Cette fois, ce n’est plus la spécification qui bouge, c’est le modèle : un **Invariant** vivait au mauvais endroit depuis la machine à jets, et une fiche de jeu l’a déplacé.

N’hésitez pas à commenter cet article, que ce soit sur la modélisation ou sur la façon de raconter le projet.

À très bientôt pour un nouvel épisode.
