---
title: Publications & Subscriptions
description: Building reactive apps with Meteor
disqusPage: 'Chapter 1: Pub/Sub'
---

## Going Live

To be able to understand what we mean when we say Pub/sub in Meteor, we need to explain to you what a pub/sub system is in the first place. 

The explanation is fairly straightforward: a pub/sub system is a communication system, where the "publishers" send messages 
to the "subscribers". Which is why, in Meteor, you will meet, very frequently, the term of **"reactive data"**.

Having this feature available in Meteor means that we can build **live applications**. If we go to our post example,
and we add a new post, all the users that are **subscribed** to that data will see the changes made to the data in real time.

## Publishing in Meteor

So how would we explain publishing in a very simple manner ? Well, let's assume you're talking to a friend about your application,
 and you want to give him access to your data. In other words, you are going to make that data **public** for him.
 As such,  **publishing** would be equivalent to saying "I will give you access to this data !".
 
```js
//file /imports/api/posts/publications.js
import { Meteor } from 'meteor/meteor';
import Posts from '/imports/db/posts/collection';

Meteor.publish('posts', function () {
    return Posts.find();
})
```

Now, don't forget to import it in `/imports/startup/server/index.js`!

It's a basic rule that a publication needs to return 2 things:
1. A cursor
2. An array of cursors

It is because of this rule that we aren't using `.fetch()` in our code.

## What is a cursor?

A good way to think about a cursor is to think of it as if you would think of an "address". 
In the example above, we returned an address to all posts.

Q: Wait, isn't that also what methods do ? <br />
A: Yes, but by using publications we benefit from reactivity.

Q: Ok, so when should I use methods and when should i use pub/sub ? <br />
A: You should use pub/sub when you want your data changes to be **live**. Otherwise you should stick to using methods.

## Subscribing

We have now published the data changes, but how are the users going to know about the data that we've made available for them ?
Well, that's where subscriptions come in ! If we are to use the "friend" analogy that we have used to explain **publishing**,
the friend to which we have given access to the data will say: "Hey! I want access to the data you are offering me."

You noticed that when we created our publication, we first passed a string to it in the code called 'posts'.
We must use that same string in order to subscribe to the publication.

Go to http://localhost:3000 and the console in your browser's . Then, type into it:
```js
var handler = Meteor.subscribe('posts');
```

```js
handler.ready(); // / will return true/false if the subscription is ready
```

```js
handler.stop(); // will return true/false and stop listening to changes
```

The variable *handler* contains:
-a *subscriptionId*, which is a unique identifier for the subscription
- *ready()*, which is a function that returns true, if the subscription is ready
- *stop()*, which is a function that will stop the subscription

When a subscription is ready, it means that the server got your request, and it gave the specified user access to the **live** data.

How the data is served to us is a very interesting process, but in order to dive into that process, we must first have to
have a good understanding of Client-Side collections.

## Observing Changes

Now every time something in the "address" changes, like a new element is inserted, or updated, or removed, Meteor will communicate that to you, 
because you subscribed to that publication, and the cursor will change, to a new address.

## Managing subscriptions

Just keep in mind: when **handle.ready()** returns *true*, if it is followed by a `.find().fetch()`, the results might be empty.
That's because *ready()* returning a *true* value does not mean that you got all the data. 
Instead, it means that the connections was established, and Meteor will pump the data there.

If you want to read more about how pub/sub works, click [here](https://docs.meteor.com/api/pubsub.html)

In order to see this how fully works in a practical way, you could clone our [github repository](www.github.com) and on `/posts/reactive` you can see the process.

 





