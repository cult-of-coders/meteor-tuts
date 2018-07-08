---
title: 'Publications'
description: How to architect publications
disqusPage: 'Chapter 3: Publications'
---

## Stop using them

Yes you read that right, unless you want to create a custom functionality that uses the beautiful LiveData utilities
from Meteor, stop using publications, instead leverage [Grapher](http://grapher.cultofcoders.com) for this.


If you would like to find one post and have it react to changes you will create a NamedQuery:

```js
// file: /imports/db/posts/queries/getPost.js
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

```js
// file: /imports/api/posts/queries/getPost.expose.js
// if you don't expose it you can't work with it
import getPost from './getPost';

getPost.expose({
    firewall(userId, params) {
        // some rules here to secure the query
    }
})
```

In the client you will use it something like this:

```js
import getPostQuery from '/imports/db/posts/queries/getPost';

export default createContainer(({postId}) => {
    const query = getPostQuery.clone({postId});
    const handle = query.subscribe();
    
    return {
        ready: handle.ready(),
        post: query.fetchOne(),
    };
})
```

The advantages are the following:
- No need to do big changes if you decide to make a query non-reactive
- Separates the concern of data fetching
- Brings modularity and easy linking with other collections.

You have to use [Grapher](http://grapher.cultofcoders.com/index.html) in your Meteor app if you want to bring it to enterprise standards. Period. It's the best way we know.

