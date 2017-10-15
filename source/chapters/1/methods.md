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
If you disable websockets, it falls back to something else, which we won't talk about in this tutorial.

So what actions we can do in our "conversations" with RPC's ?
- Fetching data
- Inserting Data
- Updating/Deleting Data
- Making API calls
- Anything you can possibly think of

Methods are created server-side, and they can be called from the client or from the server as well.
In the `meteor shell`, since we've already established where it is located, it means we are calling RPC's from the server.

We can also have methods on the client-side, which can be used for [Optimistic UI](https://uxplanet.org/optimistic-1000-34d9eefe4c05).


## Let's create a method( or RPC)
Let's keep using the sample application we created at the beginning of the tutorial (yes, i still love donuts), and in the file *imports/api/donuts/methods.js* we'll write:

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
Another error ! But we've already dealt with errors before, so fear not...you can do this ! 
```
Error: Method 'create_a_donut' not found [404]
```

Meteor has basically no idea that the method you just wrote exists. You just created a file inside imports, but there is absolutely no link to it.
So, as a golden rule, if you don't create a link to your file, it's just as ineffective as if it would not be there.

How do you create such a link ?
Well, in the "/imports/startup/server/index.js" file, import the methods.js file:
```js
import '/imports/api/donuts/methods.js';
```

Now let the server restart itself ( it does that when you change the code in your application ).
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
to "spit out"  the newly created _id, thus giving us a way of identifying the operation. So, as a rule of thumb, each transaction generates and _id !
Now let's get back to the "return()" part !
What you return in the method body is returned to the caller. 
You can return anything you want: objects, arrays, strings, dates, and, you guessed it-id's !
In the background, the data is serialized to the [EJSON](http://docs.meteor.com/api/ejson.html) format, then deserialized into a more user-friendly format where it's called.

Now let's view the results of our work in the browser.
Open your web browser and navigate to *http://localhost:3000* .
Now open your browser's console by typing Ctrl+Shift+I in Google Chrome or Command+Shift+I on a Mac. 
We recommend Google Chrome, as it is the best, but feel free to test into other browsers as well. Actually, when working
 on production projects, it is recommended that you test your code with the most used browsers out there.
It might even give you some perspective as to how you could optimize an application to work well with multiple browsers !

```js
Meteor.call('create_a_donut')
```

That's weird...nothing happened. 
Let's check the database. What's going on ?

Well, when you make a call from the client, you can't get the answer instantly, because your call initiates a "conversation" through a [websocket](https://en.wikipedia.org/wiki/WebSocket) with the server.
Then, the server needs to complete the operation you requested in that call and "get back to you" with the results.
This operation can take few milliseconds, which is why we need to provide a [callback](https://en.wikipedia.org/wiki/Callback_(computer_programming)) to our method:

```js
Meteor.call('create_a_donut', function (err, res) {
     console.log(res);
})
```

Now it works, but why did we use "err" ?  
You will find an explanation [here](http://fredkschott.com/post/2014/03/understanding-error-first-callbacks-in-node-js/).

## Errors in methods

If you try using `console.log(err)` in *Meteor Shell*, you will see that it's undefined. Because the server did not throw any error while handling your method.

Let's cause an error and see what happens:
Type this code in the *imports/api/donuts/methods.js* file:
```js
import { Meteor } from 'meteor/meteor';

Meteor.methods({
    'create_a_donut': function () {
        throw "I don't wanna!"; // notice that the "throw" statement accepts any type of argument, not just strings
    }
})
```

Now if we call the method we just created in the shell ( "initiate a conversation with the server", if you want to call it like that):
```js
Meteor.call('create_a_donut')
```

We'll get a response in the form of a string containing the error message. 
This doesn't help us very much, since we don't know if it's an actual response or an error. 
However, we ave another way for solving this, as we have at our disposal server-side callbacks as well:

```js
Meteor.call('create_a_donut', function (err, res) {
    console.log(err, res);
})
```

Now let's move to the client for a little bit. The reason we have to do this, and the reason for which we have been dealing with errors since the beginning is
 because, while working on projects you'll encounter a lot of errors and identifying and treating them is a critical part of being a good web developer.

After making the same operations we did on the server, to the client, you should get a response that looks like this:
```js
errorClass {
    details: undefined
    error: 500
    errorType: "Meteor.Error"
    message: "Internal server error [500]"
    reason: "Internal server error"
}
```

That's not very helpful, is it ? It' doesn't help you identify the error, nor does it give you tips about solving the error, unlike the message from the server.

This is why, in order to make the errors that originate from the client descriptive and comprehensive, we need to use [Meteor.Error](https://docs.meteor.com/api/methods.html#Meteor-Error)

```js
Meteor.methods({
    'create_a_donut': function () {
        throw new Meteor.Error('error', 'I do not really want it', {
            why: "I have eaten too many"
        });
        // The first argument (error): is something very general, can be a number like 500, 404, 
        // The second argument (reason): is a description of the error
        // The third argument (details): is for providing more details about the error.
    }
})
```

Now, if you do the call on the client:

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

So, from now on, to make errors comprehensible everywhere, use *Meteor.Error* for throwing exceptions.

## Arguments for our methods

Just like in a normal conversation, in which you have to use arguments, methods can receive arguments of any kind !

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

The fact that we can now use arguments with methods opens a world of possibilities to us. But this can make methods fairly complex, 
which is why, during the course of this tutorial, we will talk some more about methods.

## Work more to learn more

Now let's practice what we've learned in this section with a few simple exercises:

#### 1. Get me all the donuts
Try out this code, to get all the donuts from the database :
```js
Meteor.call('donuts.list', callback)
```

#### 2. Give me some donuts
Try out this code snippet, to get some donuts ( after all, moderation is good when consuming sweets):
```js
Meteor.call('donuts.list_filtered', {price: {$gt: 200}, callback)
```

#### 3. Updating
Test yourself !
Create a method that takes two arguments, *_id* and *data*, and *$set*s the data for the donut with that _id.

```js
Meteor.call('donuts.list_filtered', donutId, {price: 1000})
```

