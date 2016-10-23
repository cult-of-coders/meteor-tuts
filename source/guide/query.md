---
title: Using Query
description: How to create and fetch queries
---

## Introduction

Queries are a way to specify which data you want from the server using links as the backbone for creating the data graph.
Queries can be reactive (using Meteor's pub/sub system) or static (using method call) or direct if the call is done server side.

Let's configure some links, and then see how we can query them into the database.

Assuming we have these collections: Authors, Comments, Posts, Groups, Category:

- Author has many posts.
- Author can belong in many groups.
- Posts has many comments.
- Posts has one category.
- Comments has a single author.

### Don't panic! 
We'll start defining our links, if something stops making sense. Review the [Collection Links](/guide/links.html) documentation again.

```
Authors.addLinks({
    groups: {
        collection: Groups,
        field: 'groupIds',
        type: 'many'
    },
    posts: {
        collection: Posts,
        inversedBy: 'author'
    },
    // Resolver links only work on server-side or with non-reactive queries
    likesOnFacebook: {
        // in this case resolver will receive: object, filters, options, userId
        // since resolver is run server side, the author will be the full object with all the fields.
        resolver(author) {
            // do a sync call to retrieve the likes on facebook using author object.
            return count;
        }
    }
});

Posts.addLinks({
    author: {
        collection: Authors,
        type: 'one',
        field: 'authorId'
    },
    // it can have a lot of comments, so it's safer if we store the link in Comments collection
    comments: { 
        collection: Comments,
        inversedBy: 'post'
    },
    category: {
        collection: Categories,
        type: 'many',
        field: 'categoryIds'
    }
});

Comments.addLinks({
    author: {
        collection: Authors,
        field: 'authorId'
    },
    post: {
        collection: Posts,
        field: 'postId'
    }
});

Category.addLinks({
    author: {
        collection: Authors,
        inversedBy: 'category',
    }
});

Groups.addLinks({
    authors: {
        collection: Authors,
        inversedBy: 'groups'
    }
});
```

Perfect. Now that we defined our relationships we can query our database.
Assuming we exposed "Posts" server-side, we can fetch the query client-side.

{% pullquote 'warning' %}
Notes:

- Use {} to specify a link, and 1 for a field.
- "_id" will always be fetched
- You must always specify the fields you need, otherwise it will only fetch _id
{% endpullquote %}


```
const query = Posts.createQuery({
    title: 1,
    // if there are no custom fields specified, it will retrieve all fields.
    author: {
        // if you have a nested object, (no link named profile) it will not try to fetch the link, but rather give you only the fields you need.
        profile: {
            firstname: 1
            lastname: 1
        },
        likesOnFacebook: 1
    } 
    comments: {
        text: 1,
        // if you don't specify any local fields for the author, only "_id" field will be fetched
        // this will enforce the use of query and retrieve only the data you need.
        author: {
            groups: {
                name: 1
            }
        }
    }
});
```

Now that we have created our query, we have two options of fetching the data.

## Static Queries

Static methods receive their data using a method call to the exposure.

```
query.fetch((error, response) => {
    // if no error occured, the response will look something like this
    [
        {
            _id: 'XXXXXXXXXXXXXXX',
            title: 'Hello World!',
            author: {
                profile: {
                    firstname: 'John',
                    lastname: 'Smith'
                }
                likesOnFacebook: 200
            },
            comments: [
                {
                    text: 'Nice Post',
                    author: {
                        _id: 'XXXXXXXXXXXXX'
                    },
                    groups: [
                        {
                            _id: 'XXXXXXXXXXX',
                            name: 'Group 1'
                        }
                    ]
                }
            ]
        },
        ...
    ]
});
```

## Reactive Queries

```
const subsHandle = query.subscribe();
const data = query.fetch();

query.unsubscribe();
query.fetch(); // now it will fail because you did not provide a callback, because when you unsubscribe, we delete the subscriptionHandle
```


{% pullquote 'warning' %}
Important! If you previously subscribed, fetching will be done client side using client-side collections,
if you did not previously subscribe, you need to provide a callback because data will be fetched via a method call.
{% endpullquote %}

If you don't want to use .fetch() you can also use the collections as you previously used to:

```
Posts.find().fetch()
Comments.find({postId: 'XXXXXX'}).fetch()
```


## Creating a query without the collection object

```
import { createQuery } from 'meteor/cultofcoders:grapher';

createQuery({
    posts: {
        comments: {
            text: 1
        }
    }
});
```

*posts* is the name of the collection, specified as the first parameter in the *Mongo.Collection* constructor.

## $filters and $options

At any given collection node in your query you can specify different filtering techniques.

Available $filters: http://docs.meteor.com/api/collections.html#selectors

Available $options: *sort*, *skip* and *limit* 
Read more: http://docs.meteor.com/api/collections.html#Mongo-Collection-find

```
const query = Posts.createQuery({
    $filters: {isApproved: true} // this will find only posts that have isApproved: true
    $options: {
        limit: 100
    }
    title: 1
    comments: {
        $filters: { // this will only search the comments that have isNotSpam: true
            isNotSpam: true
        }
    }
});
```

## Parameters

You can pass params to your query, they will be available in every $filter() function.
Using $filter() gives you enough control to filters and options. So $filters and $options may be omitted.

```
const query = Posts.createQuery({
    $filter({filters, options, params}) {
        filters.isApproved = params.isApproved
    }
    title: 1
    comments: {
        $filter({filters, options, params}) {
            if (params.allowSpamComments) {
                filters.isNotSpam = undefined; // $filter() overrides $filters and $options
            }
        }
        $filters: { // this will only search the comments that have isNotSpam: true
            isNotSpam: true
        }
    }
}, {
    isApproved: true,
    allowSpamComments: true
});

```

Control parameters however you wish:
```
query.setParams({
    allowSpamComments: false
});
```

Using it with *React* and *react-meteor-data* package:

```
import { createContainer } from 'meteor/react-meteor-data';
import query from './listPostsQuery.js';

const localQuery = query.clone();

export default createContainer(() => {
    const handle = localQuery.subscribe();
    const posts = localQuery.fetch();
    
    return {
        isReady: handle.isReady(),
        posts: posts
    }
}, PostList);
```

{% pullquote 'warning' %}
We use .clone() method to avoid making changes on the actual query (like setParams). 
You could do is export a factory that creates your query on the fly, however, for simplicity,
we created a .clone() method, that clones the query and it is completely isolated.
{% endpullquote %}

{% pullquote 'warning' %}
When you have metadata links. A $metadata object will be stored to the children referncing the $metadata from the parent.
{% endpullquote %}

## Security and Performance

By default the options "disableOplog", "pollingIntervalMs", "pollingThrottleMs" are not available on the client.
You can control them in the firewall of your exposure.

Grapher is very performant. To understand what we're talking about let's take this example:
```
{
    users: {
        posts: {
            comments: {
                author: {}
            }
        }
    }
}
```

For a query like this, if we would've first received the users, then the posts for each user, then the comments for each post, then the author for each comments.
We would've blasted MongoDB with a lot of queries, and the number of queries increases exponentially. It can easily be around ~2000 for a simple query.

However, using the *Hypernova* module, an innovative approach, the query above is executed with only 4 queries. 
Because we have 4 collection nodes: "users", "posts", "comments" and "author"

It does this by aggregating filters and reassembles data locally.

From 2000 queries to 4 queries we experienced around 40x percent performance boost. 

## React Integration
For integration with React try out [cultofcoders:grapher-react](https://github.com/cult-of-coders/grapher-react) package

