---
title: "Multiplayer RTS - Concepting"
date: 2031-02-23

categories: ['Game Development', 'Networking']
tags: ['multiplayer', 'rts']
author: "Emanuele Manzione"
noSummary: false

resizeImages: false
---
And here we are with the first, real post about development of this project! I apologize for the huge delay. :)

As most of you certainly noticed, we are planning to create a multiplayer game. This means that we need to communicate with other players around us: nobody likes a multiplayer game that has no multiplayer features!
This directly brings us to our next need: we want a way to transfer data among players, to make them know about each other and about their actions.

That's why I start with the foundation of every multiplayer game: the networking layer.

#### REQUIREMENTS

But what do we effectively need? What should our networking layer offer to make us happy?
We can't do a really detailed estimation, because we are in early stages. But we can start with something.

The classic topology for MMO games is client/server, with authority fully owned by the server: we make no exception. So we just need a way to send a chunk of data from client to the server and back. Nothing more.
To be more specific, we require that our chunk of data can be easily sent and its arrival has to be guaranteed on the other end (when we want/need). We all know that internet is a bad place, where our little network packets often lose their path and nobody is able to find them anymore. :)

I can already hear smartest among you who are already saying: “hey, but TCP is here for this reason”.
And it is right: TCP protocol is here to solve the issue. Under the hood TCP transparently re-sends lost network packets and makes you happy: no packets have been lost, not anymore!

But TCP does a lot of stuff, not only what we said before. It also transfers packets in a guaranteed order (like a stream, the first packet sent has to be the first received one on the other end), it is connection oriented, it can fragment your packets and reassemble them on the other side, it has flow/congestion control, etc. (If interested, you can read and study about TCP starting from [here](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)).
A lot of stuff, indeed!

All these features require more resources too: CPU cycles, memory, bandwidth. Plus, these features are not always useful for our objectives.
Indeed, for a business app or for a situation where all these features (or most of them, as we will see when we will build game services instead of game itself) are required we find our Saint Graal in TCP.

But what about games?
Usually games (particularly some genres) don't require the whole amount of TCP's features.
As example, we don't always want to resend a lost packet. This happens when the data contained in that packet is time sensitive and/or sent multiple times in a short interval, like a position update: if a more recent packet already arrived with the new shining data, we don't care anymore about the lost one who contains the outdated information.
We also don't always need guaranteed order: the async/event based nature of games doesn't fit with the concept of “order”. I can move my player while moving stuff in my inventory: I don't care if the position update arrives after the inventory update. I just care about ordering actions on the same entity, if required.

I can continue with examples, but you certainly got the point. So to make it short and more general: with TCP we are not able to model our networking layer to fit our game's gameplay architecture, constraints and requirements. And this is not an ignorable point.

On the other side, we have UDP protocol (learn more [here](https://en.wikipedia.org/wiki/User_Datagram_Protocol)) that basically is a connection-less, fire-and-forget protocol. It just carries chunks of data on the other end, if everything goes well. Nothing more.

This can seem unconvenient compared to TCP, but let's come back to the top of this needed explanation. We said we need just a way to send data and to flag it to guarantee the receiving if we need.
What is the simplest way to achieve this?
TCP does everything we need, but has a huge amount of features we don't need and we have to workaround someway. We probably have to fight with them to optimize traffic, CPU load, etc. It basically satisfies and conflicts with our objective at the same time.
UDP just sends data, we have to implement the reliability: but nothing else. After this, the modified UDP exactly fits what we need.

It is just matter of being able to analyze tools we have based on our needs and being able to choose among them. With all information we collected, probably UDP is the most suitable for our needs. It allows us to have a real control over how packets will be managed (in terms of reliability, etc).
Another reason for me to pick UDP is: I want to implement reliability on top of this protocol as personal challenge, but shhh. 😀

#### TRANSPORT LAYER

So we decided to roll our own UDP-based networking layer. I am so proud of us! 😀

I will focus mainly on:

* building reliability in a UDP-based transport layer
* explaining some design choices I'll make

If you are looking for a tutorial for existing networking libraries, this isn’t the right place for you: I am sorry. This is a journey through design decisions behind the development of netcode! :)