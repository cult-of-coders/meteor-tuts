---
title: Introducing to Grapher
description: Grapher is a collection relationship manager. 
---

## What is Grapher?

Grapher is a Meteor package that:

- Allows Collections Relationship Management with an easy and intuitive API
- Allows you to securely expose collections client-side
- Allows you to fetch the data graph by JS objects.
- Offers the same API for Reactive and Non-Reactive queries.
- Integrates with your existing Meteor projects, MongoDB relations and SimpleSchema


## Modules

Grapher is separated in 3 main modules:

- Collection Links
- Query Fetcher (Reactive + Non-Reactive)
- Collection Exposure

These 3 modules work together to create the absolute tool to fetch linked data. It is our answer to the NoSQL walls of related data.

Below we have an introduction to each module. If you want to read more, read the guide.

### Collection Links

This module is a collection relationship manager, lets you specify linking information between collections, gives you ability to easily manipulate these links, and provides an easy way to fetch them.

This will help you create and manipulate relationships between collections.
In addition to that, you also have the ability create *resolver* links that can make REST-API calls or link with any type of database.

When linking MongoDB collections, we support 4 different ways:

#### One

We use this when we want to reference a single Document. For example, a comment has one author. Or a post has one author.

```js
// Comment Document
{
    ...
    userId: 'XXX'
}
```

#### Many

We use this when we want to store multiple references to a collection. This can be viewed as a *Many-To-Many* SQL relationship, but not necessarily.
For example, a user may belong in multiple groups:

```js
// User Document
{
    ...
    groupIds: ['XXX', 'YYY']
}
```

#### One Meta
Meta comes from *metadata*. This is used when we want to store additional information about the link. 
For example, if a user belongs to a single group, we want to store the fact that he is an Admin or not.

This relationship can be emulated in different ways. We could also store via a *Many* relationship "adminIds" at Group Level.
There is no right or wrong. Just choose what's best fit.

```js
// User Document
{
    ...
    groupId: {
        _id: 'XXX', isAdmin: true
    }
}
```

#### Many Meta
Same as the scenario in One Meta, but in this case, a user can belong to multiple groups and he can be admin only to some.

```js
// User Document
{
    ...
    groupIds: [
        {_id: 'XXX', isAdmin: true},
        {_id: 'YYY', isAdmin: false}
    ]
}
```

To emulate that you can also have done it in Group Document:

```js
// Group Document
{
    ...
    userIds: [
        {_id: 'XXX', isAdmin: true},
        {_id: 'YYY', isAdmin: false},
    ]
}
```

[Read more about how to link and fetch related collections in the guide.](`guide/links.md`)

### Query Fetcher

This module is used for fetching the data graph server-side or client-side. There are many interesting about this but the general idea is
that you can request only the data you need, for example:

```js
const query = Users.createQuery({
    profile: 1,
    emails: 1,
    groups: {
        name: 1
    }
})

const data = query.fetch()
```

Sample response:

```
[ 
    {
        _id: '',
        profile: {},
        emails: [],
        groups: [
            name: ''
        ]
    },
    ...
]
```

#### Query Features

- Dynamic queries configured via a set of parameters
- Filters (Mongo Filters) and Options (Sorting, Limits) at any level you want
- Very high performance with large datasets.


### Collection Exposure

This module provides a way of securing access to the data by user and exposes the collection client-side.

```js
// server-side only
import Users from '/imports/api/users/collection.js';
import { Exposure } from 'meteor/cultofcoders:grapher';

Users.expose({
    firewall(filters, options, userId) {
        if (!isAdmin(userId)) {
            filters.userId = userId;
        }
    }
    maxLimit: 100, // enforces a maximum limit
    maxDepth: 5, // this restricts the depth of a graphed request
    restrictedFields: ['services'] // removes unwanted fields from filters and options fields.
})
```

What it does it extends the *find* method of a Mongo.Collection to accept an *userId*:

```
import Users from '/imports/api/users/collection.js';

const data = Users.find(filters, options, userId).fetch();
```

{% pullquote 'warning' %}
If *userId* is undefined. Firewall will not be applied, because we don't want the firewall to interfere with secure server-side requests.
However, if it is *null* or anything else firewall will be applied.
{% endpullquote %}

Exposure is integrated with links and query. Meaning when you search for the links via *Query* or directly using *Collection Links* the firewall will be
applied automatically if that collection is *exposed*.
