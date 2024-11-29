---
layout: article
title: "Logbook of Learning Domain-Driven Design: Day 1"
date:   2024-12-02 16:00:00
categories: ddd-logbook
lang: en
resume: "On this first day of my logbook on Domain-Driven Design, I share my initial discoveries and learning journey of this domain-focused approach to designing complex software. Join me as we explore the fundamental concepts of DDD, its best practices, and lessons learned from my personal experiences."
permalink: /ddd-logbook/2024-12-02
---

Welcome to this logbook where I share my experience with **Domain-Driven Design (DDD)**, a strategic approach to designing complex software. It places the **business domain** at the core of **modeling**, making it more coherent and comprehensible. Through a series of articles, we’ll explore the **fundamental concepts**, **best practices**, and common pitfalls of **DDD**—all based on my personal experience. Whether you’re a developer, software architect, or professional, this will offer you a step-by-step introduction to **DDD** and its **core principles**.

## Before diving into the details:

- **Bold terms** are part of the **DDD lexicon** or relate to system architecture concepts, with definitions available in an appendix.
- *Italicized references* indicate book authors and their works.
- ***Bold and italicized terms*** refer to function names and specific code terms in my project.
- Words <u>underlined</u> come from the **Ubiquitous Language**.

## Why write a logbook?

The goal of this article series is to document my journey with **DDD** by sharing each step of my learning process and project creation. Through your feedback and discussions, I aim to continue learning and refining my understanding of this fascinating subject.

I’ll share my mistakes, the corrections I made, and the reasoning behind my decisions so others can learn from my discoveries (and avoid repeating the same errors!). The “**Vision of the Future**” sections explain mistakes before I delve into them further in the logbook.

## Why use **DDD**?

As I began exploring this practice, I discovered its value for **project management**.

### The Discovery

It all started when I had to learn about **DDD** for one of my freelance assignments. I purchased *Domain-Driven Design* by *Eric Evans* (the creator of **DDD**), but the book ended up sitting on my shelf unread after I was hired without needing to know more about it.

Later, during a downtime between contracts, I decided to revisit the book, intrigued by its content. As I read, I found the concept far more complex than anything I’d encountered before. However, the more I engaged with it, the more I realized how fascinating it was as a way to **design software**. The deeper I went, the more excited I became to try this method—it brought a new sense of purpose to **software creation**. I finished the book in its entirety, taking notes on the key aspects to keep handy.

After a month, I had nearly as many questions as answers. To deepen my understanding, I turned to other readings, including *Advanced Web Application Architecture* by *Matthias Noback*. While not specifically focused on **DDD**, this book provided a more practical perspective. Armed with these insights, I felt ready to develop my first **DDD** project.

While working on this project, I also read *Implementing Domain-Driven Design* by *Vaughn Vernon*, which further enhanced my approach to **DDD**.

## Ubiquitous Language

A key concept in **DDD**, the **Ubiquitous Language** refers to the terms related to the **business domain**. It lies at the heart of the system and should be developed through collaboration between developers and domain experts.

All features derive from the **Ubiquitous Language**, and the code must reflect it.

## My First Project Using Domain-Driven Design

To practice **DDD**, I decided to create an API to catalog the <u>wines</u> I drink, allowing users to <u>record</u>, <u>view</u> (via a mobile app to be developed later), and create <u>tasting notes</u>. The only thing left was to find a name—***Des amis, du vin*** (Friends, Wine) came to me naturally.

### Domain Vision Statement

Before starting the project, I followed the practice recommended by *Eric Evans*: writing a **Domain Vision Statement**.

#### What is a **Domain Vision Statement**?

A **Domain Vision Statement** serves as a guide to keep the development team aligned in the same direction while refining the **model** and **code**.

To create one, write a brief description of the **Core Domain** and the value it provides. Avoid emphasizing what sets this **Domain** apart from others; instead, show how the **model** serves and balances diverse interests. This document should be written as early as possible and revised with every new round of feedback.

#### For My Project

Here is the initial version, which focuses solely on the primary features:

- A <u>bottle</u> includes a <u>name</u>, <u>domain name</u>, <u>type</u> (<u>white</u>, <u>red</u>, <u>rosé</u>, <u>champagne</u>, <u>sparkling red</u>, <u>sparkling white</u>), <u>year</u>, <u>grape variety</u>, <u>country</u>, <u>price</u>, <u>rating</u>, <u>notes</u>, <u>photo</u>, and the <u>date it was added</u>.

- The functionalities to include are <u>creating</u>, <u>modifying</u>, and <u>deleting</u> a <u>bottle</u> with all its details, as well as <u>searching</u> based on various criteria (<u>wine name</u>, <u>domain name</u>, <u>type</u>, <u>date added</u>/<u>tasted</u>, <u>vintage year</u>, <u>rating</u>).

### Vision of the Future

At this point, I didn’t fully grasp the concept of the **Core Domain** and only documented the primary features. Nevertheless, I want to share the definitions of the **Core Domain** and the different **Subdomains** before detailing them further in an upcoming article.

## Core Domain and Subdomains

The **Core Domain** is where the most value should be concentrated. It’s essential to identify the central **domain** and provide a way to distinguish it from the supporting layers and surrounding **code**. The **Core Domain** should deliver the greatest value and introduce meaningful concepts. It may consist of multiple **Bounded Contexts**.

A **Subdomain** represents a smaller part of the **domain** that can be broken into modules to separate concerns. **Subdomains** are categorized as follows:

- **Supporting Subdomains**: These are important for the business but less critical than the **Core Domain**. They are created because they provide unique value.
- **Generic Subdomains**: These are generic areas with no specific business value.

### Vision of the Future

In my case, the **Core Domain** is creating an inventory of <u>wine bottles</u>.

This already represents a lot of information to absorb. If you remember just one thing, let it be the **Ubiquitous Language**—the central concept in **DDD**. The **Ubiquitous Language** is the connection between the business and the technical team, and it is essential that it is reflected in the **code**.

Feel free to leave a comment on this article. Were you already familiar with **Domain-Driven Design**? Which books and resources helped you get started?

See you soon for another episode of this logbook!
