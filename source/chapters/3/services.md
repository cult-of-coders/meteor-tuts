---
title: 'Services'
description: Using services to decouple your code
disqusPage: 'Chapter 3: Services'
---

## What is a service? 
A service is a unit of work (a function) or a group of very related functionality (a class)

Using services is linked to the [Single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle)

```simple-text
Q: So when do we need to use services?
A: Everytime!
Q: Where do we use services?
A: Everywhere!
```

Ok we're done here, everything is clear now, right?

Jokes aside, you should treat every functionality in your app as a service, for example:
You want to send an email, and ofcourse, you know how to do this, you use *Email.send*. Well, *Email* is actually a service, but now how you send it in your app ?

Let's say you create a Post, and you need to notify the Admin to approve it.
Where do you store that unit of logic ? Inside a Meteor method ? Inside a function ? Inside a class ?

The answer is: **NOT inside the Meteor method**

Usually, you tend to couple logic in your Meteor methods which is a very very bad terrible thing, because Meteor methods act as server-side routes, they act as the *C* (Controller) in *MVC*, 
the *C* does not contain logic, the *C* is a boss that delegates work to *M* (Services aka Models)

It is also bad because methods are a communication layer between the client and the server, they shouldn't store business logic, or logic of any kind, imagine them as proxies that communicate with your services.

Let's start focusing about services in depth, and understand some nice principles about
crafting them in a nice way.

Don't forget to read: https://github.com/ryanmcdermott/clean-code-javascript 

## Sample
```js
class ItemService {
    static createItem() {
        // put logic here for item creation
    }
    
    static updateItem(id, data) {
        const item = this._getItem(id)
        // do something with it
    }
    
    static _getItem(itemId) {
        // returns the item from database or throws an exception
    }
}

export default ItemService;
```

In most cases, you want your services to be a class with static methods, and not an instance of that class `new ItemService()`, however
we'll see below why in some cases using the instance makes more sense.

## Structuring

By default we are going to put them inside:
`/imports/api/{module}/services`

Usually {module} represents the collection name it handles it, but this is not always the case, it can represent something that is handling multiple collections to achieve its goal.

If your services need more decoupling feel free to nest them:
`/imports/api/{module}/services/{submodule}`

If you are writing HTTP APIs separate the Service of fetching/persisting data and put it in `/imports/db/{module}` and
the one that does include the app/business logic keep it in `/imports/api/{module}`. The one inside `/imports/api` should instantiate the one from `/imports/db` with the correct API keys (most likely retrieved from `Meteor.settings`)

Name your services with uppercase if classes (ItemService), or lowercase if functions (doSomething).
If your service is a class, suffix it with service, if it's a function, make the sure the filename is a verb.

Inside a function service module, you can create additional functions, but you must export only one, if you need to
export multiple functions, it will become a service class with static methods.

## Creating Services

Go API first. Don't try to think about the logic, try to think about how you are going to use it. This is why [**TDD**](https://technologyconversations.com/2013/12/20/test-driven-development-tdd-example-walkthrough/) works so well, because
it lets you think about the API and you also write a test, and passing tests means you finished implementation.

So, instead of patterns, try to think about how you would use it, and just write a test for it first. You will notice that your development speed will increase dramatically (after first tries). No kidding!

#### Helpful questions:
1. What would be the cleanest, easiest way to use this Service?
2. How can I make it so it's easier to understand by others?
3. What comments can I leave so the next developer that comes in understands this?
4. Does my service have a single responsability?
5. Is there any functionality in my service that is outside its scope so I can decouple it?

## Conventions

Again, don't forget to read: https://github.com/ryanmcdermott/clean-code-javascript. I'm not joking, read it! 

#### Functions should be actions or interrogations

Please favor longer names for small scope, lower names for large scopes.

```js
// BAD
object.author()
object.veryBad()
object.closing()
object.mkSmtGdAbtIt()
```

```js
// GOOD
object.getAuthor()
object.isGood()
object.close()
object.makeSomethingGoodAboutIt()
```

#### Variable names should be substantives or interrogations

```js
// BAD
let making;
let made;
```

```js
// GOOD
let isMaking;
let isMade;
```

#### Functions should be small

A function should not be larger than 10 lines. Honestly, if you have that, then it must be refactored. Exceptions being, large switch cases, Pure Components and render() functions from React.

#### Indent with 4 spaces

With JS and with JSX.
Why 4? Because it forces you to decouple and makes your spaghetti much more visible and annoying!
Plus it feels that the code is more airy and readable.

What about callback hell and stuff ?
1. Use promises, or the sexy async/await syntax
2. Decouple your callbacks

This is opinionated, there is no wrong way to indent, there are other factors far more important.

#### Classes hide in large functions

There are cases where you have a single function as service, and that function begins to grow,
and you find that decoupling it in multiple functions gets tedious because you will need to pass
many variables along, and you realize that those variables are strings or numbers and can't be properly modified,
so you need to return them, and even if your intentions are pure, your code will become
more messy.

The rule is simple: when it's difficult to decouple a function, create a class. Because inside a class, each method
has access to `this` context, so you no longer need to pass variables along.

#### Don't extend classes

Another opinionated advice, just don't use extend. The only exception is if the class you extend
is abstract (you don't instantiate/use it stand-alone).

#### Provide JSDoc and Comments

Whenever you feel like you are doing something that is not verbose, leave a comment, and also leave JSDoc, especially for 
variables. It becomes truly helpful when you are using an API written by someone else and the IDE shows you which variables it
needs and a description for them.

If you find a snippet, or something that describes a certain thing, don't be afraid to leave links,
especially if you're writing an facade to an external API, leave link to the docs, it will save you and other devs
time.

Leave as much comments as you can, but don't go off the grid, stay to the point. And only leave comments when necessary:

```js
// BAD (The code already tells you what you do)
// We are iterating through users 
users.forEach(user => {});
```
```js
// GOOD (Even if you can read it nicely, you need to understand the intention fast) 
// We are calculating the total cost of all products so we can use it in total calculation cost.
let totalCost = 0;
products.forEach(product => {
    totalCost += product.cost;
})
```


## Dependency Injection

I've looked around the NPM community and everything is too complex for what we really need.

Inversion principle is simple, let the service decide what dependencies it has. We are kinda doing this already
because each module imports what it needs in order to execute properly.

The reason for changing it a bit is the fact that it will allow easy testing. For example, we want to test that `PaymentService.charge` is been called when something happens.

```js
import PaymentService from 'somewhere';

class ItemService {
    static createItem(item) {
        Items.insert(item);
        PaymentService.charge(item.userId, 20.00);
    }
}

export default ItemService;
```

Now in my test how would I be sure that `PaymentService.charge` is called without actually altering `PaymentService`?
This requires a change in strategy, by injecting dependencies in the constructor:

```js
import PaymentService from 'somewhere';

class ItemService {
    constructor({paymentService}) {
        this.paymentService = paymentService;
    }
    
    createItem(item) {
        Items.insert(item);
        this.paymentService.charge(item.userId, 20.00);
    }
}

export default new ItemService({
    paymentService: PaymentService
});

export {ItemService}
```

Ok now if we would like to test it, we have access to the exported variable: `ItemService` and we can pass-in a [stub](http://sinonjs.org/releases/v4.0.1/stubs/) for `PaymentService` in its constructor.

The only problem with this pattern is the fact that it creates an instance without you wanting to,
therefore you can create a separate `ServiceModel` file  (ItemServiceModel.js) and instantiate (and export) it inside `ItemService.js`, and when you test
you only need to play with `ItemServiceModel`




 
 


