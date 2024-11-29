---
layout: article
title: "Journal de bord de l’apprentissage du Domain-Driven Design : Jour 1"
date:   2024-12-02 16:00:00
categories: ddd-logbook
lang: fr
resume: "Dans ce premier jour de mon journal de bord sur le Domain-Driven Design, je partage mes découvertes initiales et mon parcours d'apprentissage de cette approche centrée sur le domaine métier pour la conception de logiciels complexes. Rejoignez-moi pour explorer les concepts fondamentaux du DDD, ses bonnes pratiques et les leçons tirées de mes expériences personnelles."
permalink: /ddd-logbook/2024-12-02
---

Bienvenue dans ce journal de bord où je partage mon expérience du **Domain-Driven Design (DDD)**, une approche stratégique pour concevoir des logiciels complexes. Il place le **domaine métier** au cœur de la modélisation pour la rendre plus cohérente et compréhensible. À travers une série d’articles, nous explorerons ensemble les concepts fondamentaux, les bonnes pratiques et les pièges à éviter, le tout à travers mon expérience personnelle. Que vous soyez développeur, architecte logiciel ou professionnel, vous trouverez ici une présentation progressive du **DDD** et de ses fondamentaux.

## Avant de plonger dans le vif du sujet

Les termes en gras font partie du lexique DDD ou sont liés à des notions d’architecture système, vous pouvez retrouver leur définition dans un chapitre annexe.

Les références en italique mentionnent les auteurs de livre et leurs ouvrages.

Les mots en gras et en italique font référence aux noms des fonctions et aux termes de code spécifiques à mon projet.

Les mots soulignés en rouge sont issus de l’Ubiquitous Language.

## Pourquoi écrire un journal de bord ?

L’idée de cette série d’articles est de documenter mon expérience du **DDD**, en partageant chaque étape de mon processus d’apprentissage et de création de projet. Grâce à vos retours et à vos discussions, je souhaite continuer à apprendre et à affiner ma compréhension de ce sujet fascinant.

Je partagerai donc mes erreurs, les corrections que j’ai apportées et les raisonnements derrière mes choix, dans le but que d’autres puissent apprendre de mes découvertes (et évitent ainsi de refaire les mêmes erreurs que moi !). Les encadrés « Vision du Futur » servent à expliquer les erreurs avant que je n’aborde le sujet dans le journal de bord.

## Pourquoi utiliser le **DDD** ?

C’est en commençant à m’intéresser à cette pratique que j’ai découvert son intérêt dans la gestion de projets.

### La découverte

Tout a commencé lorsque j’ai dû m’intéresser au **DDD** pour l’une de mes missions freelance. J’ai donc acheté le livre d’*Éric Evans* (le créateur du *Domain Driven Design*) mais j’ai remisé le livre dans ma bibliothèque sans le lire, ayant été finalement engagé pour la mission sans qu’on ne m’en demande plus.

J’ai repris ce livre lors d’une période d’inter-contrat, intrigué par son contenu. En le lisant, j’ai trouvé le concept très compliqué, bien plus que ce que j’avais lu auparavant. Mais en m’y intéressant, j’ai découvert une façon fascinante de concevoir des logiciels. Plus j’avançais dans la lecture, plus j’avais envie d’essayer cette méthode. Cela donnait un sens nouveau à la création de logiciel. J’ai terminé le livre dans son intégralité en prenant des notes pour garder près de moi les aspects les plus importants.

Après un mois, j’avais presque autant de questions que de réponses. J’ai donc enrichi mon apprentissage avec d’autres lectures, notamment *Advanced Web Application Architecture* de *Matthias Noback*. Ce dernier livre m’a permis d’avoir une approche plus concrète du **DDD** même si ce n’est pas l’objet principal du livre. Je me suis alors senti prêt à développer mon premier projet en **DDD**.

Tout en avançant sur mon premier projet, j’ai lu *Implementing Domain Driven Design*, de *Vernon Vaughn*, qui a enrichi mon approche du **DDD**.

## Ubiquitous Language

Terme clé du **DDD**, l’**Ubiquitous Language** représente les termes liés au métier de votre domaine. Il est le cœur de votre système et doit être développé grâce à une collaboration entre les développeurs et les experts métiers.

Toutes les fonctionnalités découlent de l’**Ubiquitous Language** et le code doit en être le reflet.

## Le premier projet développé en Domain-Driven Design

Pour mettre en pratique le **DDD**, j’ai décidé de créer une API qui référence les <u>vins</u> que je bois, permettant de les enregistrer, de les visualiser (via une application mobile qui arrivera plus tard) et de créer des fiches de <u>dégustation</u>. Il ne restait plus qu’à trouver le nom, ***Des amis, du vin***, m’est venu naturellement.

### Domain Vision Statement

Avant de commencer le projet, j’ai suivi la pratique recommandée par *Eric Evans* : écrire un **Domain Vision Statement**.

#### Qu’est-ce que c’est ?

Un **Domain Vision Statement** peut être utilisé comme un guide qui maintient l’équipe de développement concentrée dans la même direction pour la distillation du modèle et du code.

Pour le mettre en place, il suffit d’écrire une courte description du **Core Domain** et la valeur qu’il apporte. Ignorez les aspects qui distinguent ce **Domain** d’un autre et montrez comment le modèle sert et équilibre les intérêts divers. Il est important d’écrire ce document le plus tôt possible et de le réviser à chaque nouveau retour.

#### Pour mon projet

Voici donc la version initiale, elle concerne uniquement les fonctionnalités principales :

Une <u>bouteille</u> est constituée d'un <u>nom</u>, d'un <u>nom de domaine</u>, d’un <u>type</u> (<u>blanc</u>, <u>rouge</u>, <u>rosé</u>, <u>champagne</u>, <u>pétillant rouge</u>, <u>pétillant blanc</u>), d’une <u>année</u>, d’un <u>cépage</u>, d’un <u>pays</u>, d’un <u>prix</u>, d’une <u>note</u>, d’une <u>remarque</u>, d'une <u>photo</u> et de la <u>date d'ajout</u>.

Les fonctionnalités à inclure sont la <u>création</u>, la <u>modification</u> et la <u>suppression</u> d’une <u>bouteille</u> avec toutes les informations, et la <u>recherche</u> par divers critères (<u>nom du vin</u>, <u>nom du domaine</u>, <u>type</u>, <u>date d'ajout</u>/<u>de dégustation</u>, <u>année du vin</u>, <u>note</u>).

### Vision du Futur

Je n’avais à ce moment-là pas la notion du **Core Domain**, j’ai donc écrit uniquement les fonctionnalités principales. Mais je partage néanmoins les définitions du **Core Domain** et des différents **Subdomain**, avant de les détailler de façon plus précise dans un prochain article.

## Core Domain et Subdomain

Le **Core Domain** est là où il faut mettre le plus de valeur. Il est nécessaire de trouver le domaine central et de fournir un moyen de le distinguer facilement de la masse du support du milieu et du code. Le **Core Domain** doit amener la plus grande valeur ainsi que des concepts. Il peut être composé de plusieurs **Bounded Context**.

Un **Subdomain** représente une petite partie du domaine qui peut être découpée en modules pour séparer le domaine. Il y a plusieurs types de sous-domaines :

- **Supporting Subdomains** : un ou plusieurs domaines importants pour le métier, mais moins que le core. On crée un domaine de support parce qu’il a quelque chose de spécial.
- **Generic Subdomains** : s’il n’y a pas de spécificité métier alors on parle de domaine générique.

### Vision du Futur

Dans mon cas, le **Core Domain** est de faire un inventaire de <u>bouteilles de vins</u>.

Tout cela représente déjà beaucoup d’informations à retenir. Si vous deviez ne retenir qu’une seule chose, souvenez-vous de l’**Ubiquitous Language** : c’est la notion centrale du **DDD**. Il est le lien entre le métier et l’équipe technique. Il est primordial qu’il transparaisse dans le code.

N’hésitez pas à commenter cet article. Connaissiez-vous le **DDD** ? Quels livres et ressources vous ont aidé à vous lancer ?

À très bientôt pour un nouvel épisode de ce journal de bord.
