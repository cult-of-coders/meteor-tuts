---
title: 'Intro'
description: The View Layer
disqusPage: 'Chapter 2: Intro'
---

## Introduction

Previously, we learned how to harness Meteor's powers for building web applications. But we did not see anything actually happen in the View Layer (the UI).

Since Meteor is a system that focuses on data more than anything else, of course it gives you some snacks to help you
code faster: The User System, Emails and so much more. But when it comes to the View Layer, it's up to you!

Don't panic! Meteor can be integrated with any View Layer/Template Engine/UI Framework you want to use! 

## Brief history

When Meteor started there weren't any stable solutions for building reactive views with it. Because of that the developers had to invent a way.
This is how [Blaze](http://blazejs.org/) was born. In Q4 of 2015, the developers decided to drop the 
support for Blaze, not because it wasn't good, but because there were other teams that focused only on the
view layer. You cannot expect one company to do everything. So they began decoupling Blaze from Meteor.

The Meteor developers gave Blaze to the community to take care of it, decoupling it from Meteor. 

Even if it was a component of Meteor for a while, we do not recommend using Blaze in your applications.
From personal experiences, we noticed that, for mobile builds, Blaze is inferior to React in terms
of speed and stability. Especially for older mobile phones.

However, as times change, new View Layers appeared that support reactivity:
- [React](https://reactjs.org/)
- [Vue.JS](https://vuejs.org/)
- [Angular2](https://angularjs.org/)

Each of these has advantages and disadvantages. The question is: what should you choose?

Our recommendation is [React](https://facebook.github.io/react/)

The reasons for which we made this choice are:

- React is modular
- Even if it is a bit hard to use at first, using React gets much easier once you get the hang of it, and the development process becomes easy and intuitive
- React is lightning Fast 

However, at this stage(Q1 2018), Vue.JS is looking very promising and it is gaining a lot of momentum. 
Who will win? It's the survival of the fittest. Time will decide. For now, pick a technology and master it. 

Also, keep in mind the fact that many companies out there still use software that was written 50 years ago in COBOL. 
Why ? Because it gets the job done!
