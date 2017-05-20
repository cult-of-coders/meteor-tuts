---
title: Methods
description: The RPC of Meteor.
disqusPage: 'Chapter 1: Methods'
---

## RPC's - a way to express yourself

Well, you probably already ask yourself what is an RPC. Even if you have already googled the term, allow us to explain to you what an RPC is. 

First of all, RPC stands for "Remote Procedure Call", which basically means that you invoke "something" with some arguments ( only if you want to use arguments, of course).
Then you expect a response, just like you would do when in a conversation. If you get an error, then your "conversation" might not be so productive, so check your code :).

Having a conversation in Meteor is quite simple.
In Meteor, the usual way of doing this, and the best, is by communicating through a websocket with the server. 
if you disable websockets, it falls back to something else, which we won't talk about in this tutorial

So what actions we can do in our "conversations" with RPC's ?
- Fetching data
- Inserting Data
- Updating/Deleting Data
- Making API calls
- Anything you can possibly think of

Methods are created server-side, and they can be called from the client or from the server as well.
In the `meteor shell`, since we've already established where it is located, it means we are calling RPC's from the server.

We can also have methods on the client-side, which can be used for Optimistic UI, something we won't be discussing.


## Let's create a method( or RPC)
Let's keep using the sample application we created at the beginning of the tutorial (  yes, i still love donuts! ), and in the file *imports/api/donuts/methods.js* we'll write:

```
import { Meteor } from 'meteor/meteor';
import Donuts from '/imports/api/donuts/collection';

Meteor.methods({
    'create_a_donut': function () {
        return Donuts.insert({price: 100})
    }
})
```

## Now...how to use it ?

Methods can be called from the server-side or from the client-side. Let's try it first in the *meteor shell*:

```js
Meteor.call('create_a_donut')
```

BANG!
Another error ! But we've already dealt with errors before, so fear not...you caninux shutd do this ! 
```
Error: Method 'create_a_donut' not found [404]
```

Meteor has basically no idea that the method you just wrote exists. So you just created a file inside imports, but there is absolutely no link to it.
So, as a golden rule, if you don't create a link to your file, it's just as ineffective as if it wouldn't exist.

How do you create such a link ?
Well, in the "/imports/startup/server/index.js" file, import the methods.js file:
```js
import '/imports/api/donuts/methods.js';
```

Now let the server refresh itself ( it does that when you change the code in your application ).
After calling the method in the *meteor shell*, it will respond to you with a message that looks like this:
```
> Meteor.call('create_a_donut')
'MLx7SF79kXvAZuEyx'
```

That's because of this line:
 ```js
 return Donuts.insert({price: 100})
  ```
Let's ignore the "return" part for now and focus on the "insert" for a little bit.
With "insert()", we just inserted content into the database. As a consequence, the occurrence of this operation will determine the system
to "spit out"  the newly created _id, thus giving us a way of identifying the operation.
Now let's get back to the "return()" part !
What you return in the method body is returned to the caller. 
You can return anything you want: objects, arrays, strings, dates, and, you guessed it-id's !
In the background, the data is serialized to the [EJSON](http://docs.meteor.com/api/ejson.html) format, then deserialized where it's called.

Now let's view the results of our work in the browser.
Open your web browser and navigate to *http://localhost:3000* .
Now open your browser's console by typing Ctrl+Shift+I in Google Chrome. 
We recommend Google Chrome, as it is the best, but feel free to test into other browsers as well.
It might even give you some perspective as to how you could optimize an application to work well with multiple browsers !

```js
Meteor.call('create_a_donut')
```

Hmm... I got no feedback here. Let me check the database. Oops, it's there! What's going on ?

Well, client-side when you do a call, you don't get the answer instantly, because it has to communicate through the websocket with the server,
the server needs to do it's thingie and return the results, this can take few miliseconds. This is why we need to provide a callback:

```js
Meteor.call('create_a_donut', function (err, res) {
     console.log(res);
})
```

Ok, now it works, but what is "err" ? Why is it like this ? 
The why is explained here: http://fredkschott.com/post/2014/03/understanding-error-first-callbacks-in-node-js/

## Method Errors

If you do a `console.log(err)` you will see that it's undefined. Because the server did not throw any error while handling your method.

Let's trigger an error:
```js
// imports/api/donuts/methods.js
import { Meteor } from 'meteor/meteor';

Meteor.methods({
    'create_a_donut': function () {
        throw "I don't wanna!"; // throw accepts any type argument, not only string
    }
})
```

Now if we do this in the shell:
```js
Meteor.call('create_a_donut')
```

We will receive a simple string with the error. Doesn't help us very much, we don't know if it's an actual response or an error. However,
don't panic and don't get confused, you can use server-side callbacks as well:

```js
Meteor.call('create_a_donut', function (err, res) {
    console.log(err, res);
})
```

Now let's move a bit to the client. The reason I insist in this, it's because you'll encounter a lot of errors and identifying and treating them
is critical to any web developer.

Make the same call as above for the client.

You should get something like:
```js
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

```js
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

```js
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

```js
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

#### 1. Get me all the donuts
```js
Meteor.call('donuts.list', callback)
```

#### 2. Give me some donuts
```js
Meteor.call('donuts.list_filtered', {price: {$gt: 200}, callback)
```

#### 3. Updating
Create a method that takes two arguments, _id and data, and *$set*s the data for the donut with that _id.

```js
Meteor.call('donuts.list_filtered', donutId, {price: 1000})
```

