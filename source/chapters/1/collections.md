---
title: Collections
description: How we store our data in Meteor.
disqusPage: 'Chapter 1: Collections'
---

## Let's talk data!

Meteor uses MongoDB as its default database system. Theoretically you can use any database you want, because
you have access to http://www.npmjs.com, therefore you have access to almost all the existing database drivers out there 
(a database driver is a program which implements a protocol for connecting to a database). 

## Show and tell comparison
Since everyone knows what MySQL is, let's compare MongoDB with it, so you can better understand why it's better for what we need. 
Unlike MySQL, in MongoDB you don't have to CREATE TABLE, or CREATE FIELD in order to to have a field, because MongoDB 
lets you structure your data the way you want. You still however need consistency across your data, which is why, later on,
we will teach you how to ensure this consistency at an application level, not at the database level.

Now we'd like to make some analogies, but you're going to need to be familiar with the syntax of MySQL at a basic level.
If you're not, read a little bit about it [here](https://www.tutorialspoint.com/mysql/).

Now let's make those analogies:
- DATABASE = Database (yes, the same name)
- TABLE = Collection (a list of data, a collection)
- ROW = Document (the actual list of data inside the collection)

I like donuts...despite knowing that they're unhealthy.
I love donuts so much that I want to store them in a database !

Firstly, let's create a new file in the project file structure we created earlier, by running these commands in the 
/imports/api folder:
```
mkdir donuts
cd donuts
touch collection.js
```

Now let's write some code in this file we have just created:

```js
import { Mongo } from 'meteor/mongo';

const Donuts = new Mongo.Collection('donuts');

export default Donuts;
```

What effect does this have ? Let's run it an see: 
```
meteor run
```
You're getting errors, right ? No problem ! Let's solve them !!
Open another terminal, go to the same folder, and run this command:
```
meteor shell
```

Meteor shell is like a console for the server-side. Some quick tips, if you do in the shell:
```js
console.log('hello shell');
```

You won't see it there, because the actual message is logged in the console of where meteor is started. Keep that in mind it will help you in the future.

## Nasty Globals

Ok so, now we want to gain access to our Donuts collection. Which is isolated in it's own module.
This is why, for this tutorial only, we are going to use globals. Never use globals, and don't say this tutorial taught you bad approaches!

```js
// file: /imports/startup/server/index.js
import './nasty-globals.js'

// file: /imports/startup/server/nasty-globals.js
import Donuts from '/imports/api/donuts/collection.js'; 

// Keep in mind: because we used export default, when we import, we can import as any name:
// import AnyNameIWantWillBeTheSame from '/imports/api/donuts/collection.js'

DonutsCollection = Donuts // we did not use var, let, const before it, so it's a global
```

## Insert

Now let's do our first insert:
```js
DonutsCollection.insert({
    color: 'pinkish-green', 
    flavors: ['mustard', 'onions'], 
    price: 999.0, 
    expiresAt: new Date(),
    isCommestible: false,
    ratings: {
        taste: 2,
        awesomeness: 10,
    }
})
```

Wow! We inserted a string, array, number, date, boolean, and even an object. We can have array of objects, array of dates,
array of array of array of objects that contains an array of array of dates (you get the idea). It's schemaless.


You may notice that you see something like:
```
'MLx7SF79kXvAZuEyx'
```

That's because .insert() returns the newly created id. (Which is stored as _id in the Document)


## Find
Now let's fetch this baby:
```js
DonutsCollection.find()
```

Now if you run that, you will be like: What the hell is that ?
That's a cursor! We'll explain later what's the deal with find() and fetch().

```js
DonutsCollection.find().fetch()
```

Ok now lets do a more complex search, for that let's add another donut:
```js
DonutsCollection.insert({price: 50, isCommestible: true});
```

Let's search our database:
```js
// Getting the commestible donuts
DonutsCollection.find({isCommestible: true}).fetch()

// Getting the donuts with a price over 200
DonutsCollection.find({price: {$gt: 200}}).fetch()
```

You have ability to use the selectors that MongoDB provides, here are all of them described:
https://docs.mongodb.com/manual/reference/operator/query/#query-and-projection-operators

There is also another argument that find() takes in, it's called "options":
```js
DonutsCollection.find({}, {
    sort: {price: -1} // sorts price in descending order
    limit: 2 // limits the results to 2
    fields: {price: 1, isCommestible: 1} // will only return the fields price and isCommestible
}).fetch()
```

There are other fields as well, but we don't want to get into many details just yet, if you simply cannot abstain yourself from the curiosity, check it out:
http://docs.meteor.com/api/collections.html#Mongo-Collection-find

## Update

What about updating ?

```js
DonutsCollection.update({price: 50}, {
    $set: {price: 51}
})
```

The first argument is "what you want to update", the second one is "how you update it". You can use stuff like "$inc" for incrementing, 
"$addToSet" if you want to add another disgusting ingredient in the "flavors" array.

The idea is you can do a lot of things, and there are many cool stuff, you can read more about it here:
https://docs.mongodb.com/manual/reference/operator/update/

## Remove

Removing is easy, the argument it takes is what to remove:
```js
DonutsCollection.remove({price: {gt: 1000}})
```

Keep playing with it, try new inserts, try new selectors, have a little bit of fun, you deserve it.

If you have a MongoDB visualizer, like "Robomongo" for Ubuntu, or any other tools out there http://lmgtfy.com/?q=mongodb+admin+software
You can connect to it by using "localhost" and port 3001 (but you must have Meteor started)


## Tips & Tricks

You can find/update/remove by using id as a string
```js
DonutsCollection.update('XXX', modifier)
// equivallent
DonutsCollection.update({_id: 'XXX'}, modifier)
```


## Homework

#### 1. Help mr John
Mr John wants to find all donuts that have a price bigger than 200 and they have at least one flavor!
Help him with a query! [Hint](https://docs.mongodb.com/manual/reference/operator/query/size/#op._S_size)

#### 2. Price Change
It's donut season. Run an update that increases all prices with 100! 
[Hint](https://docs.mongodb.com/manual/reference/operator/update/inc/#up._S_inc)

#### 3. Reality Hit
People did not like the new prices, so create a query that will decrease the prices for the most 3 expensive 
donuts with 10%. [Hint](https://docs.mongodb.com/manual/reference/operator/update/mul/#up._S_mul)








