---
title: Using Named Query
description: How to secure your queries via Named Queries
---


## Introduction

Named Queries are queries which have their form locked on the server. The reason for their existence
is security. Imagine this scenario:
- You have an Admin that can see all Users
- You expose Users collection and check if he is Admin, you allow him
- You have a collection Order which is linked to an EndUser and a TeamMember (both in Users collection)
- As an EndUser when you see your order you want to see details about the assigned Team Member.
- As an EndUser you cannot directly interogate "users" end-point, because you secured it for Admin Only
- As an EndUser you cannot interogate "orders" and retrieve "teamMember" because exposures are linked

We have two options:
- Bad Way: Manipulate exposure in such a manner to see if the user is EndUser and has an Order and the ids he wants to see
are linked to an Order he has. But if you do that, then you may also need to restrict the fields he has access to and so on.
- Good Way: Use named queries

{% pullquote 'warning' %}
NamedQuery is not affected by exposure at all. It has it's own firewall.
{% endpullquote %}

## Our First NamedQuery

```
// /imports/api/queries/orderView.js
import { createNamedQuery } from 'meteor/cultofcoders:grapher';

const query = createdNamedQuery('ordersList', {
    orders: {
        $filter({ filters, params }) {
            filters.enduserId = params._id
        }
        details: 1,
        items: 1,
        teamMember: {
            profile: 1
        }
    }
})

export default query;
```

## Fetching on Server

We have two ways of handling this on the server-side.

### Importing the query

```
import query from '/imports/api/queries/ordersList';

const data = query.clone({_id: 'XXX'}).fetch();

```
{% pullquote 'warning' %}
We use .clone() method to avoid making changes on the actual query (like setParams). 
You could do is export a factory that creates your query on the fly, however, for simplicity,
we created a .clone() method, that clones the query and it is completely isolated.
{% endpullquote %}


### Using: createQuery

This version has been designed to make it work with [Grapher-Live](https://github.com/cult-of-coders/grapher-live),
so you can test your queries on the fly.

```
import query from '/imports/api/queries/ordersList';

const data = createQuery({
    ordersList: {
        _id: 'XXX' // Note these are the actual parameters.
    }
}).fetch()
```

## Fetching on Client

In order to fetch it client-side, you need to expose this query server-side.

```
// server-side code
import ordersListQuery from '/imports/api/queries/ordersList';

ordersListQuery.expose({
    firewall(userId, params) { // optional, but important!
        // throw exception or modify params to make it secure, in the example above we'll do
        params._id = userId
    }),
    method: true, // defaults to true, allows you to fetch it statically
    publication: true, // defaults to true, allows you to fetch it reactively,
    embody: { //optional
        // deep merges with the query body after the first node
        // this allows you to have full control over the query without exposing important business logic to the client
        teamMember: {
            $filters: {isActive: true}
        }
    }
})

// if you're just playing with it and want to see it works:
ordersListQuery.expose();
```

```
// client-side code
import ordersListQuery from '/imports/api/queries/ordersList';

const query = ordersListQuery.clone(params);

// statically (Works exactly like a Normal Query)
query.fetch((err, data) => {}); // Works exactly like a Normal Query

// reactively (Works exactly like a Normal Query)
const handle = query.subscribe(); 
query.fetch();
```

If you have imported the named query anywhere, then the name will be present in the store, meaning you can do:

```
// client-side code
createQuery({
    ordersListQuery: {params}
}).fetch((err, data) => {
    // do something with the data
});
```

It removes the boilerplate of you having to clone it. the reason for this was to allow it to work with [Grapher-Live](https://github.com/cult-of-coders/grapher-live)

## Why Collection Exposure ?

You might be asking at this stage, why do we still allow "flexible" queries via Collection Exposure. And it's a damn good question.
If your database is simple and can be easily secured then it is much simpler to avoid namedQueries and it's easier to expose an API,
through which a client can request what he wants only, without having a lot of named queries.