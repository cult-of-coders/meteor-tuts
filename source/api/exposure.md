---
title: Exposure
description: Reference API for Grapher Exposure
---

Exposure is used to allow a certain collection as an *Entry Point* for your query. Exposing is done server side.

{% pullquote 'warning' %}
NamedQuery is not affected by exposure at all. It has it's own firewall.
{% endpullquote %}

## Exposure.setConfig
This is for global configuration. It will apply to all exposures unless overriden.

```
import { Exposure } from 'meteor/cultofcoders:grapher';

Exposure.setConfig({
    firewall(filters, options, userId) {}, 
    maxLimit, // integer value, force a maximum limit to results
    maxDepth, // integer value, allow the data graph to a certain depth
    publication, // allow reactivity (dynamic queries)
    method, // allow fetching via method (static queries)
});
```

## Collection.expose()

If you specify any parameters used in the global configuration, they will be overridden.

```
Collection.expose({
    firewall(filters, options, userId) {
    },
    maxLimit,
    maxDepth,
    restrictedFields: [], // array of fields you want to restrict
    method: true, // allow it via method
    publication: true, // allow it via 
    restrictLinks, // function(userId) that returns array, or array,
    body, // object that will intersect with the actual request body from the client
});
```

## Collection.find()

If you have exposed your Collection and provided a firewall. Then the find() method will be extended, allowing an additional userId field.

```
Collection.find(filters, options, userId)
```

{% pullquote 'warning' %}
If *userId* is undefined. Firewall will not be called and applied. Use *null* or *false* for non-logged-in users.
{% endpullquote %}

## Exposure.restrictFields()

Restrict certain fields, and remove them from filters and options deeply and securely.
It will even clean-out filters with $and, $or, $nor, $not. 

```
import { Exposure } from 'meteor/cultofcoders:grapher';

Users.expose({
    firewall(filters, options, userId) {
        Exposure.restrictFields(filters, options, ['services'])
    }
});
```

## Firewalls are linked

{% pullquote 'warning' %}
If you specify a body to your exposures, in that case firewalls will not be linked, they will be bypassed.
{% endpullquote %}

```
{
    posts: {
        title: 1,
        comments: {
            text: 1
        }
    }
}
```

If *comments*'s collection is exposed. The firewall of comments will be applied.

Example:

```
Posts.expose();
Comments.expose({
    firewall(filters, options, userId) {
        filters.userId = userId;
    }
})
```

Then this query:

```
{
    posts: {
        comments: {}
    }
}
```

Will only return posts and comments for which userId is the current logged in user.

When exposure rules become too complex, the best solution is to use [Named Query](/api/namedQuery.html)