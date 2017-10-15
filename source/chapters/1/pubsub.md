---
title: Publications & Subscriptions
description: Building reactive apps with Meteor
disqusPage: 'Chapter 1: Pub/Sub'
---

## The Pub/Sub System - going live!

To be able to understand what we mean when we say Pub/sub in Meteor, we need to explain to you what a pub/sub system is in the first place. 

The explanation is fairly straightforward: a pub/sub system is a communication system, where the "publishers" send messages 
to the "subscribers". Which is why, in Meteor, you will meet, very frequently, the term of **"reactive data"**.

Having this feature available in Meteor means that we can build **live applications**. If we go to our donut example,
and we add a new donut, all the users that are **subscribed** to that data will see the changes made to the data in real time.
Just like you would see on Facebook !

## Publishing in Meteor

So how would we explain publishing in a very simple manner ? Well, let's assume you're talking to a friend about you application,
 and you want to give him access to your data. I other words, you are going to make that data **public** for him.
 As such,  **publishing** would be equivalent to saying to him "I will give you access to this data !".
 
 Let's start by inserting code into "/imports/api/donuts/publication.js":
```js
import { Meteor } from 'meteor/meteor';
import Donuts from '/imports/api/donuts/collection'; 

Meteor.publish('donuts', function () {
    return Donuts.find();
})
```

And now let's put some code into "/imports/startup/server/index.js" and let the server know about the code we inserted 
into "publication.js"":
```js
import '/imports/api/donuts/publication.js'; // the server needs to know about it!
```

It's a basic rule that a publication needs to return 2 things:
1. A cursor
2. An array of cursors

It is because of this rule that we aren't using `.fetch()` in our code.

## What is a cursor ?

A good way to think about a cursor is to think of it as if you would think of an "address". 
In the example above, we returned an address to all Donuts.

Q: Wait, isn't that also what methods do ? <br />
A: Yes, but by using publications we benefit from reactivity.

Q: Ok, so when should I use methods and when should i use pub/sub ? <br />
A: You should use pub/sub when you want your data changes to be **live**. Otherwise you should stick to using methods.

## Subscribing to a publication

We have now published the data changes, but how are the users going to know about the data that we've made available for them ?
Well, that's where subscriptions come in ! If we are to use the "friend" analogy that we have used to explain **publishing*,
the friend to which we have given access to the data will say: "Hey! I want access to the data you are offering me."

You noticed that when we created our publication, we first passed a string to it in the code-'donuts'. 
We must use that same string in order to subscribe to the publication.

Go to http://localhost:3000 and the console in your browser's developer tools. Then, type into it:
```js
var handler = Meteor.subscribe('donuts');
```
<br />

```js
handler.ready(); // should return true
```
<br />

```js
handler.stop(); // should return true
```

The variable *handler* contains:
-a *subscriptionId*, which is a unique identifier for the subscription
- *ready()*, which is a function that returns true, if the subscription is ready
- *stop()*, which is a function that will stop the subscription

When a subscription is ready, it means that the server got your request, and it give the specified user access to the **live** data.

How the data is served to us is a very interesting process, but in order to delve into that process, we first have to 
have a good understanding of Client-Side collections.

## Nasty Globals...again

Again, we must sacrifice our pride for a higher purpose and use global variables. 
This time we need to access our *DonutsCollection* on the client-side.

we've talked about [Nasty Globals](/chapters/1/collections.html#Nasty-Globals) before!

## Client-side Collections

The collections we are using on the client-side have the same API  as the collections from the server-side.
We can do insert(), update(), remove() but we will explain later why this is not recommended, even if it is possible.

If you still want to do it like that because, just because it's easier, it's fine, but it is not recommended that you do so!
We could try it, just this time:
```js
DonutsCollection.insert({})
```

This works because you have the `insecure` package installed in Meteor.
 Running `meteor remove insecure` in your terminal will stop this from working.

Now that we got access to our collection, let's get those *donuts*:
```js
// Meteor, give me my donuts !
Meteor.subscribe('donuts')

// wait for 1-2 seconds while Meteor is getting the donuts...
DonutsCollection.find().fetch()
```

Now, because we can use an insecure package, let's use the Meteor shell,:
```js
DonutsCollection.insert(somethingThatYouWant)
```
<br />

Or the browser console:
```js
// browser console
DonutsCollection.find().fetch()
```

## Watching the changes while they happen

The cursors that we've used earlier are *reactive data sources*, and we can track reactive data sources by using the tracker:
```js
Tracker.autorun(() => {
    console.log(DonutsCollection.find().fetch());
})
```

Now everytime something in the "address" changes, like a new element is inserted, or updated, or removed, Meteor will communicate that to you, 
because you subscribed to that publication, and the cursor will change, to a new address.

This will allow you to build reactive web pages with ease. You simply track changes, and when changes are made, you 
"react" with some UI changes ( see what we did there ? "react"->ReactJS... No ? Ok... :( Let's move on!).

This is the most simplistic way of showing you how Meteor's reactivity works. 

It is like magic !

## Managing subscriptions

```js
var handle = Meteor.subscribe('some_publication');
handle.stop(); // will stop listening to changes
handle.ready(); // will return true/false if the subscription is ready
```

Just keep in mind: when **handle.ready()** returns *true*, if it is followed by a `.find().fetch()`, the results might be empty.
That's because *ready()* returning a *true* value does not mean that you got all the data. 
Instead, it means that the connections was established, and Meteor will pump the data there.

 





