---
layout: article
title: "Logbook of Learning Domain-Driven Design: Day 3"
date:   2025-01-13 08:00:00
categories: ddd-logbook
lang: en
resume: "This article introduces essential concepts of Domain-Driven Design (DDD), including Entities, Aggregates, Value Objects, and Domain Events. It also covers how to structure these elements within a domain, emphasizing best practices like immutability and separation of concerns."
permalink: /ddd-logbook/2025-01-13
---

In the previous articles, I introduced the structure of the project and the fundamental concepts of DDD, such as **Hexagonal Architecture**, **CQRS**, and **Ubiquitous Language**. With these foundations laid, it's time to dive into the heart of the subject: **Domain-Driven Design**. Here's a series of essential definitions related to DDD. Although they are somewhat theoretical, they are necessary for a proper understanding of how it works.

## Entity / Aggregates

### What is an Entity?

An **Entity** is a business object characterized by its unique identity, specific to the **Domain**. It can consist of **Value Objects** or primitive-type variables (integer, boolean, float...) and represents an evolving concept. **Entities** are often persisted in the database using their identifier.

In our system, for example, a **Bottle Owner** (the person who added the bottle to the system) is an **Entity** with their **ID** and **email**. I opted for **Entities** composed of **Value Objects**, except for booleans, to make the code more readable and domain-oriented. However, this does require writing more code.

### What is an Aggregate?

An **Aggregate** is a logical grouping of several **Entities** and **Value Objects** that form a coherent unit. It has a unique root (**Aggregate Root**), which is the main entity through which the **Aggregate** is referenced and manipulated. Any modification of the internal elements of the **Aggregate** must go through this root.

Since there isn't an **Aggregate** example in my application, we could imagine a **wine tasting event** that would be linked to **Invitation** entities. The root would be the tasting, and the **Invitation entities** could only be modified through it.

The **Aggregate** could also include Value Objects, such as a tasting date or a bottle name.

If you're using ***Doctrine*** to manage your databases, you may decide to put these attributes in the **Entities**. However, it's important not to forget that the **Domain** code should be able to be extracted from your application and continue functioning in another context without depending on ***Doctrine***. In our case, the ***Attributes*** will be recognized as comments in other languages.

#### Vision of the Future

I do not recommend coupling **Domain Entities** with ***Doctrine*** Entities, as it can create modeling issues, violate separation of concerns, or cause difficulties during testing. I’ve made this mistake before, and I’ll explain it in more detail soon.

## Validations and Assertions

The data in an **Entity** must adhere to applicable business rules. For example, a **Bottle** with a price must be strictly greater than 0. These constraints (called ***Assertions***) must be checked in a **Factory** that ensures the validity of the **Entities** upon creation.

All actions affecting an **Entity** should be represented by specific methods. For example, a `Create` method to create an **Entity**, or a `Taste` method to taste a bottle. These actions sometimes trigger **Domain Events**.

## Value Object

Now, let’s look at what constitutes the **Entities** and **Aggregates**: the **Value Objects**.

A **Value Object** is a representation of a business concept. It must be **immutable**. These objects are defined by their attributes rather than by a unique identity. Their immutability ensures they don't change state after creation.

In the previous example, the **Bottle Name** is a **Value Object**.

## Factory

Sometimes, creating an object doesn't belong in the **Entity**, **Value Object**, or **Aggregate** itself. In such cases, we create **Factories**. The responsibility remains grounded in the **Domain**.

In this project, I decided to use ***static methods*** for my **Factories**. This approach was sufficient, but using a **Service** instead is also totally feasible.

## Repository

A **Repository** is an interface whose role is to store, read, and modify **Aggregates** and **Entities**. The storage location is abstracted away. One key rule: each **Aggregate** or **Entity** should have only one, and exactly one, **Repository**.

For example, we might have a **Repository** for Bottles, with an `Add` method to add and an `OfId` method to retrieve a **Bottle** by its **ID**.

I decided to let the **Repository** manage the identities of **Entities** and **Aggregates**. So, I added a `NextIdentity` method to the **Repositories**, which is responsible for generating **IDs**.

## Domain Event

A **Domain Event** represents a significant occurrence within the **Domain**, reflecting an important state change of an **Entity** or **Aggregate**. A **Domain Event** is **immutable**, identifiable by a unique **ID**, and timestamped to allow tracking and managing the order of **Domain Events**. It is used to notify other parts of the system, or other systems, of actions that have taken place.

For example, when the `Create` method is called on the **Aggregate**, a **Domain Event** is recorded that will be emitted after being stored in the system. This event must reflect the action taken by the system—in our case, **BottleCreated**.

## Domain Service

A **Domain Service** is a stateless component that encapsulates significant business logic not tied to a single **Entity**, **Aggregate**, or **Value Object**. It represents operations or business processes involving multiple **Entities** or **Aggregates** and operates solely on **Domain** objects.

I don’t have an example for **Bottles**. However, taking the wine tasting scenario again, we could create a **Domain Service** responsible for verifying if a participant can be invited (if they are not already invited, or if they aren’t the organizer of the tasting). The **Domain Service** could then create the invitations.

## Shared Kernel

The **Shared Kernel** is a part of the domain shared between two **Bounded Contexts**. This includes the associated code or database. The shared part has special status and should not be changed without consulting both teams maintaining the **Bounded Contexts**. Functional tests should be integrated into this part and run frequently.

#### Vision of the Future

I had a misguided view of the **Shared Kernel**. I thought it could be used to share technical code. In fact, it is only meant to share **business concepts**.

That’s it for this very theoretical article. Next time, I would like to get into the specifics by sharing examples related to practical cases.

**See you soon!**