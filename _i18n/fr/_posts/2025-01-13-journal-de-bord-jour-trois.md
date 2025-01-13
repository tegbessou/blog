---
layout: article
title: "Journal de bord de l’apprentissage du Domain-Driven Design : Jour 3"
date:   2025-01-13 08:00:00
categories: ddd-logbook
lang: fr
resume: "Cet article propose un tour d’horizon des composants clés du Domain-Driven Design (Entity, Aggregate, Value Object, Domain Service…) pour structurer un domaine métier. Il met l’accent sur la séparation des responsabilités et sur la nécessité de modèles immuables pour préserver la cohérence du système."
permalink: /ddd-logbook/2025-01-13
---

Dans les premiers articles, j’ai présenté la structure du projet et les concepts fondamentaux du **DDD**, comme l’**Architecture Hexagonale**, le **CQRS** ou l’**Ubiquitous Langage**. Ces bases étant posées, il est temps de plonger dans le cœur du sujet : le **Domaine-Driven Design**. Voici une série de définitions essentielles concernant le **DDD**. Bien qu’elles soient assez théoriques, elles sont nécessaires afin de bien comprendre son fonctionnement.

## Entity / Aggregates

### Qu’est-ce qu’une Entity ?

Une **Entity** est un objet métier caractérisé par son identité unique, propre au **Domaine**. Elle peut être composée de **Value Object** ou de variables de types primitifs (integer, boolean, float…) et représente un concept évolutif. Les **Entity** sont souvent persistées en base de données grâce à leur identifiant.

Dans notre système, par exemple, un **Bottle Owner** (la personne ayant ajouté la bouteille dans le système) est une **Entity** avec son **id** et son **email**. J’ai opté pour des **Entity** composées de **Value Object**, sauf pour les booléens, afin de rendre le code plus lisible et plus orienté métier. Cela oblige en revanche à écrire plus de code.

### Qu’est-ce qu’un Aggregate ?

Un **Aggregate** est un regroupement logique de plusieurs **Entity** et **Value Object**, qui forment une unité cohérente. Il possède une racine unique (**Aggregate Root**), qui est l’entité principale par laquelle l’**Aggregate** est référencé et manipulé. Toute modification des éléments internes de l’**Aggregate** passe obligatoirement par cette racine.

N’ayant pas d’exemple d’**Aggregate** dans mon application, on pourrait imaginer une dégustation de vin liée à des **Entity Invitations**. La racine serait la dégustation, et les **Entity Invitations** ne pourraient être modifiées qu’à travers elle.

L’**Aggregate** pourrait également inclure des **Value Object**, tels qu’une date de dégustation ou un nom de bouteille.

Si vous utilisez ***Doctrine*** pour gérer vos bases de données, vous pouvez décider de mettre ces *Attributs* dans les **Entity**. Cependant, il ne faut pas oublier que le code du **Domaine** doit pouvoir être extrait de votre application et continuer de fonctionner dans un autre contexte, sans dépendance à ***Doctrine***. Dans notre cas, les ***Attributs*** seront reconnus comme des commentaires dans d’autres langages.

#### Vision du Futur

Je ne recommande pas le couplage entre les **Entity** du **Domaine** et celles de ***Doctrine***, car cela peut créer des problématiques de modélisation, une violation de la séparation des préoccupations ou des difficultés lors des tests. J’ai déjà fait cette erreur, je vous l’expliquerai bientôt plus en détail.

## Validations et Assertions

Les données d’une **Entity** doivent respecter les règles métiers applicables. Par exemple, une **Bottle** avec un **Price** doit être strictement supérieur à 0. Ces contraintes (appelées ***Assertions***) doivent être vérifiées dans une **Factory** qui garantit la validité des **Entity** dès leur création.

Toutes les actions affectant une **Entity** doivent être représentées par des méthodes spécifiques. Par exemple, une méthode `Create` pour créer une **Entity**, ou `Taste` pour déguster une bouteille. Ces actions déclenchent parfois des **Domain Events**.

## Value Object

Voyons maintenant ce qui constitue les **Entity** et les **Aggregate** : les **Value Object**.

Un **Value Object** est une représentation d’un concept métier. Il doit être **immuable**. Ce sont des représentations de concepts métiers, définies par leurs attributs plutôt que par une identité unique. Son immutabilité garantit qu’il ne change pas d’état après sa création.

Dans l’exemple précédent, le **Bottle Name** est un **Value Object**.

## Factory

Parfois, la création d’un objet n’a sa place ni dans l’**Entity**, ni dans le **Value Object**, ni dans l’**Aggregate**. On crée alors des **Factory**. Leur responsabilité reste ancrée dans le **Domain**.

Dans ce projet, j’ai pris la décision d’utiliser des ***méthodes Static*** pour mes **Factories**. Cette approche était suffisante, mais il est totalement envisageable d’utiliser un **Service** à la place.

## Repository

Le **Repository** est une interface dont le rôle est de stocker, lire et modifier les **Aggregates** et les **Entity**. On fait abstraction de l’endroit où ils sont persistés. Une seule règle clé : chaque **Aggregate** ou **Entity** ne doit avoir qu’un (et un seul) **Repository**.

Nous aurons par exemple un **Repository** pour les **Bottles**, avec une méthode `Add` pour ajouter et une méthode `OfId` pour récupérer une **Bottle** par son **id**.

J’ai pris le parti de laisser au **Repository** le soin de gérer les identités des **Entity** et des **Aggregates**. J’ai donc créé pour cela une méthode `NextIdentity` dans les **Repository**, responsable de générer les **id**.

## Domain Event

Un **Domain Event** est une représentation d’un événement significatif qui s’est produit dans le **Domain**, reflétant un changement d’état important d’une **Entity** ou d’un **Aggregate**. Un **Domain Event** est **immuable**, identifiable par un **id** unique et horodaté, pour permettre le suivi et la gestion de l’ordre des **Domain Event**. Il est utilisé pour notifier d’autres parties du système (ou d’autres systèmes) des actions effectuées.

Par exemple, quand on appelle la méthode `Create` sur l’**Aggregate**, on enregistre un **Domain Event** qui sera émis après avoir été stocké dans le système. Ce **Domain Event** doit être représentatif de l’action réalisée par le système (dans notre cas, **BottleCreated**).

## Domain Service

Un **Domain Service** est un composant sans état qui encapsule une logique métier importante ne pouvant être attribuée à une seule **Entity**, à un **Aggregate** ou à un **Value Object**. Il représente des opérations ou des processus métier impliquant plusieurs **Entity** ou **Aggregate** et opère uniquement sur les objets du **Domaine**.

Je n’ai pas d’exemple dans le cas des **Bottles**. Mais si on reprend le cas de la dégustation d’une bouteille de vin et de l’invitation de personnes, on peut créer un **Domain Service** en charge de vérifier si un participant peut être invité (s’il n’est pas déjà invité, ou s’il n’est pas l’organisateur de la dégustation). Le **Domain Service** pourrait ensuite créer les différentes invitations.

## Shared Kernel

Le **Shared Kernel** est une partie du domaine partagée entre deux **Bounded Contexts**. Ceci inclut le code ou la base de données associée. La partie partagée a un statut spécial et ne doit pas être changée sans consulter les deux équipes qui maintiennent les **Bounded Context**. Il faut intégrer des tests fonctionnels sur cette partie et les lancer fréquemment.

#### Vision du Futur

J’ai eu une mauvaise vision du **Shared Kernel**. Je pensais pouvoir l’utiliser pour partager du code technique, mais il n’est censé partager que des notions métiers.

Voilà pour cet article très théorique. La prochaine fois, j’aimerais rentrer dans le concret en partageant des exemples liés aux cas pratiques.

**À très bientôt !**