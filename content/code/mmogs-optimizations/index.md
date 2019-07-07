---
title: "MMOGs optimizations"
date: 2016-09-27

categories: ['Game Development']
tags: ['MMOGs', 'Performance', 'Optimizations']
author: "Emanuele Manzione"
noSummary: false

resizeImages: false
---
An user asked me about how I am dealing with main MMOGs problems in Heroes of Asgard, so I prepared an article about this topic.

#### DEFINITION
By definition, a MMOG should allow you to play with a huge amount of people at once and interact with them as if you are in a normal multiplayer game. All of this in a plausible, persistent world.
<!--more-->
Now, if we want to dissect a little more this statement, we will see that this is impossible without applying various “tricks” behind the scenes.

#### COMPLAINING ABOUT PERFORMANCES
You can definitely understand how when the amount of connected players grows, server's performance degrades.
Many operations on the server operate on all connected players or a subset of them, on all objects around the world, on all monsters and their AI, etc. All these calculations are executed several times per second: imagine, then, having to iterate over 200 players, having to iterate over 2,000 players or having to iterate over 20,000 players, on each frame of your server simulation. For each iteration, I have to send packets, make calculations, change positions, etc. There is, therefore, an exponential growth of the computational load for each new connected player.
As you can guess, it is a very large amount of work for a single machine, this due to an obvious hardware limitation.
Usually, therefore, there is a maximum threshold of concurrent players simultaneously processed, after which the server itself (the physical machine) can not keep up, creating a negative game experience (lag, unresponsive commands, etc).
You can not accept new connections beyond this threshold until a seat becomes available, in order to not ruin the experience for those who are already connected and playing.
You could then start multiple server instances on different machines. Now you can host more players, but of course they can not interact with players from other servers.
The subdivision into multiple “server instances” definitely does not fall within the definition of MMOG: it does not allow the player to interact with all players in a persistent world, but it creates different instances of the same world. It is acceptable, of course: but it isn’t what we want to achieve.
That said, what can we do to “bypass” a little bit this problem? And what did I already do for Heroes of Asgard? What I describe is the result of my experience and, therefore, it is also what I provided for Heroes of Asgard.

#### WHAT CAN WE DO?
There are several tricks that can be applied to improve the maximum threshold. Yes, improve it: there will always be a maximum threshold beyond which it is difficult to go (by maintaining the same hardware, of course).

#### YOU PAY FOR THE CODE YOU WRITE
As first thing: write good code, with your brain plugged in and without wasting resources. It may seem obvious, but it is not. Wasting resources brings to less available resources.
Wasting bandwidth means exhausting it in no time, every single piece of data that is transmitted has to be carefully selected. If I send an extra byte for each user, when my server hosts 20,000 players, it means sending about additional 20KB for each frame.
Wasting CPU cycles is like shooting myself in the foot: the actions you perform must be kept to a bare minimum; adding a single more function call per user may mean adding N additional CPU cycles, which for 20,000 users will be N x 20000 additional CPU cycles.
Wasting memory is harmful: allocations require both additional CPU cycles and memory. And system memory is not infinite.
In managed environments leaving resources allocated causes garbage collection, which may mean spending huge CPU cycles to free resources, instead of serving the players and simulate the world.
Ultimately, wasting resources in your code will ensure that you will spend more money and more frequently to improve your servers (when your userbase increases), in order to maintain acceptable performance.

#### FIX YOUR SIMULATION
As you certainly know, the simulation of a virtual world can be executed a certain number of times per second by the server. This means that every second, all entities and systems in the world are “simulated” a certain number of times. The simulation can include AI routines, positions/rotations updates, etc. It allows you to infuse “life” to your virtual world.
A simulation step is called "tick", so the number of times your simulation is performed is called TPS or Ticks Per Second. It is obvious that if the simulation is cumbersome and requires time, our hardware will tend to simulate the world less times in one second. This will lead to a degradation of the simulation.
But consider: do we need a big amount of simulations performed by the server? Do we need to strive our hardware in this manner? Can we improve this?
Yes. For most games with few players in the same map, and a high game speed (see the TPS, with a high number of commands) our world can be simulated 60 times per second (or less, obviously it depends on game type).
For a MMOG a littler amount can be enough, depending on the genre.
In Heroes of Asgard, for example, the world is simulated 20 times per second (at the moment).

#### DO WE CARE ABOUT THE ENTIRE WORLD?
We said that in an MMOG we must be able to interact with other players and with the surrounding environment and I should be able to do it with anyone in the world at that time. Quite right, of course.
But, from the point of view of a player, do you really need to know what a player is doing on the other side of the map? No, not always. 
Indeed, in the majority of cases this player isn't interested to know if another player, as example, is walking or not in another far area. Send an information that can not be displayed on the user's screen is a waste of resources.
This point is important, it allows us to implement a big optimization.
How can I inform a particular player only about entities that may interest them?
Why not split the map (or maps) in zones? A simple subdivision is a grid-division: split the map in N x M zones, where N and M are greater than or equal to 1. This technique is also known as space partitioning or zones partitioning.
In this way a player is able to only receive information about entities contained in their area, without needing to have knowledge of far entities. 

If in my map 8000 entities are uniformly distributed and it is subdivided as a 4x4 grid, the player who is in the [1, 1] zone will receive information about only 500 entities. A great advantage, isn't it?
But consider: what if the player is on zone's borders? It will not see the players in the nearby zones, although they should be visible.
We can understand that the player will have to be informed about the entities contained in its zone and in zones immediately contiguous.
Setting a good size of the zones allows you to optimize a lot this method, so depending on the size of a map the size of the grid can vary, in order to obtain the best effect. Also the shape of the zones can vary, to better fit to the composition of the map.

#### LOOK AS FAR AS THE EYE CAN SEE
As mentioned, zone division already offers a decent level of optimization, allowing us to send information about an entity to the players who really can benefit from it.
But let ask ourself a question: can we identify useless information in our zone division (remember to also include contiguous zones, so in a regular grid we have to deal with 9 zones in the worst case)? Of course we can.
Usually a player does not affect entities outside of their field of view.
If I can not see an entity, I do not care to keep track of what it is doing. Otherwise sending information about that entity is a waste of resources.
How can you determine what your server needs to send to a specific player? The easiest way is to trace, in fact, the field of view. Everything within that radius is what matters to the specific player, entities outside are not necessary to the specific player's world simulation.
And since we already have a zone subdivision, we can simply iterate over the entities in player's zones of interest (instead of all entities in the map) to determine who is within our field of view. This concept is also called area of ​​interest or AoI.
So, continuing the example, let's iterate on 500 entities instead of 8000, to extrapolate 25 of them which fall within the visual range and start exchanging information through the network only with them.
From 8000 to 25, a good result: isn't it? And the user does not suffer of missing information as it does not see them. Indeed, they will notice less usage of resources.
You can further enhance the area of ​​interest, by applying various measures:
 - organize various levels of visual rays; the most distant entity will receive updates less frequently
 - filter the interesting entities depending on the morphology of the map; if an entity is in our sight but behind a mountain, I can ignore it. This measure, however, only makes sense if you already use culling for other things, so you don't introduce additional calculations to filter out few other entities (in my opinion)

#### DISTRIBUTE YOUR COMPUTATION LOAD
We already said that a single machine will still have a certain threshold beyond which, despite all the optimizations made, you will experience performance degradation (with a resulting bad gaming experience).
Fine, but then why not to take advantage of multiple computers simultaneously?
There are obviously different ways to do it.
For example, in Heroes of Asgard each map that composes the world is hosted on a separate process. This allows each map to be hosted on a different physical machine.
Obviously, however, you can go down even more and accommodate sets of zones on separate processes (so a single map may be divided into several parts and hosted by different servers).

#### SLICE YOUR PIE 
You can also combine global services (such as chat) in different server processes, to give to your player the impression that, even being connected to different maps (so different servers), you can interact with distant players. Furthermore, breaking those services from the main world simulation brings to the table an additional gain in performance.

#### RECYCLE YOUR TOYS
As mentioned, allocating memory costs a good amount of resources. So why do not reuse what we already allocated? The use of objects pools is really important in the multiplayer development. It allows us to shift the burden of allocating costs when it can be faced with no problems, for example during bootstrap of our server app.
A monster is defeated and dies? Well, I put it aside. I can use it again when another monster must be spawned, just recovering it from my pool.
Of course it is clear that you have to use a certain criteria in order to choose which objects to keep in memory and which do not. Should I keep in memory a pool of monsters that spawn once a month? No, it may be useless. Should I keep in memory a pool of objects representing the loot of the currency? Yes, it makes more sense.