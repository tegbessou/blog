---
layout: article
title: "Trois jours avant la première ligne de code"
date:   2026-09-14 00:00:00
categories: agojie-logbook
lang: fr
resume: "Premier épisode de la série sur le développement d’Agojie, l’automatisation du jeu Dice Throne. Trois jours sans une ligne de code, et quatre décisions : la forme du modèle, la défense des héros, le découpage du travail et la stack technique."
permalink: /agojie-logbook/2026-09-14
---
Cet article parle du début du projet **Agojie**, l’automatisation du jeu de société **Dice Throne**. C’est un jeu de combat entre personnages, au tour par tour, qui se joue avec des dés. Les Agojie étaient un régiment militaire entièrement féminin du peuple Fon, dans le royaume du Dahomey. Comme tous les noms de mes projets, celui-ci est lié au roi Tegbessou, un personnage pour qui j’ai beaucoup d’affection.

Les premiers jours du projet n’ont produit aucune ligne de code, seulement des décisions. Ce qui m’intéresse ici, c’est qu’aucune n’a été prise sur une intuition. Cet article en suit quatre, dans l’ordre : la forme du modèle, la défense des héros, le découpage du travail, et enfin la stack technique.

## Ce qu’on construit

La première partie du développement porte sur le noyau du jeu en 1 contre 1, et seulement sur ce qui se joue avec les dés. La couche des cartes viendra plus tard. Dans Dice Throne, le lancer de dés permet d’attaquer, de se défendre et d’activer des capacités. Les cartes, elles, servent à influencer les dés ou à faire évoluer les capacités.

Le projet démarre avec deux héros, le Barbare et la Pyromancienne. Ce sont ceux dont les capacités sont les plus simples à modéliser. Je pourrai ainsi commencer par le plus facile et augmenter la difficulté de la modélisation au fur et à mesure.

Le projet a commencé le 21 juin et n’a pas de date de fin prévue. Je l’ai organisé en quatre séquences :

1. **Le noyau en 1 contre 1, sans cartes.**
2. **Une interface jouable du noyau, et la sauvegarde des parties.** Au début, le noyau est développé sans aucune interface. Elle arrive seulement à cette étape.
3. **La couche des cartes.** Elle sera plus facile à développer une fois les bases du jeu posées.
4. **Les effets de statut.** Ils peuvent empêcher un héros d’attaquer, ou le rendre impossible à cibler. Ils arrivent en dernier.

![Roadmap du projet](/assets/images/2026-09-14/roadmap.png)

Pour la première séquence, j’ai commencé par écrire une spécification : les règles du jeu que le noyau doit respecter, avec des exemples chiffrés pour chacune.

## Dice Throne en quelques règles

Pour suivre la suite, il suffit de connaître quatre règles.

Chaque joueur commence la partie avec des points de vie (PV). Celui qui tombe à zéro perd.

À son tour, le joueur actif lance cinq dés, jusqu’à trois fois. Entre deux jets, il choisit librement les dés qu’il relance.

Les faces des dés portent des symboles. Une combinaison de symboles active une capacité du héros, et le joueur n’en active qu’une par tour.

Le joueur attaqué répond par un jet défensif. Selon le héros, ce jet réduit les dégâts reçus, soigne le défenseur, ou renvoie des dégâts à l’attaquant. Ce dernier cas s’appelle la riposte.

## Idées d’architecture

Le premier jour, avant même d’avoir découpé le sujet, trois idées d’architecture me sont venues.

La première : séparer la **définition** d’un héros de l’**état** d’une partie. Le Barbare a toujours les mêmes capacités, quelle que soit la partie. Ses points de vie, eux, changent à chaque coup reçu. Le catalogue des héros ne bouge pas, la partie bouge sans arrêt.

La deuxième : faire de la partie, ***Match***, un **Aggregate**. *Eric Evans* définit l’**Aggregate** comme un groupe d’objets que le code traite comme un tout chaque fois qu’il modifie des données. Il existe pour protéger des **Invariants** : des règles métier qui doivent rester vraies à tout moment, même pendant une modification. Toute modification passe par un seul objet, l’**Aggregate Root**, qui vérifie ces règles avant et après chaque opération.

La troisième : une liste de **Domain Events** candidats, comme « le dé a été lancé », « la capacité a été activée » ou « les dégâts ont été résolus ». *Martin Fowler* a donné son nom au **Domain Event**. *Vaughn Vernon* (*Implementing Domain-Driven Design*) et *Mathias Verraes* le décrivent comme un fait passé du domaine, que le métier juge assez important pour lui donner un nom.

Je n’en ai appliqué aucune. Je les ai notées pour les reprendre proprement après le découpage. Une intuition n’est pas une raison. Tant qu’aucune règle du jeu ne justifiait un **Aggregate**, en poser un aurait été décider à l’aveugle.

Le 23 juin, au moment de préparer la première tranche de développement, l’**Aggregate** a été remis en question : fallait-il plutôt un **Aggregate** par joueur ? Il a tenu, pour deux raisons précises. La règle « un seul joueur joue à la fois » et la règle « dès qu’un joueur tombe à zéro, la partie s’arrête » portent sur les deux joueurs en même temps. Aucune des deux ne tient si chaque joueur vit dans son propre **Aggregate**. C’est pour cela que la partie entière forme l’**Aggregate**. Il a seulement changé de nom : ***match*** est un mot réservé de ***PHP***, et ***Match*** est devenu ***Battle***.

Les **Domain Events**, eux, n’ont pas été retenus. Aucune partie du système ne les écoutait encore, et je me suis donné une règle : pas de **Domain Event** sans abonné. Un **Domain Event** que personne n’écoute ne fait qu’ajouter du bruit au modèle.

### Vision du futur

Le 11 juillet, la séparation entre définition et état est devenue un **ADR** (Architecture Decision Record). *Michael Nygard*, qui en a fixé le format, le décrit comme une note courte qui consigne une décision d’architecture, son contexte et ses conséquences au moment où elle est prise. Ce que décide cet ADR sera le sujet du prochain article.

Les **Domain Events** candidats ont été notés sur chaque tranche, pour le jour où une partie du système en aura besoin.

## Des héros réels, et une donnée qui corrige la spécification

Le 22 juin, j’ai décidé de travailler avec des héros réels plutôt qu’avec des héros inventés. Le prix à payer est de les amputer de leurs effets de statut, qui n’arrivent qu’à la quatrième séquence. D’ici là, ces effets sont ignorés.

Encore fallait-il trouver leurs chiffres. Ils n’étaient pas dans le texte du wiki du jeu, mais sur les images des cartes. À ce stade, deux choses étaient sûres : les faces des dés, et les attaques de base des deux héros. <u>Frappe</u> pour le Barbare et <u>Boule de Feu</u> pour la Pyromancienne sont symétriques : trois, quatre ou cinq symboles infligent quatre, six ou huit dégâts.

La vraie surprise est venue de la défense. J’avais écrit la spécification en pensant qu’une défense réduit les dégâts reçus. Aucun des deux héros ne fait ça.

Le Barbare se soigne. Sa défense, <u>Peau Dure</u>, lui rend deux PV par cœur obtenu, sans rien renvoyer à l’attaquant. La Pyromancienne, elle, riposte sans rien réduire. Sa défense, <u>Armure en Fusion</u>, inflige un dégât par flamme obtenue. Même sa défense est offensive.

Les deux défenses ont pourtant la même forme de calcul : une valeur par symbole. Seule la nature de l’effet change, un soin d’un côté, des dégâts de l’autre.

J’ai donc réécrit la définition de la défense dans la spécification, et ajouté un cas limite : une défense qui ne réduit rien.

## Le découpage, de trois à six tranches

Une fois les règles posées, il fallait découper le travail en **tranches verticales** : des morceaux du jeu assez petits pour être livrés un par un, mais qui fonctionnent chacun de bout en bout. Après chaque tranche, on peut en jouer un peu plus qu’avant.

Le premier découpage, le 22 juin, comptait trois tranches : un squelette de partie, l’attaque et la défense. Il n’a pas tenu la journée. L’attaque, à elle seule, regroupait le lancer des dés, les relances, la lecture de la combinaison et le calcul des dégâts. Quatre comportements, chacun avec ses propres scénarios de test. C’était trop pour une seule livraison, et la défense avait le même problème. Le jour même, le découpage en comptait six.

![Le découpage refait](/assets/images/2026-09-14/decoupage.png)

Voici ce qu’on peut faire après chacune des six tranches :

1. **Squelette de partie.** Deux joueurs à 50 PV jouent chacun leur tour et s’infligent des dégâts fixes, jusqu’à ce que l’un d’eux tombe à zéro. Il n’y a pas encore de dés. C’est ce qu’*Alistair Cockburn* appelle un **Walking Skeleton** dans *Crystal Clear* : la version la plus mince du système qui fonctionne de bout en bout.
2. **Lancer et relances.** Le joueur lance cinq dés, puis relance jusqu’à deux fois ceux qu’il veut.
3. **Lecture de la combinaison.** Les dés désignent une capacité. Chez le Barbare, les faces 1, 2 et 3 sont des épées : un jet de [1, 2, 3, 4, 6] donne trois épées, donc <u>Frappe</u>.
4. **Dégâts bruts.** <u>Frappe</u> à trois épées inflige 4 dégâts : l’adversaire passe de 50 à 46 PV.
5. **Jet défensif et soin.** Le Barbare attaqué lance <u>Peau Dure</u> et se soigne. S’il encaisse 6 dégâts et se soigne de 2, il termine à 46 PV. Cette tranche devait s’appeler « atténuation » : ce sont les héros réels qui l’ont renommée.
6. **Riposte et application ordonnée.** La Pyromancienne attaquée renvoie des dégâts à son adversaire.

La sixième tranche porte aussi une décision prise contre le règlement officiel. Le règlement cumule tous les dégâts d’un tour avant de les appliquer, si bien que les deux joueurs peuvent tomber à zéro en même temps. J’ai choisi d’appliquer les dégâts un par un : d’abord l’attaque, ensuite la riposte, et la défaite est vérifiée après chaque effet.

Le scénario qui le montre est simple. La Pyromancienne est à 6 PV, son adversaire lui en inflige 6, et sa riposte vaudrait 50. Elle tombe à zéro, la partie s’arrête, et la riposte n’a jamais lieu. On retrouve la règle « dès qu’un joueur tombe à zéro, la partie s’arrête », celle qui a fait de la partie un **Aggregate**.

Ce choix a un coût. Sur ce cas précis, le jeu de société et le logiciel ne donnent plus le même résultat. Je l’ai assumé, et je l’ai écrit dans la spécification : « Divergence assumée avec le règlement officiel, qui cumule les dégâts puis les applique. »

Reste une question : comment vérifier tout ça sans interface ? Chaque tranche est vérifiée par des scénarios écrits comme une petite partie : « B est à 50 PV, A obtient [1, 2, 3, 4, 6], alors B passe à 46. » Le test joue la partie à la place du joueur.

Mais un test doit toujours donner le même résultat, et le résultat des dés dépend du hasard. Dans le scénario, les dés rendent donc exactement [1, 2, 3, 4, 6]. L’application demande un résultat de dés sans savoir d’où il vient. En jeu, c’est le hasard qui le fournit. En test, c’est le scénario qui l’impose. Ce point de passage s’appelle un **Port**. *Alistair Cockburn* le définit comme une interface par laquelle l’application parle au monde extérieur.

## La stack choisie en dernier

Le choix de la technologie est arrivé après le découpage, et c’était voulu. Un découpage en comportements observables ne dépend d’aucune stack. En revanche, le backend devait être choisi avant de préparer la première tranche.

Je suis parti de la question de l’interface : un client riche, ou un rendu côté serveur ? C’est elle qui devait décider du backend, et pas l’inverse.

C’est là que l’**Architecture Hexagonale** entre en jeu. *Alistair Cockburn*, qui l’appelle aussi Ports & Adapters, la définit comme une façon de séparer l’application de tout ce qui l’entoure : interface, base de données, services externes. Le **Port** des dés vu plus haut est l’un de ces points de séparation. L’intention de cette architecture est de pouvoir faire tourner l’application sans interface ni base de données. Le choix de l’interface ne touche donc pas au domaine. Je pouvais le faire sur des critères de finition et de confort de développement, sans contrainte d’architecture.

Avant de décider, j’ai vérifié que du HTML suffisait pour un jeu comme celui-ci. Des dés 3D en CSS, c’est un problème déjà résolu. Et il existe une démonstration d’un jeu de cartes interactif en temps réel avec ***Symfony UX***, le *Live Memory Card Game*.

Le 23 juin, j’ai choisi un rendu côté serveur avec ***Symfony UX***, et un backend en ***PHP*** avec ***Symfony***. Cette décision est consignée dans l’ADR 001. Grâce à l’**Architecture Hexagonale**, la porte reste ouverte vers un client riche si le rendu serveur ne suffit plus.

Le projet a été posé dans la foulée : ***Symfony*** 8.1, ***PHP*** 8.4, ***FrankenPHP*** et ***MariaDB***. ***Doctrine*** est installé mais inutilisé dans le noyau, puisque la sauvegarde des parties n’arrive qu’à la deuxième séquence.

## Ce que ces trois jours ont produit

Trois jours, aucune ligne de code, et quatre décisions. La forme du modèle a été décidée par deux règles du jeu, pas par une intuition. La défense des héros a été corrigée par une donnée lue sur une carte. Le découpage a été refait parce que deux tranches étaient trop lourdes. La stack a été choisie en fonction de l’interface.

Dans le prochain épisode, le premier code arrive avec les deux premières tranches, et je vous expliquerai ce que décide l’ADR sur la séparation entre les héros et la partie. Je vous parlerai aussi de deux défauts repérés en relecture, que j’ai choisi de reporter sur les tranches suivantes plutôt que de les corriger dans l’urgence.

N’hésitez pas à commenter cet article, que ce soit sur la modélisation ou sur la façon de raconter le projet.

À très bientôt pour un nouvel épisode.
