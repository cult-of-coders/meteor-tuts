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
Unlike MySQL, in MongoDB you don't have to CREATE TABLE, or CREATE FIELD in order to have a field, because MongoDB 
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

You've already tried to run your project and now you're getting a ton of errors,right ? 
Do not fear, we will solve them, and then we'll run the project !

## Nasty Globals

So let's start solving those errors,right ?
Since our Donuts are isolated in their own module, we need to gain access to them. For that purpose, and only in this tutorial, we will use some very nasty global variables.
Don't try this in a production project ! It fits into the category "Bad practices" !!

Go to "/imports/startup/server/index.js" and type this in the source file:

```js
import './nasty-globals.js'
```
Now, let's move over to "/imports/startup/server/nasty-globals.js":

```js
import Donuts from '/imports/api/donuts/collection.js';
```

As a reminder: because we used export default, when we import, we can import as any name:
```
import AnyNameIWantWillBeTheSame from '/imports/api/donuts/collection.js'
```
```js
DonutsCollection = Donuts 
```
Because we did not use var, let, const before it, it's a global variable !

## Inserting data 
Since we have gained access to our donuts in the file '/imports/startup/server/nasty-globals.js', we need to continue in there.
This is how we will do our first insert:
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

We inserted a string, array, number, date, boolean, and even an object. We can have array of objects, array of dates,
array of array of array of objects that contains an array of array of dates (you get the idea). The sky's the limit ! 

## Result

What effect does all of this have ? Let's run it an see: 
```
meteor run
```
No errors,right ? :)
Open another terminal, go to the same folder, and run this command:
```
meteor shell
```

Meteor shell is like a console for the server. Type this into the console:
```js
console.log('hello shell');
```

You won't see it where you types it, however you will notice it will appear in the console from where meteor was started. Keep that in mind as it will help you in the future.

You may notice that you see something like:
```
'MLx7SF79kXvAZuEyx'
```

That's because .insert() returns the newly created id. (Which is stored as _id in the Document in the database)


## Finding data
Now let's fetch the data from the database:
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








