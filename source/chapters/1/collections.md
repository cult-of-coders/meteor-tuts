---
title: Collections
description: How we store our data in Meteor.
disqusPage: 'Chapter 1: Collections'
---

## Let's talk data!

Meteor uses MongoDB as its default database system. Theoretically you can use any database you want, because
you have access to *http://www.npmjs.com*, therefore you have access to almost all the existing database drivers out there 
(a database driver is a program which implements a protocol for connecting to a database). 

## Show and tell comparison
Since everyone knows what MySQL is, let's compare MongoDB with it, so you can better understand why it's better for what we need. 
Unlike MySQL, in MongoDB you don't have to CREATE TABLE, or CREATE FIELD in order to create a table or a field, because MongoDB 
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
import Donuts from '/imports/api/donuts/collection.js'
```

As a reminder: because we used export default, when we import, we can import as any name:
```js
//import Any_Name_I_Want_Will_Have_The_Same_Effect from '/imports/api/donuts/collection.js'
DonutsCollection = Donuts 
```
Because we did not use var, let, const before it, it's a global variable !

## The Meteor console
In Meteor, we have at our disposal a console for the server side of the application.
We can use this for debugging purposes.
To use it, go to the file in which you have created your application, and open a terminal.
Type the following command into your terminal:
```
meteor shell
```

This will open a terminal from which you can test code snippets. The results of your test will not appear in the command prompt 
from which you started your terminal

## Inserting data 
To get a better idea about how data is inserted into MongoDB from Meteor, run the following code snippet in your Meteor Shell, after running your project:
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

After running the code snippet for data insertion in your Meteor shell, you will get the following output in the terminal window:
```
'MLx7SF79kXvAZuEyx'
```

This represents the response we get from .insert() after inserting the data. This represents the newly created id. 
(Which is stored as _id in the Document in the database)


## Finding data
Now let's fetch all the the data related to our donuts from the database (we run this in Meteor shell):
```js
DonutsCollection.find()
```

If you run this code snippet, you will get a bunch of data displayed into your Meteor shell.
So how was that accomplished ?
With a cursor! That involves using the methods find() and fetch(), about which we'll talk later!

```js
DonutsCollection.find().fetch()
```

Let's add another *donut* to our "inventory", so that we can do something more advanced with our data: 
```js
DonutsCollection.insert({price: 50, isCommestible: true});
```
As a tip, you should see an id in your shell. That way you know you did the tutorial right !

Now let's search our database:
```js
// we only have to eat commestible donuts
DonutsCollection.find({isCommestible: true}).fetch()

// and we want to get the good stuf, even if it's expensive, so let's get the donuts with a price bigger than 200
DonutsCollection.find({price: {$gt: 200}}).fetch()
```
To be able to query MongoDB efficiently, you need to know some things about selectors.
Read [this](https://docs.mongodb.com/manual/reference/operator/query/#query-and-projection-operators) to get acquainted with them.

find() also takes arguments, named "options":
```js
DonutsCollection.find({}, {
    sort: {price: -1}, // sorts price in descending order
    limit: 2, // limits the results to 2
    fields: {price: 1, isCommestible: 1} // will only return the fields price and isCommestible
}).fetch()
```

There are other fields as well, but we don't want to get into the details just yet, and if you're as curious as we are, 
you can read about them [here](http://docs.meteor.com/api/collections.html#Mongo-Collection-find).

## Updating the data

What about updating the data ?
Well, it's so simple you won't believe it:
```js
DonutsCollection.update({price: 50}, {
    $set: {price: 51}
})
```

The first argument represents the data that you "want to update", and the second tells the system "how you update it". 
You can use arguments such as "$inc" for incrementing, 
"$addToSet" if you want to add another delicious ingredient in the "flavors" array.

The idea is you can do a lot of things, and the options are endless.
So if you want to read more about updating in MongoDB, go [here](https://docs.mongodb.com/manual/reference/operator/update/).

## Removing data

Removing data is easy, and the argument it takes to do it is what we want to remove:
```js
DonutsCollection.remove({price: {gt: 1000}})
```

Keep playing with the code, try new inserts, try new selectors. Have a little bit of fun! You deserve it!

If you have a MongoDB visualizer, like "[Robomongo](https://robomongo.org/)" for Ubuntu, or any other tools out [there](http://lmgtfy.com/?q=mongodb+admin+software),
you can connect to the database by using "localhost" and port 3001, when Meteor is started, and see how it works for yourself!


## Tips and tricks

You can also find/update/remove by using the id as a string:
```js
DonutsCollection.update('XXX', modifier)
// equivallent
DonutsCollection.update({_id: 'XXX'}, modifier)
```

## Let's have some fun !

#### 1. Help mr John
Mr John wants to find all donuts that have a price bigger than 200 and they have at least one flavor!
Help him with build a query! [Hint](https://docs.mongodb.com/manual/reference/operator/query/size/#op._S_size)

#### 2. Price change
It's donut season. Run an update that increases all prices by 100, so you can make more profit! 
[Hint](https://docs.mongodb.com/manual/reference/operator/update/inc/#up._S_inc)

#### 3. Reality hit
People did not like the new prices, so create a query that will decrease the prices for the most 3 expensive 
donuts by 10%. [Hint](https://docs.mongodb.com/manual/reference/operator/update/mul/#up._S_mul)








