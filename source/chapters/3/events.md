---
title: 'Events'
description: Observe and react to changes in a decoupled manner 
disqusPage: 'Chapter 3: Events'
---

## The Problem

Let's start with the problem. Something happens in your system, like: you add a new item for sale.

When this happens you need to do:
- Notify people that may be interested in your sale
- Send an email to the Admin
- Charge the person 0.02$ for the fact he posted a new item for sale on your platform

So far so good, you know that you need to have units of logic (services) for this, so after some coding hours (or minutes!) you come up with these functions:

```js
notifyInterestedPeopleInSale(itemId);
notifyAdmins(itemId);
processCharging(itemId);
```

Now you go to your `ItemService` and have something like:
```js
static createItem(data) {
    const itemId = Items.insert(data);
    
    notifyInterestedPeopleInSale(itemId);
    notifyAdmins(itemId);
    processCharging(itemId);
}
```

And it seems that you are happy with this. It looks modular and decoupled. However it's not ok because:
- **It knows too much** about other services
- **It does too much**, it's purpose is to merely create an item, that's it.
- **It depends on too many modules**, if by any chance you decide to remove admin notifications you need to see wherever you use it and remove it.

Besides that, the name we have is not very verbose, what if we change it?
```js
createPostAndNotifyAdminsAndInterestedPeopleInSaleThenProcessCharging(data);
```

## The Solution

Ok we can't work with something like that, name too long, and we break the single responsability principle.
Is there any hope for us ?
Can we have good code when we have a lot of functionality ?

Ofcourse, let's rock and roll with the Observer pattern. The observer pattern is so simple to understand:
- When X happens I want to do Y
- Ability to say X happens
- Ability to do Y when X happens

In code translation:
```
meteor npm i -S event-emitter
```

PS: In Node 8 (Meteor 1.6), it is native: https://nodejs.org/api/events.html  

```js
// file: /imports/api/events.js
import EventEmitter from 'event-emitter';

const Emitter = new EventEmitter();

const Events = {
    ITEM_CREATED: 'item_created'
};

export { Emitter, Events }
```

Now we need to say to the system that an item has been created:
```js
import {Emitter, Events} from '/imports/api/events';

function createItem(userId, data) {
    const itemId = Items.insert(data);
    
    Emitter.emit(Events.ITEM_CREATED, {itemId, userId});
}
```

Now, notifications and payment are two modules that aren't necessarily related, they don't really need to know about each other.
This is why our listeners, should be close to their code:

```js
// file: /imports/api/notifications/listeners.js
import {Emitter, Events} from '/imports/api/events';

Emitter.on(Events.ITEM_CREATED, function({itemId}) { 
    notifyInterestedPeopleInSale(itemId);
    notifyAdmins(itemId);
})
```
```js
// file: /imports/api/payments/listeners.js
import {Emitter, Events} from '/imports/api/events';

Emitter.on(Events.ITEM_CREATED, function({itemId}) { 
    processCharging(itemId);
})
```

What do we exactly gain by using this strategy ?
- You can plug-in additional functionality by listening to events
- If item creation is done from multiple places, if you want to remove/add functionality you do it in one place
- It's easy to find what listeners you have if you respect the pattern, simply searching for `Events.ITEM_CREATED` in your project finds you everything you need (listeners, emitters)
- The code is independent and doesn't know about other functionality, which is the way it should be


Watch the pattern: `MODEL_ACTION: 'model_action'`. Begin all your event names with the subject of interest, and if the event name
also needs to contain the one who did an action, like the admin for example: `MODEL_ACTION_BY_ADMIN`

Be very careful with verb tenses, if it's present, then it's before the item creation, if it's past-tense it's after that action has been made:

```js
Emitter.emit(Events.ITEM_CREATE, {item});
const _id = Items.insert(item);
Emitter.emit(Events.ITEM_CREATED, {_id, item});
```



**Bottom line**
When we are doing non-related logic inside the service, just dispatch an event and hook in a listener. You are doing yourself a big favor!

## Event Params

When you emit an event, send an object as a single parameter, instead of multiple parameters or other types.
This gives verbosity to the code.

A common mistake is the fact that when you dispatch an event, you tend to send data,
which would help the listener, especially if it's your first one.

For example, you dispatch something like:
```js
Emitter.emit(Events.ITEM_CREATED, {
    itemPrice: X,
})
```

Because you want this data to be used in the payment processor. However, this is a bad way of sending events, events should be dispatched with object or objectId of their interest and other data
related to the Event itself.

Basically the rule is: when you dispatch events, imagine that you don't know about who is going to use them. There aren't any strict rules here.

When item is created I want to send:
- Item identification
- Who added this item

It's not bad to send the full item, it's up to you, my personal preference is to send ids as much as I can, but there
are ofcourse scenarios where I'd rather send the full `item` object, because I know a lot of listeners will need it.

Doesn't matter if you do 3 additional finds in your listeners. Most of the times when you do the finds, you will use [Grapher](http://grapher.cultofcoders.com),
and fetch only the fields you need. 

Aren't Events just great ?

## When to use

You can use them in most situations but I strongly recommend using them when:
- You have to create notifications of any kind (push, app, emails)
- You have a service that knows too much 
- You want to have something easily pluggable and upluggable in your system

## Validation & Maintenance

How do I enforce validation for events parameters ?

Again, you have to keep things simple, but if your code-base grows a lot, this will be needed,
especially if events are dispatched from multiple places. You need to be sure that the events receive valid parameters.

Just a simple way to architect this:
`/imports/api/events.js` -> `/imports/api/events/index.js`

```js
// file: /imports/api/events/events.js

const Events = {
    ITEM_CREATED: {
        key: 'item_created',
        schema: {
            itemId: {type: String}
        }
    }
};

export default Events;
```

```js
// file: /imports/api/events/events.js
import SimpleSchema from 'simpl-schema';
import _ from 'underscore';

const Events = {
    ITEM_CREATED: {
        key: 'item_created',
        schema: {
            itemId: {type: String}
        }
    }
};

// put this in a separate module, it's for demo here
// we create SimpleSchema validator function for each event with schema 
_.each(Events, (value, key) => {
    if (!Events[key].schema) {
        return;
    }
    
    Events[key]['validator'] = new SimpleSchema(Events[key].schema).validator();
});

// we also need the old simple Events style so we can easily use it in our app
// so we don't have to do Events.ITEM_CREATED.key when dispatching or listening
let SimpleEventsMap = {};
_.each(Events, (value, key) => {
    SimpleEventsMap[key] = value.key;
});

export default SimpleEventsMap;
export {Events};
```

```js
// file: /imports/api/events/extendForValidation.js
import {Events} from './events';

export default (emitter) => {
    const oldEmit = emitter.emit;
    const newEmit = function (event, data) {
        if (!Events[event]) {
            return;
        }
        
        if (Events[event].validator) {
            Events[event].validator.validate(data);
        }
        
        oldEmit(event, data);
    };
    
    emitter.emit = newEmit.bind(emitter);
    
    return emitter;
}
```

```js
// file: /imports/api/events/index.js
import Events from './events';
import EventEmitter from 'event-emitter';
import extendForValidation from './extendForValidation';

const Emitter = extendForValidation(
    new EventEmitter()
);

export default Emitter;

export {Emitter, Events};
```

## Testing

Event listeners **must** delegate their job to services directly, they are proxies. Event Listeners should not contain any logic. Create unit-tests for services,
and then you can run an integration test easily.

If you do want to test them, use the strategy in the [**Services - Dependency Injection**](/chapters/3/services.html#Dependency-Injection) and make your listener a class,
and inject the services it uses.
