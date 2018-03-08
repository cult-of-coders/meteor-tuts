---
title: Methods
description: The RPC of Meteor.
disqusPage: 'Chapter 1: Methods'
---

## RPC's - a way to express yourself

Well, you are probably already asking yourself what is an RPC. Even if you have already googled the term, allow us to explain to you what an RPC is. 

First of all, RPC stands for "Remote Procedure Call", which basically means that you invoke "something" with some arguments ( only if you want to use arguments, of course).
and you expect a response, just like you would do when in a conversation.

In Meteor, the usual and most recommended way of doing this, is by communicating through a websocket with the server.

So what actions we can do with RPC's ?
- Fetching data
- Inserting Data
- Updating/Deleting Data
- Making API calls
- Anything you can possibly think of

## Let's create a method(or RPC)

```
// file /imports/api/methods.js
import { Meteor } from 'meteor/meteor';

Meteor.methods({
    'find.random_number' (min, max) {
        if (!min) min = 0;
        if (!max) max = 100;
        const generatedNumber = Math.random() * (max - min) + min;
        return Math.floor(generatedNumber);
    },
})

```

# Important
Do not forget to  `` import '/imports/api/methods.js' `` in the server's startup file: ` /imports/startup/server/index.js ` otherwise meteor won't know of the existence of this file

## Now...how to use it ?

Methods can be called from the server-side or from the client-side. Let's try it in the browser's console:

```js
Meteor.call('find.random_number', 25, 50 (err, result ) => {
   console.log(err); // in case an exception is thrown, then this will contain information about the error
   console.log(result); // this will contain a random number between 25 and 50
})
```

To fully understand the reason why we added 2 arguments `err` and `result`, go ahead and read the explanation [here](http://fredkschott.com/post/2014/03/understanding-error-first-callbacks-in-node-js/)

## Errors in methods

Let's cause an error and see what happens:
Type this code in the */imports/api/methods.js* file:
```js
// file /imports/api/methods.js
import { Meteor } from 'meteor/meteor';

Meteor.methods({
   'method.checkString'(myString) {
           if (myString === 'exception') {
               throw new Meteor.Error(500, 'An error has occurred', 'You are not allowed to enter this string');
           }

           return true;
       }
       // Arguments for throw new Meteor.Error:
           // The first argument (error): is something very general, can be a number like 500, 404,
           // The second argument (reason): is a description of the error
           // The third argument (details): is for providing more details about the error.
})
```

Now, make a method call in your browser's console to `method.checkString` and catch the exception:

```js
Meteor.call('method.checkString', 'exception', (err, result) => {
    console.log(err);
})
```