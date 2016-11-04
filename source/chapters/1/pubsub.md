---
title: Publications & Subscriptions
description: The Meteor's way of building reactive apps
disqusPage: 'Chapter 1: Pub/Sub'
---

Hey there, you seem tenacious, it seems that Methods & Tracker did not put you down, and you continued, I like that.

## Pub/Sub System

The first step in understand the pub/sub system in Meteor, is to understand what a pub/sub system is in the first place.

Basically, it's a communication system, where the "publishers" send messages to the "subscribers".

In Meteor we use this to have reactive data, meaning when something happens, like a new donut is added, I can see it live in my web-page.

## Publishing in Meteor

**Publishing** = "Hey bro, I will give you access to this data."

```
// file: /imports/api/donuts/publication.js
import { Meteor } from 'meteor/meteor';
import Donuts from '/imports/api/donuts/collection'; // no .js ? yep, works like that too!

Meteor.publish('donuts', function () {
    return Donuts.find();
})

// file: /imports/startup/server/index.js
import '/imports/api/donuts/publication.js'; // the server needs to know about it!
```

Hmm, wait, why didn't we returned the elements directly, where is `.fetch()` in this equation ?

It's because a publication needs to return:
1. A cursor
2. An array of cursors

## What is a cursor ?

A good way to think about a cursor is to think of it as an "address". In the example above, we returned an address to all Donuts.

Q: Wait, isn't that what methods also do ?

A: Yes, but with publication we benefit of reactivity and we'll discover how this works next.

Q: Ok, so when should I use methods and when pub/sub ?

A: You'll use pub/sub when you want to see the data changing live for you. Methods for the rest.

## Subscribing to a publication

**Subscribing** = "Hey bro, I want access to the data you are offering me."

You noticed that when we created our publication, we first passed a string, 'donuts'. We must use
that same string in order to subscribe to it.

Go to http://localhost:3000 and open your console:
```
var handler = Meteor.subscribe('donuts');
```

The handler contains:
- subscriptionId : which is a unique identifier for your subscription
- ready() : a function that returns true, if the subscription is ready
```
handler.ready(); // should return true
```
- stop() : stop the subscription

When a subscription is ready, basically it means that the server got your request, and it will fill the data.

How do we get the data is very very interesting, but we need to understand Client-Side collections first. But before that! 
Let's see how we can access these modules from our browser.

## Nasty Globals

We meet again! Darn you globals. This time we need to access our DonutsCollection on the client.

This time for the client: [Nasty Globals](/chapters/1/collections.html#Nasty-Globals)

## Client-side Collections

It's got the same API. You can do insert(), update(), remove() but we will discuss later why this is not recommended, even if it is possible.

If you still want to do it like that because it's easier, fine by me. You could try it:
```
DonutsCollection.insert({})
```

This works because you have `insecure` package installed in Meteor. `meteor remove insecure` and this will no longer work.

Now that we got access to our collection, let's do:
```
// Meteor, give me my donuts:
Meteor.subscribe('donuts')

// wait for 1-2 seconds
DonutsCollection.find().fetch()
```

```
// meteor shell, or browser (because we have insecure package)
DonutsCollection.insert(somethingThatYouWant)
```

```
// browser console
DonutsCollection.find().fetch()
```

## Seeing changes live
You should see your elements here. Wow! Cool. But you mentioned something about reactivity ?

Cursors are reactive data sources, and we can track reactive data sources using the tracker:

```
Tracker.autorun(() => {
    console.log(DonutsCollection.find().fetch());
})
```

Now everytime something in the "address" changes, like a new element is inserted, or updated, or removed. Meteor will communicate that to you, because you subscribed to that, and the cursor will change.

This will allow you to build reactive web pages with absolute ease. You simply track changes, and when changes are made, you do something in the UI.

This is the most barebones, straight forward way of showing you how Meteor's reactivity works. 

It is magical, and it the way it works is absolutely genious, but! we shouldn't really discuss this right now,
you don't have to understand how electricity works in order to use it right ?

## Managing the subscription

```
var handle = Meteor.subscribe('some_publication');
handle.stop(); // will stop listening to changes
handle.ready(); // will return true/false if the subscription is ready
```

Just keep in mind: when you have handle.ready() true, if you do super quick `.find().fetch()` might be empty, that's because
ready() does not mean that you got all the data, it means that the connections was established, and Meteor will pump the data there.

 





