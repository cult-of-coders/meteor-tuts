---
title: Tracker
description: The Meteor's way of building reactive apps
disqusPage: 'Chapter 1: Tracker'
---

## Let's talk about...a tracking system

Don't freak out! We're not tracking anyone, we're not teaching you how to become the Big Brother !

Let's now explore a nice concept for Meteor, the Tracker system.

Q: I'm getting sick and tired of this. I'm a web dev, I don't want to see things in my console no more..

A: Man, it's like Mister Myiagi trying to teach Karate Kid, before we reach the level on making everything work in a page, you'll be a master and you'll flow through it,
bare with me, this is the best way to take you to the learning curve.

Let's make sure you have the right packages.
```js
// terminal
meteor add tracker reactive-var
```

Now open up your browser:

```js
// browser console
var a = new ReactiveVar('a default value');
a.get() // will display 'a default value'
a.set('some other value')
a.get() // will display 'some other value'
```

Nothing fancy, what's the deal with this ?

ReactiveVar is a **reactive-data source** and you can follow it's changes. To be able to follow the changes we'll be using our infamous *Tracker*

```js
var computation = Tracker.autorun(() => {
    // this is the run function
    console.log(a.get())
})
```

Now each time you set some value to it, it will get *console.log*ed. Try it!

```js
a.set('something')
```

Doing:
```js
computation.stop()
```

Will stop the computation, so now if you set values to your data source, the run function will no longer execute.

Let's recap, we can track changes to reactive data sources, using Tracker. That's all you need to know for now. We needed
to explain this concept before we dive into the next step.

Tracker has many interesting things about it. Read more:
- https://docs.meteor.com/api/tracker.html

There are other reactive "watchers" out there, it's not the single one in the world, but this one is used by Meteor in many use-cases, and it just works.


## Homework

#### 1. Stop. It's too much!
Create a tracker that will stop after 5th time it entered the computation

#### 2. Reactive-Dict ?
Find out what's ReactiveDict and use it to track it's changes.