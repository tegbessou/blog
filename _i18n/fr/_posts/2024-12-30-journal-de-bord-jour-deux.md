---
layout: article
title: "Journal de bord de l’apprentissage du Domain-Driven Design : Jour 2"
date:   2024-12-26 14:00:00
categories: ddd-logbook
lang: fr
resume: "Cet article examine la pagination dans un contexte d'architecture logicielle, en mettant l'accent sur l’approche DDD et CQRS. Il montre comment séparer les préoccupations techniques et métiers pour préserver l'intégrité du domaine."
permalink: /ddd-logbook/2024-12-27
---

Dans mon précédent article du journal de bord, j’ai introduit les concepts de base du **Domain-Driven Design (DDD)**, et en particulier l’importance de l’_Ubiquitous Langage_. Aujourd’hui, j’expose la mise en place et la structure de mon projet.

## Mise en place du projet

J’ai démarré le développement avec ***Symfony***, mon framework ***PHP*** préféré. Il facilite vraiment la création d’applications web complexes grâce à son ensemble de bibliothèques modulaires, tout en encourageant les bonnes pratiques de développement.

Pour l’environnement de travail, j’ai mis en place un conteneur ***Docker*** avec ***MariaDB***, ***Nginx***, et un proxy pour gérer le ***HTTPS***, garantissant un déploiement local fluide.

Le plus gros défi au lancement d’un nouveau projet est l’absence de revue externe. Après des heures et des heures passées sur la même fonctionnalité, il devient difficile de discerner les points faibles. Pour garantir le respect des règles de développement que je me suis fixées, j’ai intégré des outils essentiels.

### Quatre outils pour respecter les standards

1. ***PHP CS Fixer*** : garantit que mon code est conforme aux standards de style et de bonnes pratiques d’écriture  
2. ***PHPStan*** : analyse le code pour détecter les bugs, les oublis de typage et d’autres erreurs potentielles  
3. ***Rector*** : un puissant outil pour refactoriser le code et gérer facilement les montées de versions de PHP et de Symfony  
4. ***Deptrac*** : crucial pour vérifier que je respecte les principes de l’architecture hexagonale ou ceux des _Bounded Context_.

## Structure et architecture du projet

Pour ce projet, en plus du **DDD**, je souhaite partir sur une **architecture hexagonale** combinée au pattern **CQRS**, afin de créer une application adaptable et aisément testable. Cela facilite en plus l’évolution de la base de code.

### Architecture Hexagonale

L’architecture est organisée autour de deux zones principales : l’intérieur (la logique métier) et l’extérieur (les interfaces).

L’extérieur se compose des _Ports_ et d’_Adapter_, qui gèrent l’interaction avec les clients en transformant les requêtes en actions compréhensibles pour l’interface. Ces Ports permettent la communication avec le monde extérieur, que ce soit pour des demandes HTTP ou des messages. L’extérieur fournit également des mécanismes pour récupérer des données sauvegardées, stocker les résultats de l’application et les envoyer ailleurs.

La logique métier (l’application et le domaine) se trouve à l’intérieur de l’hexagone. J’ai défini un type d’interface client avec quatre types de requêtes. Trois utilisent le même Port (probablement via HTTP) et la quatrième passe par un Port différent (comme AMQP).

Les _Adapter_ d’entrées convertissent ces demandes en opérations pour l’application, qui les traite ensuite en interne.

L’_Adapter_ de sortie, quant à lui, envoie des données vers des systèmes externes, comme un message AMQP pour notifier un changement d’état. Le Port utilisé est différent de celui utilisé pour la persistance.

![Architecture héxagonale](/assets/images/2024-12-27/hexagonal-architecture.png)

### CQRS (Command Query Responsibility Segregation)

Le **CQRS** est un pattern qui stipule que toutes les méthodes doivent être séparées en deux catégories : les **Command** et les **Query**.

- Une **Command** exécute une action. Elle modifie l’état d’un objet et ne retourne pas de valeur, à l’exception de l’id de la ressource créée si nécessaire.  
- Une **Query**, en revanche, ne fait que retourner des données, sans jamais changer l’état d’un objet.

La séparation stricte clarifie la structure et rend le code plus lisible et maintenable.

#### Vision du Futur

Il y a une autre partie du **CQRS** que je n’évoque pas ici, car je ne l’ai découverte que plus tard : la séparation de la couche de lecture et d’écriture. J’aborderai ce point plus en détail dans un article ultérieur du Journal.

![CQRS](/assets/images/2024-12-27/cqrs.png)

### Bounded Context

Afin de mieux structurer un projet, la notion de **Bounded Context** est importante.

#### Qu’est-ce qu’un Bounded Context ?

Chaque **Bounded Context** est délimité de manière à encapsuler les concepts, les règles métier, les processus et les modèles de données relevant de sa responsabilité. Il représente les limites où s’applique l’_Ubiquitous Language_.

#### Comment identifier un Bounded Context ?

En m’appuyant sur mon **Domain Vision Statement** [lien vers l’article 1], j’ai identifié trois **Bounded Context** :
- **Country** : gère la liste des pays
- **Bottle** : gère les bouteilles et les cépages (variété de plants de vigne cultivée)
- **User** : gère les utilisateurs et la sécurité

![Des amis, du vin context map](/assets/images/2024-12-27/bottles.png)

## Structure du projet

Chaque **Bounded Context** a son propre dossier. Dans mon cas, je vais donc en avoir trois : **Bottle**, **Country** et **User**. Il est crucial qu’aucun code ne traverse les frontières des **Bounded Context**. Si on veut faire communiquer des **Bounded Context**, on doit passer par des messages asynchrones ou des appels API. On valide cette règle avec l’outil ***Deptrac***.

Dans chaque **Bounded Context**, on va retrouver :

- Un dossier **Domain**, dans lequel on retrouve les **Entity**, les **Value Object**, les **Repository**, les _Services_ liés au Domaine, les _Events_ du Domaine.
- Un dossier **Application**, dans lequel se trouve toute la couche **CQRS**, avec un dossier **Command** et un dossier **Query**. J’ai ajouté également les _Event Listener_ qui écoutent les _Events_ du Domaine. Cette couche est le liant entre la partie **Infrastructure** et la couche du **Domain**.
- Un dossier **Infrastructure**, dans lequel on place les implémentations concrètes des interfaces définies dans le **Domaine** et tout ce qui est en charge de la communication avec l’extérieur (stockage, mailing, authentification), mais aussi les ressources API.

Voici ce que ça donne pour mon projet (j’utilise ***PHPStorm*** pour développer) :

![Des amis, du vin structure du projet](/assets/images/2024-12-27/code-structure.png){:width="25%"}

La couche **Application** est facultative. On peut avoir des implémentations de l’**Architecture Hexagonale** avec seulement le **Domaine** et l’**Infrastructure**.

Ce journal de bord est, une fois encore, dense en nouveaux concepts. Se lancer dans un projet avec toutes ces nouvelles idées est à la fois un défi et une opportunité d’apprentissage. Je vous donne rendez-vous très bientôt pour un nouvel épisode.