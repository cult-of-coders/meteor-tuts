---
title: 'Microservices'
description: Microservices - Decouple & Scale
disqusPage: 'Chapter 3: Microservices'
---

## What?
This does not relate to "services" term we used previously, when we regarded it as an unit of logic.

`The term "Microservice Architecture" has sprung up over the last few years to describe a particular way of designing software applications as suites of independently deployable services.`

Indepenently deployable, so basically it means that we need to have multiple meteor apps, but how to manage and allow them to re-use code ?
It's quite easy.


## Why?

For example you have an app, and you have to manage the users and control the content. Doing it in the same app may be a wrong approach, either your bundle size will increase, you may
use modules that you don't need. And to use dynamic imports everywhere may be too much. Instead just separate the concerns, and move your admin app into a separate microservice.

## Solution

First of all, understand the fact that we can have multiple Meteor apps inside the same git repository:

```js
mkdir myapp
git init .
meteor create --bare admin
meteor create --bare web
```

That's it, you began your microservice architecture. Now if you want to re-use code, we use symlinks. Sorry Windows users, for you it will not work.
```
cd admin/imports
ln -s ../../web/imports app
```

Now look at this example:
```js
// file: ./admin/imports/api/posts/server/methods.js
import {Meteor} from 'meteor/meteor';
import {Posts} from '/imports/web/imports/db';

Meteor.methods({
    'posts.approve'({postId}) {
        // ...
    }
})
```

That's it. If you plan on only using the db models, and not interact with other services, than just do:

```
cd admin/imports
ln -s ../../web/imports/db db
```

and use it naturally:

```js
// file: ./admin/imports/api/posts/methods.js
import {Meteor} from 'meteor/meteor';
import {Posts} from '/imports/db';

Meteor.methods({
    'posts.approve'({postId}) {
        // ...
    }
})
```

So nice!
Note that if you use WebStorm, make sure to mark both "admin" and "web" folders as Resource Root (By right clicking on the folder), so you have autocomplete on absolute imports properly.

## Local Database

In order for this to work nicely, you would have to configure them to open on different ports and connect to the same database.

Note that when you start a meteor app, it opens it's own mongodb on PORT+1 (3001 in most cases).

But if we want the two microservices to work together, either install a docker mongodb locally, either [install mongodb directly](https://docs.mongodb.com/manual/installation/).

```js
// file: app/package.json
  "scripts": {
    "start": "MONGO_URL=mongodb://localhost:27017/web meteor run --port 3000",
  },
```
```js
// file: admin/package.json
  "scripts": {
    "start": "MONGO_URL=mongodb://localhost:27017/web meteor run --port 3050",
  },
```

Now you can start them both:
```
cd admin
npm start
```

```
cd web
npm start
```

## Deployment

Each individual microservice will have it's own `.deploy` folder inside it with the proper configuration of deployment, depending on which tool you use to do so.
The production bundle will be properly generated.

## A better strategy

You will most likely head into this issue, especially if you decide to microservicify your already existing app:

For example:
```js
// file: /web/imports/api/posts/services/PostService.js
import PaymentService from '/imports/api/payments/services/PaymentService.js';
// ...
```

This will fail if you want to use the PostService inside `admin` microservice! Because doesn't have that file in that path. 

The initial solution would be to use `imports PaymentService from '../../payments/services/PaymentService'`

Another great solution is to begin thinking about it as a [reusable module](/chapters/3/reusable-modules.html). Which you can store in `/modules` folder directly.
And create the following symlinks:
```bash
/web/imports/modules -- /modules
/admin/imports/modules -- /modules
```

If your module depends on things outside it, simply create a config inside it, and inject what you need inside it, keep it simple or make it as complex as you wish.
```js
// file: /modules/chat/config.js
export default {}
```

But there will be cases where you can't make everything completely independent, as they interact with many other system components,
this would require a change in strategy:

We have our `App Logic Layer` our `Persistence Layer` and our `Event System`

Let's work with the following architecture:

```bash
/core/services
/core/events.js
/core/security.js
/core/db
```

Now let's symlink it:
```bash
/web/imports/core -- /core
/admin/imports/core -- /core
```

Sample:
```js
// file: /web/imports/api/posts/server/methods.js

import PostService from '/imports/core/services/posts/PostService.js';

Meteor.methods({})
```

I personally don't recommend you using absolute imports any longer inside `/core`, even though you can do it if it starts with `/imports/core`.

## Conclusion

This "better" strategy would require you to do some refactoring to ensure it works properly provided you have an existing app already. However, if you don't and you know 
you will have microservices, start with this approach right from the beginning.

 
 


