---
title: Exposing Collections
description: Learn how to securely expose your queries for client fetching
---

Collection Exposure
===================

In order to query for a collection from the client-side and fetch it or subscribe to it. You must expose it.


Exposing a collection does the following things:

- Creates a method called: exposure_{collectionName} which accepts a query
- Creates a publication called: exposure_{collectionName} which accepts a query and uses [reywood:publish-composite](https://atmospherejs.com/reywood/publish-composite) to achieve reactive relationships.
- If firewall is specified, it extends *find* method of your collection, allowing an extra parameter:
```
Collection.find(filters, options, userId);
```


{% pullquote 'warning' %}
If *userId* is undefined, the firewall and constraints will not be applied. If the *userId* is *null*, the firewall will be applied. This is to allow server-side fetching without any restrictions.
{% endpullquote %}

Exposing a collection to everyone
---------------------------------

```
Collection.expose()
```

Exposing a collection to logged in users
----------------------------------------

```
Collection.expose({
    firewall(filters, options, userId) {
        if (!userId) {
            throw new Meteor.Error('...');
        }
    }
});
```

Enforcing limits
----------------

```
Collection.expose({
    firewall(filters, options, userId) {
        filters.userId = userId;
    }
    maxLimit: 100, // The publication/method will not allow data fetching for more than 100 items. (Performance related)
    maxDepth: 3 // The publication/method will not allow a query with more than 3 levels deep. (Performance related)
    restrictedFields: ['services', 'secretField'] // this will clean up filters, options.sort and options.fields and remove those fields from there.
});
```


Exposure firewalls are linked.
------------------------------

When querying for a data-graph like:
```
{
    users: {
        comments: {}
    }
}
```

It is not necessary to have an exposure for *comments*, however if you do have it, and it has a firewall. The firewall will be called.
The reason for this is security.

{% pullquote 'warning' %}
Don't worry about performance. We went great lengths to retrieve data in as few MongoDB requests as possible, in the scenario above,
if you do have a firewall for users and comments, both will be called only once, because we only make 2 MongoDB requests.
{% endpullquote %}

Global Exposure Configuration
-----------------------------
```
import { Exposure } from 'meteor/cultofcoders:grapher';

// Make sure you do this before exposing any collections.
Exposure.setConfig({
    firewall,
    maxLimit,
    maxDepth,
    restrictFields
});
```


When you expose a collection, it will extend the global exposure methods.
The reason for this is you may want a global limit of 100, or you may want a maximum graph depth of 5 for all your exposed collections,
without having to specify this for each.

Important: if global exposure has a firewall and the collection exposure has a firewall defined as welll, the collection exposure firewall will be applied. 

## Taming The Firewall

```
// control what to show

Collection.expose({
    firewall(filters, options, userId) => {
        if (!isAdmin(userId)) {
            filters.isVisible = true;
        }
    }
});
```

```
// make certain fields invisible for certain users
import { Exposure } from 'meteor/cultofcoders:grapher'
Collection.expose({
    firewall(filters, options, userId) => {
        if (!isAdmin(userId)) {
            Exposure.restrictFields(filters, options, ['privateData'])
            // it will remove all specified fields from filters, options.sort, options.fields
            // this way you will not expose unwanted data.
        }
    }
});
```

## Restricting Links

Restrict using a simple array:
```
Collection.expose({
    restrictLinks: ['privateLink', 'anotherPrivateLink']
});
```

Compute restricted links when fetching the query:
```
Collection.expose({
    restrictLinks(userId) {
        return ['privateLink', 'anotherPrivateLink']
    }
});
```

## Exposure Body 

Creating an exposure body basically states that:
"I allow the client to request anything he wants from what I allow him to."

If *body* is specified, it is first applied on the request and then the subsequent rules such as *restrictedFields*, *restrictLinks*, *firewall*

This is for advanced usage and it completes the security of exposure. 
This may be a bit tricky to understand at first because there are many rules, but don't give up hope, it's quite easy.

{% pullquote 'warning' %}
By using body, Grapher automatically assumes you have control over what you give, meaning all firewalls from other exposures for linked elements (if they exist) will be bypassed. But not the firewall of the current exposure.
{% endpullquote %}

### Basic Usage

```
Collection.expose({
    body: {
        firstName: 1,
        link: {
            someField: 1
        }
    }
})
```

If you query from the *client-side* something like:
```
createQuery({
    collection: {
        firstName: 1,
        lastName: 1,
        link: {
            someOtherField: 1
        }
    }
})
```

The intersected body will look like:
```
{
    firstName: 1,
    link: {}
}
```

Ok, but what if I want to have a different body based on the userId ? Body can also be a function that takes in an userId, and returns an object.

```
Collection.expose({
    body(userId) {
        let body = { firstName: 1 };
        
        if (isAdmin(userId)) {
            _.extend(body, { lastName: 1 })
        }
        
        return body;
    }
})
```

{% pullquote 'warning' %}
Deep nesting with other links not be allowed unless your *body* specifies it.
{% endpullquote %}

The special fields "$filters" and "$options" are allowed at any link level (including root). However, they will go through a phase of cleaning,
meaning it will only allow to filter and sort for fields that exist in the body.


{% pullquote 'warning' %}
We got your back covered!

This check goes deeply to verify "$and", "$or", "$nin" and "$not" special MongoDB selectors. This way you are sure you do not expose data you don't want to.
Because, given enough requests, a hacker playing with $filters and $sort options can figure out a field that you may not want to give him access to.
{% endpullquote %}

If the *body* contains functions they will be computed before intersection. Each function will receive userId.
```
{
    linkName(userId) { return {test: 1} }
}

// transforms into
{
    linkName: {
        test: 1
    }
}
```

You can return *undefined* or *false* in your function if you want to disable the field/link for intersection.

```
{
    linkName(userId) {
        if (isAdmin(userId)) {
            return object;
        }
    }
}
```

### Linking Grapher Exposure Bodies

You can link bodies in your own way and also reference other bodies'links.
Functions are computed on-demand, meaning you can have self-referencing body functions:

```
// Comments ONE link to Users as 'user' 
// Users INVERSED 'user' from Comments AS 'comments'

const commentBody = (userId) => {
    return {
        user: userBody
        text: 1
    }
}

const userBody = (userId) => {
    if (isAdmin(userId)) {
        return {
            comments: commentBody
        };        
    }
    
    return somethingElse;
}

Users.expose({
    body: userBody
})

Comments.expose({
    body: commentBody
})
```

This will allow requests like:
```
{
    users: {
        comments: {
            user: {
                // It doesn't make much sense for this case
                // but you can have 
            }
        }
    }
}
```