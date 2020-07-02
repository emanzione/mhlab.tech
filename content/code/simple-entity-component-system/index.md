---
title: "A simple Entity Component System"
date: 2030-04-15

categories: ['Game Development']
tags: ['ECS', 'Performance', 'Optimizations']
author: "Emanuele Manzione"
noSummary: false

resizeImages: false
---
Despite the fact ECS pattern and data-oriented programming are a thing in game development industry since a very long time, in the indie world I've heard a lot of raising hype around it only in the last few years. Probably because of Unity pushed it a lot recently, but more developers are approaching it for their games.
I never used it, so I am very curious to understand this pattern in a more detailed way and what it could bring to the table.

<!--more-->

The best way for me to learn something new is to code it. So this article is all about my own implementation of this pattern in C#. Let's start!

#### What is an Entity Component System?

It is an architectural pattern (it controls the fundamental structure of how your software) that follows the **composition over inheritance** principle. It is composed by three main concepts:

* **Entity**: it represents the unit, a general purpose object. Usually it is just an ID.
* **Component**: it represents the data and has no logic.
* **System**: it represents the logic to execute on components.

#### Why is it interesting?

It has some interesting upsides. Obviously, it promotes a data-oriented approach so it exploits the way the CPU **cache** works (with evident gainings in performance). But that's not the most important thing, imho.

It is **strict**: it forces you to follow a specific code architecture, so you can actually think about how to solve the actual problem instead of focusing on the architecture to support the code you're going to write. The **composition over inheritance** principle makes your code easily expandable and tunable, with a general improvement in **maintainability**. Not only, it is easily **testable**: you just have to mock some components in order to test a system.

So, in general, my interest for it is oriented on the improvements it can bring in my team's workflow and on the **quality** of our codebases/games. 

So let's explore it!

#### Implementation goals

Before I dive into the implementation, I want to fix some goals and constraints.

* I use C# and .NET Core
* I am learning from this, so my implementation will be simple and easy to understand
* It should be usable by other devs in the team, so a good API is mandatory
* Components have to be structs and I want to avoid boxing/unboxing where possible
* No runtime reflection

#### The Entity

Let's start simple! The entity is just an ID, so nothing fancy or complicated:

```csharp
[StructLayout(LayoutKind.Sequential)]
public struct Entity
{
    public uint Handle;
}
```

#### The World

A World instance is basically a big container of entities and components. It also handles and runs all systems.

#### The Component

Components are just data, so I am going to represent them with simple structs (in this way they are contiguous in memory).
