---
title: 'Fixtures'
description: Generate dummy data. 
disqusPage: 'Chapter 3: Fixtures'
---

## Why?

The reason for writing fixtures is that once you setup your project, you fill it with relevant data, like users of different types and roles, posts, items, comments, whatever.

This removes the reason of having a shared database, and it makes onboarding new developers so easy.

## Structure

We will store our fixtures inside `/imports/fixtures` because this way you will not forget about them. They are an essential part of development.

```js
// file: /imports/fixtures/index.js
import createUsers from './createUsers';
import createPostsForUsers from './createPostsForUsers';

const runFixtures = function () {
    const shouldRun = Meteor.users.find().count() == 0;
    
    if (shouldRun) {
        createUsers();
        createPostsForUsers();
    }
}

if (!Meteor.isDevelopment) {
    runFixtures();
}
```

Keep a configuration so you can use it for load testing later on:

```js
// file: /imports/fixtures/config.js
export default {
    USERS_COUNT: 10,
    POSTS_PER_USER: 2,
}
```

And make sure you use the config in every fixture module you've got!

If it helps you for generation you can also use: https://github.com/versolearning/meteor-factory/ 

## Caution

If you have hooks that send emails or do API calls make sure you disable them when running your fixtures.

Keep your fixtures in sync with your schemas, so whenever you update it, make sure to update your fixtures as well.

Always work on adding fixtures as  you add different data.

Keep fixtures small and be careful when nesting them, you don't want to insert 100,000 documents because of a big nested loop.