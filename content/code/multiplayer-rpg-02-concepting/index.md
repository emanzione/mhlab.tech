---
title: "Multiplayer RPG Series - Concepting"
date: 2017-08-28

categories: ['Game Development', 'Networking']
tags: ['multiplayer', 'rpg']
author: "Emanuele Manzione"
noSummary: false

resizeImages: false
---
So – as I said – to create this blog posts series I will develop a game step-by-step to help myself to understand what I need to write down.

Before I start with code and tech, I want to do a little bit of concepting: just to have a straightforward target and to have well-formed ideas. Also, I will try to establish what I need to develop in order to complete this game. Yeah, it will be boring but necessary: so grab your patience and embrace it! ;)

#### CONCEPT
Well, first of all – of course – I have to decide what game I want to develop. Of course this series is focused on multiplayer RPGs with persistent worlds. But I would like to go more deep by defining more details, rules and lore. In this way I can have a better overview of what I exactly need and I can give you a little bit of context about what is in my mind.

- I want it to be an __open world__, because I want to deal with "area of interest" and "zones partitioning": they are interesting challenges.
- I want to keep it relatively __simple__ and focused on multiplayer part: so no complicated gameplay concepts and contents. They will come later, on finalization steps.
- I want to build __reusable__ core, components and systems. I will develop them a single time and I will reuse them in my future games. For this reason I will try to maintain them as much general purpose as possible. The test will be: I have to be able to use some of its modules like a library/plugin.
- I want the world simulation to be __persistent__.
- I would like to play a little bit with __AI__, so I will introduce enemy entities with various and different behaviors.

About lore and world context: I love Norse mithology and Vikings, so that's my reference. I will create the virtual world with this in mind.
Based on previous point, the name will be __"Till Valhalla"__. Yeah, I know, I know: I am so much creative… :)
I think it is all for now: I will update this set of rules later if something else comes up in my mind. ;)

#### TECHNOLOGIES
I should also define the stack of technologies I want to work with. As I mentioned, I would love to work with stuff I didn't use before or I want to learn better. In this way you can easily check what this series will contain and you can decide if you want still to follow me: if chosen technologies aren't interesting to you, I will not waste your time. :)

- I will use __C#__ with __.NET Core__ and __Rust__ as main programming languages and environments
- I will use __MariaDB__ as RDBMS
- I will use __Redis__ as in-memory cache and shared storage
- I will use __Azure__ or __AWS__ services and APIs to host servers and services
- I will use __Unity3D__ as game engine

That’s basically the stack of technologies I will use. Probably I will add something else later, but I think it is a good starting point!

#### CODE OF CONDUCT
In order to have nailed in mind the direction I want to follow during the development, I would like to set some targets and a little code of conduct.

- __Optimization__. Multiplayer games need a lot of optimizations, especially when we are talking about open world. That's why I will try to optimize everything I can think of: it is my main focus.
- __Multithreading__. This is a subpart of "optimization": it is important to use each core of our computers, especially on server side. A better usage of resources will end up in a costs saving!
- __DOD__. Data Oriented Design is something that fascinates me a lot. I would like to apply its concepts and rules in this project.
- __Comments__. I will try to follow these rules for comments:
 - Public methods, classes, structs, etc will be commented (except if it is really unnecessary)
 - For remaining entities, they will be commented just if they are complex or to explain a weird behavior/design choice

#### VERSIONING
__Git__ will be used as version control software. This project will be entirely pushed on a private repository (probably GitLab or Bitbucket) and some parts of it will be pushed on a public repository (on GitHub).

 

Also remember that this is not a tutorial. I will not guide you step-by-step over "how to build a MMORPG in 5 simple steps" and it will not be a way to copy-paste some code. I will explain main concepts, interesting challenges, logic behind features, etc. The objective is to learn together in order to solve hard problems and main challenges that exist behind a multiplayer game: and that's what I will try to accomplish! :)

 

Well, here we are: seems that the starting part is done. Now I just need to study and start coding!

Stay tuned!