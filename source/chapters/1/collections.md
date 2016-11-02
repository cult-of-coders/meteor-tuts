---
title: Collections
description: How we store our data in Meteor.
disqusPage: 'Chapter 1: Collections'
---

Hello again.
So you want to learn how to store the data in Meteor, fine...

Let's proceed.

Meteor uses MongoDB as its backbone. Theoretically you can use any database you wish with Meteor, because
you have access to http://www.npmjs.com, therefore you have access to all db drivers. Even MySQL !
 
However, MongoDB gives Meteor its magic, unlike SQL, you don't have to CREATE TABLE, or CREATE FIELD to have a field,
it lets you structure your data the way you want. However, you still need consistency, this is why later on,
we'll explore how we can ensure this consistency at APP level, not DB level, very easily.

Ok, so if you're familiar with SQL. Make this analogy, it'll help you grasp things faster:
- **DATABASE** = Database (yep, we got the same name)
- **TABLE** = Collection (something that a list of data, a collection)
- **ROW** = Document (the actual list of data inside the collection)

I like donuts, don't you like donuts, I love them so much I want to store them in a database:

```
// file: /imports/api/donuts/collection.js
import { Mongo } from 'meteor/mongo';

const Donuts = new Mongo.Collection('donuts');

export default Donuts;
```

Ok now what. Well, now let's play with it. Go to your terminal and type: 
```
meteor run
```

Then open another terminal, go to same folder and run:
```
meteor shell
```

Meteor shell is like a console for the server-side. Some quick tips, if you do in the shell:
```
console.log('hello shell');
```

You won't see it there, because the actual message is logged in the console of where meteor is started. Keep that in mind it will help you in the future.

Now, in the shell, let's import what we'll use:
```
import Donuts from '/imports/api/donuts/collection.js'
```

Now let's do our first insert:
```
Donuts.insert({
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

Now let's fetch this baby:
```
Donuts.find().fetch()
```

Note: We'll explain later what's the deal with find() and fetch().

Ok now lets do a more complex search, for that let's add another donut:
```
Donuts.insert({price: 50, isCommestible: true});
```

You may notice that you see something like:
```
'MLx7SF79kXvAZuEyx'
```

That's because .insert() returns the newly created id. (Which is stored as _id in the Document)

Let's search our database:
```
// Getting the commestible donuts
Donuts.find({isCommestible: true}).fetch()

// Getting the donuts with a price over 200
Donuts.find({price: {$gt: 200}}).fetch()
```

You have ability to use the selectors that MongoDB provides, here are all of them described:
https://docs.mongodb.com/manual/reference/operator/query/#query-and-projection-operators

There is also another argument that find() takes in, it's called "options":
```
Donuts.find({}, {
    sort: {price: -1} // sorts price in descending order
    limit: 2 // limits the results to 2
    fields: {price: 1, isCommestible: 1} // will only return the fields price and isCommestible
}).fetch()
```

There are other fields as well, but we don't want to get into many details just yet, if you simply cannot abstain yourself from the curiosity, check it out:
http://docs.meteor.com/api/collections.html#Mongo-Collection-find

What about updating ?

```
Donuts.update({price: 50}, {
    $set: {price: 51}
})
```

The first argument is "what you want to update", the second one is "how you update it". You can use stuff like "$inc" for incrementing, 
"$addToSet" if you want to add another disgusting ingredient in the "flavors" array.

The idea is you can do a lot of things, and there are many cool stuff, you can read more about it here:
https://docs.mongodb.com/manual/reference/operator/update/

Removing is easy, the argument it takes is what to remove:
```
Donuts.remove({price: {gt: 1000}})
```

Keep playing with it, try new inserts, try new selectors, have a little bit of fun, you deserve it.

If you have a MongoDB visualizer, like "Robomongo" for Ubuntu, or any other tools out there http://lmgtfy.com/?q=mongodb+admin+software
You can connect to it by using "localhost" and port 3001 (but you must have Meteor started)

Q: Ok so collections are nice, you showed me how to use them in a shell, but I'm a web dev, when do I start to see
things in ma freaking browser bro ?

A: Soon.











