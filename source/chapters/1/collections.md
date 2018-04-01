---
title: Collections
description: How we store our data in Meteor.
disqusPage: 'Chapter 1: Collections'
---

## Let's talk data!

Meteor uses MongoDB as its default database. You can use any database you want, because you have access to
*http://www.npmjs.com*, therefore you have access to almost all the existing database drivers out there 
(a database driver is a program which implements a protocol for connecting to a specific database, like MongoDB, MySQL, SqLite and so on). 

## How to create a collection?

```js
// imports/db/posts/collection.js
import { Mongo } from 'meteor/mongo';

const Posts = new Mongo.Collection('posts');

export default Posts;
```

## Inserting data

We already learned how to create and use a method so we are going to insert in the database a Post
```js
// file: /imports/api/posts/methods.js
// do not forget to import it in the '/imports/startup/server/index.js';
import {Meteor} from 'meteor/meteor'
import Posts from '/imports/db/posts'; // or, as it's on github, for simplicity, import {Posts} from '/db';

Meteor.methods({
    'post.create'(post) {
        Posts.insert(post);
    }
});

```
To test this, you can go in the browser's console and try something like this:
```js
const data = {title: 'test', description: 'some description'};
Meteor.call('post.create', data); // this will call the 'post.create' method and will add a post in the database
```
We can add in the database whatever we want, from strings to array of objects. The sky is the limit!

## Mongo tools - check what's in the database
You could go in a new terminal, go in the root directory of your project and then `meteor mongo` or use an application like [robomongo](https://robomongo.org/) so you can visualize the data from your database

If we choose to use the terminal and run the command written above, we could check all the posts from the database by typing: `db.posts.find({})`, this will search for all the posts from the database and retrieve an array of objects of there are any.

## Find all posts
```js
// file: /imports/api/posts/methods.js
    'post.list' () {
        return Posts.find().fetch();
    }
```

To be able to query MongoDB efficiently, you need to know some things about selectors (meaning what are you going to select from your collection).
Read [this](https://docs.mongodb.com/manual/reference/operator/query/#query-and-projection-operators) to get acquainted with them.

The second argument for `find()` is useful for sorting, limiting the amount of received documents, to get only specific fields and many others. Read [this](https://docs.meteor.com/api/collections.html#Mongo-Collection-find) for more details

## Updating a post
```js
    'post.edit' (_id, post) {
        Posts.update(_id, {
            $set: {
                title: post.title,
                description: post.description
            }
        });
    }
```

The first argument represents the data that you "want to update", and the second tells the system "how you update it".
The idea is you can do a lot of things, and the options are endless.
So if you want to read more about updating in MongoDB, go [here](https://docs.mongodb.com/manual/reference/operator/update/).
## Remove a post with a specific _id
```js
    'post.remove' (_id){
        Posts.remove(_id);
    }
```

## Fetch a post with a specific _id
```js
    'post.get' (_id) {
        return Posts.findOne(_id);
    }
```

## Work more to learn more

Now let's practice what we've learned in this section with a few simple exercises:

We'll consider a post  in the following format:
```js

const post = {
title: 'something', // string
description: 'meteor-tuts', // string
isApproved: true, // boolean
views: 10,
linkedPostIds: ['aaa', 'bbb', 'ccc', 'ddd'],
createdAt: new Date // Date
}
```
#### 1. Posts with a specific title
Make a method which returns all the posts with title: 'test'
```js
Meteor.call('post.get_by_title', callback)
```

#### 2. Posts with views
Make a method which returns all the posts that have between 100 and 200 views and that returns only the title (as fields)
```js
Meteor.call('post.get_by_views', {title: 'test', views: 300}, callback)
```

#### 3. Removing posts
Make a method that removes all the posts with title: 'test' or description: 'test'
```js
Meteor.call('post.remove_by_test', callback)
```

#### 4. Updating posts
Make a method that increments the number of views by 1 for each post that is approved
```js
Meteor.call('post.increment_views', callback)
```

#### 5. Remove old posts
Make a method which removes all the posts from last month (from the first day of month, to the last day of that month)
```js
Meteor.call('post.remove_from_last_month', callback)
```



