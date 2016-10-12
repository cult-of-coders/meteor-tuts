---
title: Linking
description: Reference API for Grapher Links
---

## Adding Direct Links

```
Posts = new Mongo.Collection('posts');
Comments = new Mongo.Collection('comments');

Comments.addLinks({
    post: {
        type: 'one'|'many',
        collection: Posts,
        field: 'postId' // optional, it generates a unique one
        metadata: {} // optional, use it for meta relationships
        index: false // optional, if set to true will create indexes for the links
        autoremove: false // if set to true it will remove the linked objects from database
        unique: false // optional, if you want a one-to-one kind of relationship 
    }
});
```

## Adding Inversed Links

```
Posts.addLinks({
    comments: {
        collection: Comments,
        inversedBy: 'post' // the name of the link in Comments
        // for inversed links these are the only options allowed
    }
});
```

## Adding Resolver Links

```
Posts.addLinks({
    commentsCount: {
        resolve(post) {
            // here you can put any sync call, like a REST API call
            return Posts.getLink(post, 'comments').find().count();
        }
    }
});
```

## Fetching Links

```
const postCommentsLink = Posts.getLink(postId, 'comments');
const commentPostLink = Comments.getLink(commentId, 'post');

commentPostLink.fetch(); // retrieves a single post object
commentPostLink.find().fetch(); // retrieves an array with one element called object.

postCommentsLink.fetch(); // retrieves all linked comments
postCommentsLink.find().fetch(); // retrieves all linked comments
```


{% pullquote 'warning' %}
*getLink* also accepts objects that contain an *_id*: Collection.getLink(post, 'comments')
{% endpullquote %}

When fetching the links you can specify additional filters and options:
```
postCommentsLink.find(filters, options).fetch(); // retrieves linked comments with specific filters and options
postCommentsLink.fetch(filters, options); // retrieves linked comments with specific filters and options
```

{% pullquote 'warning' %}
link.fetch() is smarter than link.find().fetch() because it may return a single result or array of results, depending on the type.
{% endpullquote %}

## Managing Links

{% pullquote 'warning' %}
When adding, removing links. Please not that the arguments acceptable are smart meaning you can provide:

- Single Object With *_id*
- String *_id* 
- Single Object without *_id* (It will also create it in the database)
- Array of Single Objects with *_id*
- Array of *_id*
- Array with any mixture: [objectWithId, objectWithoutId, _id]
{% endpullquote %}

{% pullquote 'warning' %}
When removing links, you must have objects or *_id* strings, otherwise it wouldn't make sense.
{% endpullquote %}

## Managing Links (One Relationship)

```
// for one relationships
link.set(relatedId);
link.unset();
```


## Managing Links (One Meta Relationship)

```
link.set(relatedId, {isSomething: true});
link.metadata(); // returns the metadata: // {_id: relatedId, isSomething: true}
link.metadata({isSomething: false}); // sets the metadata by extending the current metadata.
link.metadata({otherStuff: true}); // will update the metadata
link.metadata(); // returns the metadata: {_id: relatedId, someConditions: true, otherStuff: true}
link.unset(); // removes the link along with metadata.
```

## Managing Links (Many Relationship)

```
link.add(relatedId) // accepts as arguments: [relatedId1, relatedId2], relatedObject, [relatedObject1, relatedObject2]
link.add(objectWithoutIdProperty) // it will create the object and save it in the db and also link it
link.add([objectWithoutIdProperty, _id]) // it will create the object objectWithoutIdProperty and link it, and it will just link it with _id string

link.remove(relatedId) // accepts as arguments: [relatedId1, relatedId2], relatedObject, [relatedObject1, relatedObject2]
```

## Managing Links (Many Meta Relationship)

```
link.add(relatedId, {someConditions: true}); // it also works with [relatedId1, relatedId2, relatedObj1, relatedObj2]
link.metadata(relatedId) // {_id: relatedId, someConditions: true}
link.metadata() // [{_id: relatedId, someConditions: true}, {_id: relatedI2, someThingElse: false}, ...]
link.metadata(relatedId, {otherStuff: true}); // will update the metadata
link.metadata(relatedId) // {_id: relatedId, someConditions: true, otherStuff: true}
link.remove(relatedId); // will remove the link with the relatedId along with the metadata
```

## Managing Links - Inversed

This is an inversed link because the storage is stored in *Comments* collection.

{% pullquote 'warning' %}
When managing links from inversed side you are allowed to use set(), unset(), add(), remove(), 
to be able to use it in a natural language. postCommentsLink.set(commentId) wouldn't make much sense in this case.
{% endpullquote %}

```
postCommentsLink = Posts.getLink(postId, 'comments');

postCommentsLink.add(commentWithoutId) // will create a comment object in database and link it to post
postCommentsLink.add(arrayOfCommentsWithorWithoutIds) // for those without id it will create them and link them, for the others it will just link them

postCommentsLink.remove(commentWithId) // it will remove the links
postCommentsLink.remove(_id) // it will remove the links
```