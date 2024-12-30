---
layout: article
title: "Logbook of Learning Domain-Driven Design: Day 2"
date:   2024-12-30 10:00:00
categories: ddd-logbook
lang: en
resume: "This article explores a project setup using Symfony, Docker, and essential coding tools, combined with the principles of DDD, hexagonal architecture, and CQRS. It also covers the concept of Bounded Context to maintain clear domain boundaries."
permalink: /ddd-logbook/2024-12-30
---

In my previous article, I introduced the basic concepts of **Domain-Driven Design (DDD)**, particularly emphasizing the importance of **Ubiquitous Language**. Today, I will discuss the setup and structure of my project.

## Project Setup

I began development using ***Symfony*** my favorite ***PHP*** framework. It greatly simplifies the creation of complex web applications with its modular libraries while promoting good development practices.

For the work environment, I configured a ***Docker*** container with ***MariaDB***, ***Nginx***, and a proxy to manage ***HTTPS***, ensuring smooth local deployment.

The biggest challenge in starting a new project is the lack of external review. After spending countless hours on the same functionality, it becomes hard to identify weaknesses. To ensure adherence to the development rules I established, I integrated essential tools.

### Four Tools to Maintain Standards

1. ***PHP CS Fixer***: Ensures my code adheres to style standards and best writing practices.  
2. ***PHPStan***: Analyzes the code to detect bugs, typing omissions, and other potential errors.  
3. ***Rector***: A powerful tool for refactoring code and easily managing ***PHP*** and ***Symfony*** version upgrades.  
4. ***Deptrac***: Crucial for verifying compliance with the principles of **hexagonal architecture** or **Bounded Contexts**.

## Project Structure and Architecture

For this project, in addition to **DDD**, I aim to use **hexagonal architecture** combined with the **CQRS** pattern to create a scalable and easily testable application. This also simplifies codebase evolution.

### Hexagonal Architecture

The architecture is organized around two main areas: the inside (business logic) and the outside (interfaces).

The outside comprises **Ports** and **Adapters**, which handle client interactions by transforming requests into actions understandable by the interface. These **Ports** enable communication with the external world, whether for HTTP requests or messages. The outside also provides mechanisms to retrieve stored data, save application results, and send them elsewhere.

The business logic (application and domain) resides within the hexagon. I defined a client interface with four types of requests. Three use the same Port (likely via HTTP), while the fourth uses a different Port (such as AMQP).

- **Input Adapters** convert these requests into operations for the application, which processes them internally.  
- **Output Adapters** then send data to external systems, such as an AMQP message to notify a state change. The Port used differs from the one used for persistence.

![Architecture héxagonale](/assets/images/2024-12-30/hexagonal-architecture.png)

### CQRS (Command Query Responsibility Segregation)

**CQRS** is a pattern stipulating that all methods must be categorized as either **Commands** or **Queries**.

- A **Command** executes an action. It modifies the state of an object and does not return a value, except for the ID of the created resource if necessary.  
- A **Query**, on the other hand, only returns data without ever altering the state of an object.

This strict separation clarifies the structure and makes the code more readable and maintainable.

#### Future Vision

There’s another aspect of **CQRS** I won’t cover here, as I only discovered it later: the separation of the read and write layers. I’ll delve into this topic in more detail in a future journal entry.

![CQRS](/assets/images/2024-12-30/cqrs.png)

### Bounded Context

To better structure a project, the concept of a **Bounded Context** is essential.

#### What Is a Bounded Context?

Each **Bounded Context** is defined to encapsulate the concepts, business rules, processes, and data models it is responsible for. It represents the boundaries where the **Ubiquitous Language** applies.

#### How to Identify a Bounded Context?

Using my **Domain Vision Statement** (*linked in Article 1*), I identified three **Bounded Contexts**:

- <ins>Country</ins>: Manages the list of countries.  
- <ins>Bottle</ins>: Manages bottles and grape varieties.  
- <ins>User</ins>: Manages users and security.

![Des amis, du vin context map](/assets/images/2024-12-30/bottles.png)

## Project Structure

Each **Bounded Context** has its own folder. In my case, I will have three: **Bottle**, **Country**, and **User**. It’s crucial that no code crosses the boundaries of the **Bounded Contexts**. If communication between **Bounded Contexts** is needed, asynchronous messages or API calls must be used. This rule is validated using the **Deptrac** tool.

Within each **Bounded Context**, you will find:

- A **Domain** folder, containing **Entities**, **Value Objects**, **Repositories**, Domain **Services**, and Domain **Events**.
- An **Application** folder, housing the **CQRS** layer, with separate folders for **Commands** and **Queries**. I’ve also included ***Event Listeners*** that monitor **Domain Events**. This layer serves as the bridge between **Infrastructure** and **Domain**.
- An **Infrastructure** folder, which contains concrete implementations of the interfaces defined in the **Domain**, as well as components responsible for external communication (storage, mailing, authentication), and API resources.

Here’s what this looks like for my project (I use ***PHPStorm*** for development):

![Des amis, du vin structure du projet](/assets/images/2024-12-30/code-structure.png){:width="25%"}

The **Application** layer is optional. Hexagonal architecture implementations can consist solely of the **Domain** and **Infrastructure** layers.

This journal entry is dense with new concepts. Embarking on a project with these ideas is both a challenge and a learning opportunity. Stay tuned for the next episode.
