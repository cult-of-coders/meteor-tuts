---
title: 'Methods & Structure'
description: How to architect methods
disqusPage: 'Chapter 3: Methods'
---

Usually, you tend to put logic in your Meteor methods which is a very very bad terrible thing, because Meteor methods act as server-side routes, they act as the *C* (Controller) in *MVC*, 
the *C* does not contain logic, the *C* is a boss that delegates work to *M* (Services aka Models)

It is also bad because methods are a communication layer between the client and the server, they shouldn't store business logic, or logic of any kind, imagine them as proxies that communicate with your services.

## Folder Structure

Keep your methods inside a `/server` folder, this will allow Meteor to only refresh the server, without refreshing your browser,
if you make a change inside a method.

Take this scenario for example:
- You have a form you click submit
- An error happens server-side because of your fault
- You go in the code, you fix the error, and you save the file
- Meteor refreshes and it also refreshes your browser, loosing the form data

By coupling server-only logic under `/server` folder you will benefit a faster development speed.

So, I suggest the following design:

```js
// file: /imports/api/posts/server/methods.js
import {Meteor} from 'meteor/meteor';

Meteor.methods({
    'posts.create'() { ... }
})
```

```js
// file: /imports/api/posts/server/index.js
import './methods';
import './publications'; // You can also import your publications
import './listeners'; // And also your listeners in here!
```

```js
// file: /imports/api/server.js
import './posts/server';
import './comments/server';
import './items/server';

// And don't forget to import this in your main startup script in /server/main.js
```

## Method Naming

Use `{module}.{submodule[1..n]}?.{action}`
I recommend lowercase and plural form and camelCase for multi-words. Whatever you choose, just stick to it!

Examples:

```
posts.create
posts.comments.create
posts.comments.impression.create
```

## Method Calls

Whenever you do a method call I recommend sending an object, rather than multiple arguments.

For example instead of:
```js
// BAD
Meteor.call('posts.edit', postId, postData)
```

```js
// GOOD
Meteor.call('posts.edit', {
    postId,
    postData
})
```

The reason is simple: easier inspection on the server and in the websocket frames. Plus, it's easier to validate. You validate against one object.

If your method is heavily used, ok, I agree use arguments because they cost less, but I highly doubt you will need this. Don't micro-optimize. 

## Method Body

A method should do the following:
1. **Validate** - Check that the request is valid
2. **Secure** - Check user permissions
3. **Delegate** - Delegate job to services

We can do #1 by using `check` but I'd rather use SimpleSchema:
```js
// ...
Meteor.methods({
    'posts.edit'(data) {
        new SimpleSchema({
            postId: {type: String},
            postData: {type: Object, blackbox: true},
        }).validate(data);
        // it will throw an exception if it's not good
        
        const {postId, postData} = data;
        // ...   
    }
})
```

For #2, always use a Security service, which you either store at top level in `/imports/api/security.js` or locally in `posts` module folder,
or you could use both, why not?

```js
import PostSecurity from '/imports/api/posts/security.js';
import Security from '/imports/api/security.js';

Meteor.methods({
    'posts.edit'({postId, postData}) {
        // ...
        Security.checkLoggedIn(this.userId);
        PostSecurity.checkAllowedToEdit(this.userId, postId);
        // ...
    }
})
```

These security checks must throw a `Meteor.Error` if the check does not pass.

For #3, you know it, we [use a service](/chapters/3/services.html).

## Optimistic UI

Sometimes you need that when you do an action which updates a local collection (meaning you observe reactive data) and
you have a form that does an insert, you want it to immediately appear on your view. (not wait for the publication to send it to you)

The best way to do this is to use ValidatedMethod: https://github.com/meteor/validated-method

Store them inside: `/imports/api/{module}/optimistic/methodName.js`
Import them all in: `/imports/api/{module}/optimistic/methods.js`
And if you have `imports/api/{module}/server/index.js` make sure to `import ../optimistic` inside `/imports/api/{module}/server/methods.js`

In most cases you won't use this, but it's a very nice feature when you need to.

## Mixins

There will be scenarios where all your methods defined need a logged in user, or a user with a certain role, it can become tedious and repetitive, that 
inside every method you do this check, you need to do it once and keep your code [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).


```js
// file: /imports/api/mixins/index.js
import _ from 'underscore';

const mix = function(mixins, methods) {
    _.each(methods, (value, key) => {
        methods[key] = function (...args) {
            mixins.forEach(mixin => mixin.call(this, ...args));
            return value.call(this, ...args);
        }
    })
}

const checkLoggedIn = function () {
    if (!this.userId) {
        throw new Meteor.Error('not-allowed');
    }
};

export {mix, checkLoggedIn};
```

And where you define your methods:
```js
import {mix, checkLoggedIn} from '/imports/api/mixins';

const methods = mix([checkLoggedIn], {
    'posts.create'() {
        // do something;
    }
);

Meteor.methods(methods);
```

These few lines of code can have great impact on readability and elegance of your code. Feel free to use them!




