---
title: 'Persistence Layer'
description: Using collections the right way
disqusPage: 'Chapter 3: Persistence Layer'
---

## What is it ?

The Persistence Layer (PL) is that "thingie" in your app that communicates with another API to store the data we want.
In our case, we communicate with MongoDB for that. Therefore, an instance of `Mongo.Collection` is our API to communicate with the Persistence Layer, therefore in our app we can regard collections as the PL.

What if at some point in the future, you decide that a certain
collection should not be kept in MongoDB but rather used via an external HTTP API? 

Your code architecture should try to be as independent as possible from the PL, because this opens the path for scaling and decoupling later on.

Just try and think this way, and whenever you want to start a new project: disregard the way you are going to store the data, focus more on the app/business logic layer.

Then build your database on top of it.

## Structure

Since we want to explicitly separate it, I propose that instead of storing collections inside `/imports/api`, we give them their own folder `/imports/db`

`/imports/api/posts/collection.js` -> `/imports/db/posts/collection.js`


```js
// file: /imports/db/posts/collection.js
import {Mongo} from 'meteor/mongo';

const Posts = new Mongo.Collection('posts');

export default Posts;
```

Now let's assume you have a new collection for comments on posts, and it's used only for posts. What do we do ?

Should we store it in `/imports/db/posts/comments/collection.js` or in `/imports/db/postComments/collection.js` ?

It's a matter of preference here. I prefer the first approach, but the collection's name I use is `PostComments` not `Comments`, so
we know it's context. But this only applies if my collection is coupled in a way to posts, if not, then go with second approach.

## Validating Documents

We need to be able to validate easily the data we save in a collection to prevent bad things from happening.

For this we are going to use [`aldeed:collection2-core`](https://github.com/aldeed/meteor-collection2-core) and [`SimpleSchema`](https://www.npmjs.com/package/simpl-schema)

Read more about it here: https://github.com/aldeed/meteor-collection2-core then come back!

```js
meteor add aldeed:collection2-core
meteor npm i -S simpl-schema
```

Now let's create a schema for our `Post`:

```js
// file: /imports/db/posts/schema.js
import SimpleSchema from 'simpl-schema';

export default new SimpleSchema({
    title: {
        type: String,
    },
    tags: {
        type: Array,
    },
    'tags.$': {
        type: String,
    },
    isApproved: {
        type: Boolean,
        defaultValue: false,
    },
    userId: {
        type: String,
    }
})
```

Now let's attach it:
```js
// file: /imports/db/posts/collection.js
import {Mongo} from 'meteor/mongo';
import PostSchema from './schema';

const Posts = new Mongo.Collection('posts');

Posts.attachSchema(PostSchema);

export default Posts;
```

Nice! This structure works fine. This will never allow you to insert or update a document without validating it properly.

Beautiful, but now we see those "tags" in there, and we know from our previous chapter, that we need a way to re-use them properly,
this is why we introduce here to concept of Enums.


## Enums

An Enum is a set of constants, and they should be in their own module. Let's try an example

```js
// file: /imports/db/posts/enums/tags.js

const PostTagsEnum = {
    PSYCHOLOGY: 'psychology', 
    PHILOSOPHY: 'philosophy',
}

// maybe you want to have them used as human readable 
const PostTagsLabels = {
    [PostTagsEnum.PSYCHOLOGY]: 'Psychology & Education',
    [PostTagsEnum.PHILOSOPHY]: 'Philosophy & Arts',
}

export default PostTagsEnum;

export {
    PostTagsEnum,
    PostTagsLabels,
}
```

The value of the enum, represents how you internally want to store it. Does not represent how you want to display it. Keep that in mind.

And if we go back to our schema it can look something like:
```js
import _ from 'underscore'; // don't forget to do meteor npm i -S underscore
import SimpleSchema from 'simpl-schema';
import PostTagsEnum from './enums/tags';

export default new SimpleSchema({
    ...
    'tags.$': {
        type: String,
        allowedValues: _.values(PostTagsEnum) // this returns ['psychology', 'philosophy']
    },
    ...
})
```
Another quick example to ilustrate their purpose, imagine a status of a Task:
```js
// /imports/db/tasks/enums/status.js
export default {
    ACTIVE: 'active',
    PENDING: 'pending',
    COMPLETED: 'completed'
}
```

The rule using enums is that when you are dealing with them, you must never use the string/value itself.

In your code, client or server, when you have to see if a status is active:
```js
// VERY BAD
if (object.status === 'active') {
    // do something
}
```

```js
import ObjectStatusEnum from '...';
// OK
if (object.status === ObjectStatusEnum.ACTIVE) {
    // do something
}
```

Keep in mind that you can use Enums anywhere, not only for storing stuff in your db. Whenever you find there's a list of constants needed,
you are looking at an enum. And ofcourse you can use them client-side or server-side.

## Meteor.users

This collection feels like it is from another world, it's best that we standardize it as well, and also
give it a schema:

```js
// file: /imports/db/users/collection.js
import {Meteor} from 'meteor/meteor';
import UserSchema from './schema';

const Users = Meteor.users;

Users.attachSchema(UserSchema);

export default Users;
```

A sample schema that works with Accounts configuration:
```js
// file: /imports/db/users/schema.js
import SimpleSchema from 'simpl-schema';

export default new SimpleSchema({
    _id: {type: String},
    username: {type: String, optional: true},
    emails: {type: Array},
    'emails.$': {type: Object},
    'emails.$.address': {type: String},
    'emails.$.verified': {type: Boolean},
    createdAt: {type: Date},
    services: {type: Object, blackbox: true},
    roles: {
        type: Array,
        optional: true
    },
    "roles.$": {
        type: String
    },
    profile: {
        type: Object,
        optional: true
    },
    'profile.firstName': {
        type: String,
        optional: true
    },
    'profile.lastName': {
        type: String,
        optional: true
    },
});
```

And don't forget about this nice package: https://github.com/alanning/meteor-roles. Allows you to nicely manage roles in the system.


## Hooks

You may want to do something after an element is removed or updated or inserted from a certain collection.

The almost official way to do this is by using this awesome package: https://atmospherejs.com/matb33/collection-hooks
 
Give it a read, you'll love it.

My recommendation here is to dispatch an event on a given hook, and store the hooks inside `/imports/db/{collection}/hooks.js`,
and the logic for handling the hooks should be inside `/imports/api`

## Behaviours

Most likely, your collections will store the user who created something, or store things like when it was created, or when it was updated.

So instead of creating hooks for every collection, it's easier to define a behaviour and re-use it.

Please read about it on these links:
https://github.com/zimme/meteor-collection-behaviours/

For storage, I think it's fine to just attachBehaviours where you store the collection:

```js
// file: /imports/db/posts/collection.js

// ...
Posts.attachBehaviour('timestampable');
```

## Extensions

Don't be afraid to extend your collection, especially when you need something that is specific to the collection. For example, you have the `_id` of an user,
and you want to quickly get the email:

```js
// file: /imports/db/users/extensions.js

export default {
    getEmailForUserId(userId) {
        const user = this.findOne(userId, {
            fields: {emails: 1}
        });
        
        if (!user) {
            throw new Meteor.Error('not-found');
        }
        
        return user.emails[0].address;
    }
}
```

```js
// file: /imports/db/users/collection.js
import UsersExtension from './extensions';

// ...
_.extend(Users, UsersExtension);
// ...
```

## Helpers

With the use of this package: https://github.com/dburles/meteor-collection-helpers you can make your documents smarter if they are fetched through your collections.
```js
// file: /imports/db/users/helpers.js

export default {
    getEmail(userId) {
        return this.emails[0].address;
    }
}
```

```js
// file: /imports/db/users/collection.js
import UsersHelpers from './helpers';

// ...
Users.helpers(UsersHelpers);
// ...
```

This will allow you to easily do:

```js
Users.findOne(userId).getEmail()
```

## Shortcuts
 
Create an index for your collections:

```js
// file: /imports/db/index.js

import Posts from './posts/collection';
import Comments from './comments/collection';

export {
    Posts,
    Comments
}
```

This will allow you to use it like this:

```js
import {Posts, Comments} from '/imports/db';
```

You can go ahead and even put enums inside index.js, but be very careful with their naming,
as they need to be specific: `${collection}${enumName}Enum`, example:
- TaskStatusEnum
- PostTagsEnum 
- etc

## Relations (Grapher)

How do we work with relations in a non-relational database? We use [Grapher](http://grapher.cultofcoders.com), ofcourse.
And even if we don't have relations, we still should use [Grapher](http://grapher.cultofcoders.com) because of the way it works, as you will see below:

Here's how simple it is:

```bash
meteor add cultofcoders:grapher
```

```
// file: /imports/db/posts/links.js
import {Posts, Users} from '/imports/db';

Posts.addLinks({
    author: {
        type: 'one',
        collection: Users,
        field: 'authorId'
    }
})
```

We need those links loaded, so aggregate all links importing into a single file. 
Import this file in the client-side and server-side init modules.

```js
// file: /imports/db/links.js
import './posts/links.js';
```

And when you want to get the post alongside with the user's profile for example:

```js
const post = Posts.createQuery({
    $filters: {
        _id: postId,
    },
    title: 1,
    user: {
        profile: 1,
    }
}).fetchOne((err, post) => {
    console.log(post.user.profile.firstName);
});
```

[Grapher](http://grapher.cultofcoders.com) is a very complex tool and it can work with queries client-side as well. You can have reactive queries, there's simply a lot related to it,
but it's properly documented here: http://grapher.cultofcoders.com. 

Begin with the Guide, and patiently read through it. Once you read the guide you can understand the following patterns:

### Creating a simple query

The recommended way of working with [Grapher](http://grapher.cultofcoders.com) is by working exclusively with NamedQueries.
If you would like to store and expose NamedQueries:

```js
// file: /imports/db/posts/queries/getPost.js
import Posts from '../collection';

export default Posts.createQuery('getPost', {
    $filter({filters, params}) {
        filters._id = params.postId;
    },
    title: 1,
    user: {
        profile: 1,
    }
});
```

Don't be afraid to create shortcuts for all your queries in one place:
```js
// file: /imports/db/queries.js
import getPost from './posts/queries/getPost',

export {
    getPost
}
```

Now that we created the query, we need to expose and secure it:

```js
// file: /imports/api/posts/queries/getPost.expose.js
// if you don't expose it you can't work with it
import {getPost} from '/imports/db/queries'

getPost.expose({
    firewall(userId, params) {
        // you can manipulate params object here
        if (isAllowed(userId)) {
            // you can throw exceptions if he's not allowed access to this query
        }
        // the firewall can either: throw exception or modify params
    }
})
```

Aggregate it in one file and import this file on server-side only.
```js
// file: /imports/api/exposures.js
import './posts/queries/getPost.expose.js';
```

Now from the client you can easily do something like:
```js
// barbaric (but it's also nice)
createQuery({
    getPost: {
        postId: 'XXX'
    }
}).fetchOne((err, post) => {
    console.log(post);
    // you will also have access to post.users.name
})

// modular
import {getPost} from '/imports/db/queries'

getPost.clone({
    postId: 'XXX'
}).fetchOne((err, post) => { ... })
```

By abstracting the data retrieval layer to [Grapher](http://grapher.cultofcoders.com) and allowing it to do the linking of collections for you, you will find yourself with so much less headache.
Because [Grapher](http://grapher.cultofcoders.com) forces you to specify only the fields you need, you will find your apps running fast and secure right from the start.

### Separating concerns

You may feel that having your query in `/imports/db` and your exposure inside `/imports/api` to be a little odd. 
This is why you can put your query inside `/imports/api` as well. But the reason for not putting exposure inside `/imports/db` is
because it contains app logic, and that layer it should be independent from it.





