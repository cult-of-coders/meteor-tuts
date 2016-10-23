---
title: Query
description: Reference API for Grapher Query
---

## Collection.createQuery()


Available $filters: http://docs.meteor.com/api/collections.html#selectors

Available $options: *sort*, *skip* and *limit* 

```
const query = Posts.createQuery({
    $filters: {
        isPublished: true
    },
    $options: {},
    $filter({filters, options, params}) {
        // modify filters and options don't override
    }
    title: 1,
    nestedField: { // {} can be used for nested fields
        nestedFieldItem: 1
    }
    comments: {
        $filters: {
            isApproved: true // only retrieves the comments linekd to the post, that have isApproved: true
        }, // will only be applied to linked elements
        $options: {}, // will only be applied to linked elements
        $filter({filters, options, params}) {
            // params is global at query level, so params here are the same as params from the parent
        }
        linkField: 1
    }
}, params)
```

## createQuery()

```
import { createQuery } from 'meteor/cultofcoders:grapher';

const query = createQuery({
    posts: {
        $filters: {},
        $options: {},
        $filter({filters, options, params}) {
            // modify filters and options don't override
        }
        title: 1,,
        nestedField: { // {} can be used for nested fields
            nestedFieldItem: 1
        }
        comments: {
            $filters: {}, // will only be applied to linked elements
            $options: {}, // will only be applied to linked elements
            $filter({filters, options, params}) {
                // params is global at query level, so params here are the same as params from the parent
            }
            linkField: 1
        }
    }
}, params)
```

## Query.fetch()
```
// server-side
query.fetch() 

// impersonate user to work with exposure
query.fetch({userId: someUserId})
```

```
// client-side
query.fetch((err, data) => {
    // do something with data
})
```

## Query.fetchOne()

Same as fetch() but it always returns one result. Works very well when you want to retrieve a single item.

## Query.setParams()

```
query.setParams(params).fetch()
```

## Query.subscribe()

This will subscribe to the current query configuration. If you want to change parameters. You will call *setParams()*, then subscribe again. 

```
// client-side
query.subscribe();
query.fetch(); // no callback needed, it will fetch from client-side collections
```

## Query.unsubscribe()

```
// client-side
query.unsubscribe();
query.fetch(callback); // this time it needs callback because the query is now unsubscribed
```

## Query.clone()

When you import your query, any changes you do to the parameters will stick. This is why it is recommended to clone it where you use it,
so it becomes isolated.

```
const usableQuery = query.clone(newParameters);
```

Another approach would be to export a factory function of createQuery like

```
export default () => {
    return createQuery(body);
})
```