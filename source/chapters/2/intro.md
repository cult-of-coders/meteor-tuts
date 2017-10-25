---
title: 'The View Layer'
description: Understanding how to build interfaces
disqusPage: 'Chapter 2: Intro'
---

## Introduction

In the previous chapter we learned how to harness Meteor's powers. But we didn't see anything actually happening in the UI (The View Layer).

Meteor is a data-system more than anything else, ofcourse it gives you some snacks to help you
code faster: The User System, Emails, much more. But the View Layer it's up to you.

Don't get this the wrong way, Meteor can be integrated with any View Layer/Template Engine/UI Framework you want to use. 

## Brief history

When Meteor started there was no stable solution for building reactive views. They had to invent a way.
This is how Blaze was born. [http://blazejs.org/](http://blazejs.org/). In Q4 2015, they decided to drop
support for Blaze, not because it wasn't good, but because there were other teams that focused only on the
view layer. You cannot expect one company to do everything. So they began decoupling Blaze from Meteor.

Even if Blaze is one of the most friendliest View Layers you'll ever meet, they gave it to the community
to take care of it, decoupling it from Meteor. This does not mean you should not use Blaze. Blaze is rock-solid and battle-tested.

However, from a personal opinionated experience, we noticed that for mobile builds, Blaze is inferior to React, in terms
of speed and stability. Especially for older mobile phones.

However, times change, new View Layers appeared that support reactivity such as:
- React
- Vue.JS
- Angular2
- etc

Each has advantages and disadvantages. So question now is what to choose ?

Our current recommendation is: [React](https://facebook.github.io/react/)

Reasons for this choice:

- Modular
- A bit hard to use at first, but much easier once you get the hang of it, it becomes easy and intuitive
- Lightning Fast 
- Maintained by Facebook

However, at this stage, (Q4 2016) Vue.JS is looking very promising and gaining a lot of momentum. 
Who wins? It's a race. It's the survival of the fittest. Time will decide. For now, pick a technology
and master it. 

Keep in mind, many companies out there still use software written 50 years ago in COBOL. Why ? Because they work and to the job.