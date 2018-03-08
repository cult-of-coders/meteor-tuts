---
title: Tracker
description: Building reactive apps with Meteor
disqusPage: 'Chapter 1: Tracker'
---

## Introducing the tracking system

What is a tracking system ?

The tracker is one of the concepts that sets Meteor apart from other frameworks out there. As such, before we show you
what it's all about, we need to make sure you have all the packages that you need installed on your machine.
For that, run this in your terminal:
```js
meteor add tracker reactive-var
```

Now open up your browser and type this into the browser console 
(the browser console is available for you in the "developer tools" menu of your browser):
```js
var a = new ReactiveVar('a default value');
a.get() // will display 'a default value'
a.set('some other value')
a.get() // will display 'some other value'
```

That does not seem to be very complicated, but you might be asking yourself "what's the deal with this ?". 
Allow us to explain it step by step, so you can understand it!

**ReactiveVar** is a reactive-data source and because of that, you can follow its changes at runtime. 
To be able to do that, we will be using the *Tracker* feature that got mentioned earlier in the tutorial:
```js
var computation = Tracker.autorun(() => {
    // this is the function that runs it
    console.log(a.get())
})
```

Now each time you set some value to the **a** variable, it will get logged into the console. Let's try it:
```js
a.set('something')
```

To stop the execution, use:
```js
computation.stop()
```

Now, if you give values to your data source, the **run** function's execution will be stopped.

So, basically:
With the **Tracker**, we can track changes occurring to reactive data sources, in real time. 
That's everything you need to know for now in order to use this feature.

What we presented for you was a brief introduction, the **Tracker** tool is way more complex than that. You can read more about them [here](https://docs.meteor.com/api/tracker.html).

At its core, tracker is a reactive "watcher". And it is not unique !
However, this one is very well integrated with Meteor, and this brings down the integration issues you might face when 
working on projects, which is why we are using it !

## Practice makes perfect

#### 1. Stop. It's too much!
Create a tracker that will stop after the 5th change to a reactive variable.

#### 2. Reactive-Dict ?
Find out what's ReactiveDict and use it to track changes.