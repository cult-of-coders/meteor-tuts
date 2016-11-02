---
title: Publications & Subscriptions
description: The Meteor's way of building reactive apps
disqusPage: 'Chapter 1: Pub/Sub'
---

Hey there, you seem tenacious, it seems that Methods did not put you down, and you continued, I like that.

## Pub/Sub System

The first step in understand the pub/sub system in Meteor, is to understand what a pub/sub system is in the first place.

Basically, it's a communication system, where the "publishers" sends messages to the "subscribers".

Let me give you an example, you have a chat application, // TODO: make it work

## Publishing in Meteor

```
import { Meteor } from 'meteor/meteor';
import Donuts from '/imports/api/donuts/collection';

Meteor.publish('donuts', function () {
    return Donuts.find();
})
```

Hmm, wait, why didn't we returned the elements directly, where is `.fetch()` in this equation ?

It's because a publication needs to return:
1. A cursor
2. An array of cursors

You can create custom publications and enable reactivity for anything you want (not only collections), 
but this is for advanced stuff.

## What is a cursor ?

A good way to think about a cursor is to think of it as an "address". In the example above, we returned an address
to all Donuts.

A publication basically says to the client:
Hey man, I allow you access to this data.

Q: Wait, isn't that what methods also do ?

A: Yes, but with publication we benefit of reactivity and we'll discover how this works next.

Q: Ok, so when should I use methods and when pub/sub ?

A: ...

## Subscribing to a publication

So, publishing says "I give you access to this", subscribing says "I want access to that data".

You noticed that when we created our publication, we first passed a string, 'donuts'. And we need to use
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

## Accessing modules from the Client

In any client file you can import anything you want.
```
// file: /imports/startup/client/index.js
import '/imports/api/donuts/collection.js';
```

For now, we imported it in the `/imports/startup/client/index.js` file because it's the way of Meteor saying:
`Ok, I allow access to this module in the browser`

If that wasn't the case, we would have all of our modules accessible from the browser, which we definitely don't want.

```
// browser console:
const Donuts = require('/imports/api/donuts/collection.js').default
```

Wait what ? `require()` ? I thought we use imports.

The `import` syntax does not work in the browser, but we have the require function, which basically does the same thing, but a bit differently, 
and because we `export default` the collection, to access it we need to access the `default` key inside the returned object by require()

If we would've done something like:

```
// file: /imports/api/donuts/collection.js
... 
export {Donuts}
```

Then the require would look like:

```
// client-side
const Donuts = require('/imports/api/donuts/collection.js').Donuts
```

Confused yet?

Come on, it's not that hard, give it a try! Hint: try it without importing it in the `/imports/startup/client/index.js` file, see what happens.

The reason we don't dive into creating client-side files, is because we don't want to touch the `View Layer` just yet. That's something for the next chapter.

## Client-side Collections

```
const Donuts = require('/imports/api/donuts/collection.js').default
```

It's got the same API. You can do insert(), update(), remove() but we will discuss later why this is not recommended, even if it is possible,
and it may seem like the easiest way to do, in the long-run it is not, for now we will keep the focus only on finding elements

Now that we got access to our collection, let's do:
```
// Meteor, give me my donuts:
Meteor.subscribe('donuts')

// wait for 1-2 seconds
Donuts.find().fetch()
```

Now go open a *meteor shell* and try inserting a new document in Donuts collection. Come back to the browser, and do the fetch() again.

You should see your element there!
Wow! Cool. But you mentioned something about reactivity ?

Cursors are reactive data sources, and we can track reactive data sources using the tracker:

```
Tracker.autorun(() => {
    console.log(Donuts.find().fetch());
})
```

Now everytime something in the "address" changes, like a new element is inserted, or updated, or removed. Meteor will communicate that to you, because you subscribed to that, and the cursor will change.

This will allow you to build reactive web pages with absolute ease. You simply track changes, and when changes are made, you do something in the UI.

This is the most barebones, straight forward way of showing you how Meteor's reactivity works. 
It is magical, and it the way it works is absolutely genious, but! we shouldn't really discuss this right now,
you don't have to understand how electricity works in order to use it right ?

This chapter's focus is on getting you going with Meteor.



 





