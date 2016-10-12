---
title: Named Query
description: Reference API for Grapher Named Query
---

{% pullquote 'warning' %}
NamedQuery is not affected by exposure at all. It has it's own firewall.
{% endpullquote %}

## createNamedQuery()

```
import { createNamedQuery } from 'meteor/cultofcoders:grapher';

const query = createNamedQuery('postList', {
    posts: {
        title: 1,
        comments: {
            text: 1
        }
    }
});
```

*query* has the same API as Query. [Check it out here](/api/query.html)

## NamedQuery.expose(config)

Server-side only. This is used for giving access to a named query client-side.

```
query.expose({
    firewall(userId, params) {
        // throw-out an exception or do anything you want
    },
    method: true, // defaults to true, allows you to fetch it statically
    publication: true, // defaults to true, allows you to fetch it reactively,
    embody: {
        // deep merges with the query body after the first node
        // this allows you to have full control over the query without exposing important business logic to the client
        teamMember: {
            $filters: {isActive: true}
        }
    }
})
```

## Basic usage

```
// client-side
import postList from '/imports/api/posts/queries/postList.js';

postList.clone(params).fetch((err, data) => {
    // do something with data
})
// server-side
import postList from '/imports/api/posts/queries/postList.js';

postList.clone(params).fetch()
```

## createQuery()

{% pullquote 'warning' %}
If you use *createQuery*, your query must be already loaded for the client. Imported in your startup file, or however you do. The reason is that
once a *NamedQuery* is registered, it is kept in a store. And when using *createQuery()* it will try to interogate that store.

The only reason we allow this, instead of importing it, is to allow testing it live with [Grapher-Live package](https://github.com/cult-of-coders/grapher-live)
{% endpullquote %}

```
import { createQuery } from 'meteor/cultofcoders:grapher';

const query = createQuery({
    postList: {
        text: 'Hello World!' // in this case the body of postList are actually the parameters.
    }
})
```

*query* has the same API as Query. [Check it out here](/api/query.html)
