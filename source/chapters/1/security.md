---
title: Security
description: Let's talk about Security
disqusPage: 'Chapter 1: Security'
---

Security is a very important aspect of any application. When your codebase grows, security must be one of the core aspects 
that you address. Users need to feel that the data they give you via the application is secure and that it respects the three 
most important requirements of data security - CIA:
- **C**onfidentiality
- **I**ntegrity
- **A**vailability. 
 
Let's start by securing our methods and publications, then we will talk about implementing security best practices for an evergrowing code base.

## Securing methods and publications in Meteor

Remember our talk about users ?
```js
Meteor.methods({
    'do_something': function () {
        console.log(this.userId);
    }
})
```
Here you get access to the userId, which represents the logged in user. If your user is not logged in, its value will be null.


The same goes for publications:
```js
Meteor.publish('something', function () {
    console.log(this.userId);
    return this.ready(); 
})
```

## Managing roles, or "who's who"

Based on the userId you can check if he/she is logged in, the roles it has in the system, the privileges granted to 
that user and, basically, you can do anything you want.

We recommend installing the [alanning:roles](https://atmospherejs.com/alanning/roles) package:
```
meteor add alanning:roles
```

## Order!

Now let's centralize all of our security in */imports/api/security.js*, so we can see how the "big picture" looks like:
```js
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
}
```
Word of advice: always keep decoupling your code if classes get huge. And by decoupling we mean that you should separate 
code into classes, depending on its functionality. Big classes, with multiple functionalities will make it much harder 
to debug your application in case errors appear. Because of this, we strongly recommend that you abide to KISS : Keep It Smart and Simple. 

Pretty KISS, right ? The reason for which we centralize security in one place is to remove the boilerplate code from 
inside our methods and separate the purposes of each code file. 
You can however structure your application however you want. There is no right or wrong.
However, from our experience, the applications that are structured like this are easier to maintain, and this kind of structure
 helps beginners write secure code right from the start!

Let's now cover some simple use cases:
```js
import {Meteor} from 'meteor/meteor'
import {Posts} from '/db';
import Security from '/imports/api/security';

Meteor.methods({
    'do_something': function () {
        // throw exception if the user is not logged it:
        Security.checkLoggedIn(this.userId);
        
        // throw exception if the user does not have a role:
        Security.checkRole(this.userId, 'ADMIN');
        
        if (Security.hasRole('ADMIN')) {
            return sensitiveData;
        } else {
            return publicData;
        }
    }
});
```

Publications usage shouldn't be too hard:
```js

import Security from '/imports/api/security.js';
import {Meteor} from "meteor/meteor";

Meteor.publish('posts', function () {
    let filters = {};
    if (!Security.hasRole(this.userId, 'ADMIN')) {
        // if the user is not an admin, we only show posts where "isPublic" is true
        filters.isPublic = true;
    }
    
    return Posts.find(filters);
})
```

You can now build secure apps with Meteor!
