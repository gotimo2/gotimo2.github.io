---
layout: post
title: Exploiting a JET promotional game
date: 2022-11-15 15:43 +0100
tags: programming
categories: programming
published: false
---

So Just Eat Takeaway AKA the orange delivery company ~~AKA the guys who rip off small business owners~~, made a little game for their delivery drivers where you control a delivery driver in a game that, according to the TOS "Simulates working as a courier for JET" by making you play a very generic shooter, reskinned to have a delivery driver on a bike as the main player character. Also the bike has a gun for some reason. You drive forward, dodge and shoot obstacles, and rack up points and powerups.

If you've tried playing the game up until this point, you'll very quickly notice one thing: ~~the game is bad~~ the game is of less-than-stellar quality. It often does not start, the game runs horribly while also being memory-hungry, and the game's artstyle is uninspired at best. ~~they definitely got some poor intern to make this.~~ The game is in fact of such poor technical quality, [that the people in charge of the bug bounty program for Takeaway.com has announced they do not take reports of bugs and exploits and vulnerabilities with the game anymore.](https://bugcrowd.com/takeaway/updates/253db5b5-fb3b-4904-8488-fc022fc6a2e7).  

This game is bad. They attached a contest with a prize of monetary value to it. and then they left their un-webpacked source code open and visible for everyone to see.

![image of the game's file structure](/assets/img/JET/filestructure.png)

so, here is a guide on how to break a bad game.

Upon first starting up the game, the game will make a request to a master server which contains the following payload, telling the server 
 - what game we want to join (if i had to guess, this game belongs to the region where my friend lives)
 - who we are in the form of an identifier token (which is a friend of mine, i am impersonating him)
 - what language we are playing the game in (if i had to guess, to save for later)
 - if we have read the terms and conditions (which i have actually done for once, to figure out if there is a rule against cheating (there is not))

In the response body is the identifying bearer token, which the game will use from now on until we close the tab, to make requests to the server and identify us with.

![Image of the token](/assets/img/JET/token.png)

![Image of the identifier](/assets/img/JET/identifier.png)

