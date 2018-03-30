---
title: Security
description: Let's talk Security
disqusPage: 'Chapter 1: Security'
---

Security is a very important aspect of any application. When your code-base grows,
you need to more and more careful on how you handle this. We'll first show how we can
secure our methods and publications, then we will get into some tips and tricks meant to teach you how to handle 
the security for an evergrowing code base.

## Securing Methods & Publicaions

So, remember the Methods ?

```js
Meteor.methods({
    'post.create'(post) {
        Posts.insert(post);
    },
})
```

In every method & publication we can access `this.userId`, this will return either `null`, meaning the user is not authenticated, or a string with the actual userId, something like `8qLGFKSn6szJkzsyG`

## Managing Roles

Based on the userId you have the ability to check if the user is logged in, maybe you have multiple roles in the system,

We recommend installing the infamous package, [alanning:roles](https://atmospherejs.com/alanning/roles):

```
meteor add alanning:roles
```


## Security Module

Centralize security in a module:

```js
// file: /imports/api/security.js
// example of a module for security
import { Roles } from 'meteor/alanning:roles';

export default class Security {
    static checkRole(userId, role) {
        if (!this.hasRole(userId, role)) {
            throw new Meteor.Error('not-authorized');
        }
    }

    static hasRole(userId, role) {
        return Roles.userIsInRole(userId, role);
    }

    static checkLoggedIn(userId) {
        if (!userId) {
            throw new Meteor.Error('not-authorized', 'You are not authorized');
        };
    }
    
    // add other business logic checks here that you use throughout the app
    // something like: isUserAllowedToSeeDocument()
    // always keep decoupling your code if this class gets huge.
}
```

Pretty straight forward right ? The reason we do it like this, by centralizing security in one place,
is to remove boilerplate code inside our methods and keep separation of concerns. You can do it however you want it, there is no right or wrong way to do it,
depends on your use-case, but we believe that it is easier to maintain, and newly onboarded developers were writing secure
code right from the start!

Simple usage in methods:

```js
import {Meteor} from 'meteor/meteor'
import {Posts} from '/db';
import Security from '/imports/api/security';

Meteor.methods({
    'secured.post_create'(post) {
        Security.checkLoggedIn(this.userId);
        post.userId = this.userId;
        Posts.insert(post);
    },

    'secured.post_list' () {
        return Posts.find().fetch();
    },

    'secured.post_edit' (_id, postData) {
        Posts.update({_id: _id, userId: this.userId}, {
            $set: {
                title: postData.title,
                description: postData.description
            }
        });
    },

    'secured.post_remove' (_id){
        Posts.remove({_id: _id, userId: this.userId});
    },

    'secured.post_get' (_id) {
        return Posts.findOne(_id);
    }
});
```

Simple usage in publications:

```js

import Security from '/imports/api/security.js';
import {Meteor} from "meteor/meteor";

Meteor.publish('posts', function () {
    return Posts.find({userId: this.userId});
})
```

That's it. With this knowledge you can build more secured apps!