---
title: Meteor Snacks
description: Useful tools and features that Meteor has to offer.
disqusPage: 'Chapter 1: Meteor Snacks'
---

## Environment Variables

Meteor uses environment variables to know various things, such as which MongoDB database should it connect to, to whom 
it should send emails and other information, such as:
- MONGO_URL : you don't have to have this variable by default, but if you connect to another database you should have it here
- MAIL_URL : the smtp:// for your email. The one you use to interact with your users. We'll show you in a bit how easy it is to set it up.
- METEOR_PROFILE : if you set it to 1, you'll see how much time meteor spends on building/rebuilding your project
- ROOT_URL : the real path of Meteor(the default address is http://localhost:3000)

To specify these variables you would use the following snippet:
```
ROOT_URL="http://localhost:3000" MAIL_URL="smtp://gmail.com:25" meteor run
```

## Step into an asynchronous world-*Meteor.wrapAsync*
Since you are here, you know that Javascript is a single-threaded system, meaning that it can do only one thing at a time. 
This would imply the fact that a slow operation would “block” the system, and keep it all to itself while executing, 
and everything else would have to wait for the slow operation to finish before they can continue.

This is where asynchronous operations come into play! Asynchronous calls let us perform long-running operations without blocking the system.
 These operations can be network requests, disk reads/writes and so on.

**Meteor.wrapAsync* enables you to do asynchronous operations in your methods. Let's say you use Amazon S3, or an Npm library, 
that requires you to specify a callback. Well if you do it like this: 
```js
Meteor.methods({
    'something_async': function () {
        coolLibrary.coolFunction(function (err, res) {
            // it will get here after some time.
        })
    }
})
```
you may get a very weird error saying that "code cannot run outside "Fibers" ". 
Since we don't want to overcomplicate things, we won't go into details regarding the error message. 
But here's the right way of doing things...asynchronously, of course:

```js
Meteor.methods({
    'something_async': function () {
        const run = Meteor.wrapAsync(coolLibrary.coolFunction, coolLibrary);
        /* "coolLibrary" is there to provide context to the function, because if that function uses "this" inside of it, 
        then it will fail because the context was not specified.*/
        
        try {
            const results = run(); 
            // if the callback accepts err result will be synced with results.
        } catch (e) {
            
        }
        
        // some API's will not have callbacks with (err, res), but will instead give you other arguments with other order
        // for those edge-cases you may need to use an "async" syntax, just like here:
        
        const results = run((arguments, of, the, callback) => {
            // do something here
            // since this time you are inside a fiber, your application will not crash
        })
        
        return results;
    }
})
```

## Timers

You may already be familiar with *setTimeout*, *setInterval* from plain JavaScript.
Well, Meteor has them too, and they will run inside fibers. For example:

```js
Meteor.methods({
    'something_async': function () {
        Meteor.setInterval(() => {
            console.log('tick');
        }, 1000);
    }
})
```

After you called the method, you will get a 'tick' in your console, for every second passed. You will not be able to stop this, 
unless you restart the application or implement a handler that stops it, so be careful, because it unnecessarily 
takes up computing power from your system!

## Emails

Remember the emails that we received in the console when we were talking about users ? Well, in the background, the system used this module:
If you don't specify MAIL_URL, all the emails that you send will go to your console.

If you want an email for your system, we recommend [Mailgun](http://www.mailgun.com/). Pro tip: for less than 10 000 email per month, it's free!

Here, we use %40 to represent @ because we need URI encoding:
```
MAIL_URL="smtp://postmaster%40yourdomain.com:b23f5872166c187ad5b5f1abece071b2@smtp.mailgun.org:25" meteor run
```

This is an example that will demonstrate the most basic usage for your email:
```js
import { Email } from 'meteor/email';

Email.send({
  to: 'you@meteor-tuts.com',
  from: 'no-reply@meteor-tuts.com',
  subject: "I'm an email",
  html: '<p>Hello!</p>'
});
```

If you're interested to learn more about communicating with your users from your Meteor-based application, 
you can read more [here](http://docs.meteor.com/api/email.html).

## Meteor.defer

Sometimes, when you want to do something, and notify the user by email afterwards. However, if you use SMTP, sometimes
it can take between 50ms to 1s for the mail to be sent, and this can give the user the impression that your application is slow. 
This is why you should use this function:
```js
Meteor.methods({
    'action_plus_email': function () {
        // do something
        
        Meteor.defer(() => {
            Email.send(...)
        })
        
        return 'hello there, user';
    }
})
```

*Meteor.defer(fn)* is same as *Meteor.setTimeout(fn, 0)*. 
Basically, *Meteor.defer(fn)* will do a "background" job in a fiber (asynchronously...get it ? ).

## HTTP

"-Would you like an external REST API with that sir ?
 -Yes of course!
 -Coming right up!"
 This is how a conversation between a "hungry" developer and Meteor would look like, if the developer wanted to know if Meteor can serve him an external REST API.
  For that, Meteor has a simple [HTTP](http://docs.meteor.com/api/http.html) package built-in.

```js
Meteor.methods({
    'an_api_call': function () {
        const data = HTTP.get('https://jsonplaceholder.typicode.com/posts/1')
        
        console.log(data);
        
        return data;
    }
})
```

## Assets

The best documentation for assets is available [here](http://docs.meteor.com/api/assets.html), at the official Meteor website!

Let's go ahead and put something in "/private/some_folder/test.txt" by running this in our Meteor shell:
```
Assets.getText('/some_folder/test.txt')
```

You would use this when, for example, you have a business, logic-packed csv, or an .xls file that you need to work with.
Or you may have a JSON with car models. 
The idea is that you can use any type of file, even binary, that you can use privately on the server where the application is deployed on.

## Meteor Settings

Again, the guys and girls at [Meteor](https://docs.meteor.com/api/core.html#Meteor-settings) explain this subject in the best way.
Use this code in your *.deploy/local.json* file:
```json
{
    "public": {
        "visible": "Something that the client can see"
    },
    "private": {
        "API_KEY": "XYZ"
    }
}
```

You can access the settings from the client-side with:
```
Meteor.settings.public.visible
```

Or you can access all of the settings from the server-side by using:
```
Meteor.settings.private.API_KEY
```

## Running Meteor - the easy way

Inside your "Meteor" folder you have a file named "package.json". That packages keeps track of what npm packages you use, and other
information about your system setup. So, for example, you may start an app with different settings than the ones you developed it with on your workstation.
 To restore those settings, you would use something like this:
```json
{
  ...
  "scripts": {
    "start": "MAIL_URL='xxx' meteor run --port 3000",
    "deploy": "We'll get into that in another chapter ;)"
  }
}
```

Now run this in your terminal:
```
npm run start
```
and you should be okay to start your application!
