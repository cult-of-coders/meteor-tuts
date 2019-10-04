---
title: 'Testing'
description: Writing Tests is so joyful!
disqusPage: 'Chapter 3: Testing'
---

## Why

Because your code grows and you make less mistakes.
Because it's often faster for you to write a service and test it in a simple unit-test, rather then going on browser for manual testing.

There are so many articles about why testing is mandatory. Just take my word for it.

## Configuration 
This article assumes you thoroughly read: https://guide.meteor.com/testing.html 

If you decoupled your code nicely, it's so easy to write tests:

```
meteor add practicalmeteor:mocha
meteor npm i --save-dev chai sinon
```

```
// package.json
{
  "scripts": {
     "test": "meteor test --driver-package practicalmeteor:mocha --port 3050"
  }
}
```

Now you can easily do in your terminal: `npm run test`. Then open your browser on http://localhost:3050 to see the results,
they will be updated live as you write your tests or update your files.

Test files should end in `*.test.js` and they should have a parent (not necessarily direct) with name `client` or `server`
depending where we want to run them.

```js
// file: /imports/api/posts/testing/server/PostService.test.js

import {assert} from 'chai';
import PostService from '/imports/api/posts/services/PostService';

describe('Post Service', function () {
    it('Should be able to create a post', function () {
        const postId = PostService.createPost({title: 'Hello'})
        
        assert.isString(postId);
    })
})
```

## Utilities

- https://github.com/xolvio/cleaner
- https://github.com/versolearning/meteor-factory/
- http://sinonjs.org/releases/v4.0.1/
- http://chaijs.com/api/assert/

## Continous Integration

Whenever someone does a Pull-Request, you can see if the code breaks some tests or not. This is invaluable, and so easy to implement.
Really!

Some good CI tools:

- https://circleci.com/docs/1.0/getting-started/ 
- https://travis-ci.org/

