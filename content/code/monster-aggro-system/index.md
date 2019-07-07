---
title: "A simple monster aggro system"
date: 2015-08-31

categories: ['Game Development', 'Rants']
tags: ['aggro', 'gameplay', 'Heroes of Asgard']
author: "Emanuele Manzione"
---
During the development of Heroes of Asgard's server logic I came across the development of a system for monsters and bosses' AI (as would be obvious in any game).
<!--more-->
The general system was already set up and I have to say that I was satisfied, each monster has its *AI_GROUP* pointing to a list of Behaviors and each one of them is running into its update loop.

But now I want to create specific behaviors, in the specific case of this blog post I want to create an _aggro*_ behavior.

_*For those who do not know what is the aggro: it is the entity's aggressiveness._

### Let's start
So I analyzed the problem.
The behavior begins with the monster in neutral state (or IDLE), it will perform only its passive behaviors (such as randomly walking around its spawn point).
The aggro is enabled when a player joins the list of attackers/threats.

The player has two ways to join it:

- They directly attack the monster
- They enter the monster's field of vision (if it is aggressive)

The monster then continue to perform its behavior accordingly with the current state of aggro and will decide when to attack, when to retire or chase.
**So far so smooth**. 

My dilemma arises here: what if the list of attackers contains more than one player? Clearly, the monster has to choose who to honor with its loving attention and who to ignore.
This behavior has a dramatical impact on the gameplay, as I’ll explain in a while.

Mini-glossary for you:

- **_Tank_**: the one who gets the beating in most RPGs, characterized by huge life points, little damage dealt and slowness.
- **_Damage dealer_**: the one who beats in most RPGs, characterized by fragile life points, high damage and a good speed.

### Attack the closest player
I thought of a simple _“attack the one who is closest, don’t give a f*ck about others“_, but this can create unexpected situations.
In this case the tank class (which is by definition a short-range melee character) will be **_ALWAYS_** the one who receives the damage, leaving damage dealers happy and safe. 
You will then have an extremism: the tank is the one who will equip any item able to prolong his life and damage dealers will equip anything to deal more damage, not having to worry about receiving damage.
Unpleasant.

### Attack who deals more damage
I then thought about the reverse case: _“attack who deals you more damage“_. Again, it can rise unpleasant situations.
In a comparison of dealt damage between a tank and a damage dealer, aggro will **_ALWAYS_** be forced on damage dealers, sending to hell roles and the concepts itself of tank and damage dealer classes.
Rejected.

### A mix of them
I then thought again: why not mix both methods? Attacking those around you until its damage is around the average (X%) of the damage dealt by all the attackers, otherwise go to break the neck to the one which damage exceeds this average.
In this way the mechanic is already more balanced, but even here you can appreciate the obvious limitations: once the damage thresholds, the tank has no more chance to obtain again the aggro if not chasing and attacking the monster, who is chasing the damage dealer, trying to deal more damage than the damage dealer does.
It will be funny.

### The end of our journey
How to solve these problems and give a consistent feeling to PvE fights? 
The last proposed solution is a good starting point, so I will work on it.
What to do, however, to solve these problems?
**_Aggro points_**. Any attacker will be aggred when they have the highest value of aggro points among all attackers.
How does it work then?
It adds a new bonus: the probability of obtaining aggro (AGGRO_PROB). It is a simple value from 0 to 1.
Depending on player's race, the basic AGGRO_PROB can be more or less high (higher in the tank, lower in the damage dealer). Of course it is possible to add this bonus on equipment.
When someone attacks a monster, he gains aggro points:
```csharp
aggro_points = damage_dealt * AGGRO_PROB
```
It is clear that a tank with 100% AGGRO_PROB gains maximum aggro points, while a damage dealer with little AGGRO_PROB gains much less. It is also clear that the bonus AGGRO_PROB will be highly sought by tank classes, but can act as a penalty on damage dealing classes (and this also adds a positive note to the gameplay).
Another addition is to make aggro points decreasing over time, with inverted speed.
Then a tank over time will lose few points, a damage dealer will lose many.
```csharp
aggro_points_decay = (((current aggro_points * k) / 100) / AGGRO_PROB) * k / 100)
```
At this point we can give values ​​to these expressions and do the math.

Tank:
```csharp
damage_dealt = 800
AGGRO_PROB = 1
```
Damage Dealer:
```csharp
damage_dealt = 2000
AGGRO_PROB = 0.1
```
The tank attacks the monster A.
```csharp
tank_aggro_points = damage_dealt * AGGRO_PROB = 800
```
The damage dealer attacks the monster A.
```csharp
dd_aggro_points = damage_dealt * AGGRO_PROB = 200
```
Damage dealer dealt a lot of damage, but the aggro remains on our tank.

Now we modify damage dealer’s damage: `damage_dealt = 10000`.
```csharp
dd_aggro_points = damage_dealed * AGGRO_PROB = 1000
```
This time aggro is assigned to damage dealer, because he did a huge amount of damage more than our tank.
But now it will be easier to return aggro at tank. Let k = 20:
```csharp
tank_ap_decay = (((800 * 20) / 100) / 1) * 20/100) = 32
dd_ap_decay = (((1000 * 20) / 100) / 0.1) * 20/100) = 400
```

After a tick of decay, aggro pass back to the tank. Obviously the numbers are a little bit random, but you can get it. We need to tune them in future.
In addition, the calculation can also be added the level of the player or the difference of levels, or Dexterity, etc.