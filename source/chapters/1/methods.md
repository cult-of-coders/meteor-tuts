---
title: Methods
description: The RPC of Meteor.
disqusPage: 'Chapter 1: Methods'
---

## What is Method ?

Q: First off, what the heck is RPC ?

A: RPC stands for "Remote Procedure Call", in basic terms it means that you invoke "something" with (optionally) some arguments, and expect a response, OR AN ERROR! HAHAHAHAHAHA. Ok.

In Meteor, this is achieved by communicating with a websocket. You can disable websockets and it falls back to something else, but let's not get into
this right now, lets stay focused.

In a project we use methods to perform actions such as:
- Fetching the data
- Inserting Data
- Updating/Deleting Data
- Making API calls
- Any other dirty thing you can possibly think of

Methods are created server-side, but they can work client-side also (we'll explain it later)

## Creating a Method

```
// imports/api/donuts/methods.js
import { Meteor } from 'meteor/meteor';
import Donuts from '/imports/api/donuts/collection';

Meteor.methods({
    'create_a_donut': function () {
        return Donuts.insert({price: 100})
    }
})
```

## Calling a Method

Methods can be called from server or client. First let's try it in our *meteor shell*:

```
Meteor.call('create_a_donut')
```

OOOPS! 
```
Error: Method 'create_a_donut' not found [404]
```

Meteor has basically no idea about your method, you just created a file, inside imports. There is absolutely no link to it.

Remember the "/imports/startup/server/index.js" file, welp, that's where you have to import the methods.js file:
```
// file: /imports/startup/server/js
import '/imports/api/donuts/methods.js';
```

Now let the server refresh itself and try again you should see something like:
```
> Meteor.call('create_a_donut')
'MLx7SF79kXvAZuEyx'
```

That's because we do `return Donuts.insert({price: 100})` and insert() returns the newly created _id,
and what you return in the method body is returned to the caller. You can return anything you want, objects,
arrays, strings, dates. Back scene, data is serialized to [EJSON](http://docs.meteor.com/api/ejson.html) then deserialized where it's called.

Now let's finally open that browser. http://localhost:3000 open it.
Now open your browser's console. We recommend Chrome, but you can use anything you want.

```
Meteor.call('create_a_donut')
```

Hmm... I got no feedback here. Let me check the database. Oops, it's there! What's going on ?

Well, client-side when you do a call, you don't get the answer instantly, because it has to communicate through the websocket with the server,
the server needs to do it's thingie and return the results, this can take few miliseconds. This is why we need to provide a callback:

```
Meteor.call('create_a_donut', function (err, res) {
     console.log(res);
})
```

Ok, now it works, but what is "err" ? Why is it like this ? 
The why is explained here: http://fredkschott.com/post/2014/03/understanding-error-first-callbacks-in-node-js/

## Method Errors

If you do a `console.log(err)` you will see that it's undefined. Because the server did not throw any error while handling your method.

Let's trigger an error:
```
// imports/api/donuts/methods.js
import { Meteor } from 'meteor/meteor';

Meteor.methods({
    'create_a_donut': function () {
        throw "I don't wanna!"; // throw accepts any type argument, not only string
    }
})
```

Now if we do this in the shell:
```
Meteor.call('create_a_donut')
```

We will receive a simple string with the error. Doesn't help us very much, we don't know if it's an actual response or an error. However,
don't panic and don't get confused, you can use server-side callbacks as well:

```
Meteor.call('create_a_donut', function (err, res) {
    console.log(err, res);
})
```

Now let's move a bit to the client. The reason I insist in this, it's because you'll encounter a lot of errors and identifying and treating them
is critical to any web developer.

Make the same call as above for the client.

You should get something like:
```
errorClass {
    details: undefined
    error: 500
    errorType: "Meteor.Error"
    message: "Internal server error [500]"
    reason: "Internal server error"
}
```

Not very helpful is it ? You have no clue about what the error is about on the client, unlike the server.

This is why, in order to throw errors that are descriptive for the client as well we need to use [Meteor.Error](https://docs.meteor.com/api/methods.html#Meteor-Error)

```
Meteor.methods({
    'create_a_donut': function () {
        throw new Meteor.Error('error', 'I do not really want it', {
            why: "I have eaten too many"
        });
        // First argument (error): is something very general, can be a number like 500, 404, 
        // Second argument (reason): is a description of the error
        // Third argument (details): is for providing more details about the error.
    }
})
```

Now if you do the call on the client:

```
Meteor.call('create_a_donut', function (err, res) {
    if (err) {
        console.log('There was an error: ', err);
        // in err object you have to (err.error, err.reason, err.details)
    } else {
        console.log('Wooho! No Errors');
    }
})
```

Makes sense ? Just keep this in mind, use Meteor.Error for throwing exceptions.

## Method Arguments 

Methods can ofcourse receive arguments (any kind of arguments):

```
// server
Meteor.methods({
    'create_a_donut': function (one, two, three) {
        return [one, two, three];
    }
})

// client or server
Meteor.call('create_a_donut', 'One', 'Two', {three: 'Arguments'}, function (err, res) {
    console.log(res);
})
```

Now, methods are very complex in functionality, however, we will get into them later in this or other chapters.


## Homework

1. Create a method that returns all donuts
```
Meteor.call('donuts.list', {price: {$gt: 200}, callback)
```

2. Create a method that returns all donuts with a set of filters that you pass in the client:
```
Meteor.call('donuts.list_filtered', {price: {$gt: 200}, callback)
```

3. Create a method that takes two arguments, _id and data, and *$set*s the data for the donut with that _id

