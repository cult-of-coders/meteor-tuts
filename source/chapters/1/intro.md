---
title: 'Intro'
description: Getting started with Meteor
disqusPage: 'Chapter 1: Intro'
---

So you want to begin learning Meteor huh ?

You think you got what it takes ?

You think you are ready to learn the most beautiful and elegant JS Framework ?

<strong style="font-size: 1.4em">Yes, I am ready.</strong>

Cool, let's begin our journey.

## Installing

https://www.meteor.com/install

```
meteor run
```

That's it. This is how easy it is to get up and running with Meteor.

## Resources

[Meteor Guide](https://guide.meteor.com) contains recipes for a lot of stuff, don't dive directly into it, it requires you to know a bit about how Meteor works

[Meteor Docs](https://guide.meteor.com) describes the API and functionalities of the modules offered by Meteor

## Folder Structure

This is the default folder structure that we will use.

<pre>
├── client 
│   └── main.js // this file contains "import /imports/startup/client"
├── imports
│   ├── ui // contains anything User-Interface related
│   ├── startup
│   │   ├── client
│   │   │   └── index.js // loads everything that is needed for the client to function
│   │   └── server
│   │       └── index.js // loads everything that is needed for the server to function
│   ├── api // contains the rest
└── server 
    └── main.js // this file contains "import /imports/startup/server"
</pre>

Everything in "/imports" must be explicitly imported so Meteor can know about it.
The advantage of this is that it offers the ability to build modular applications.

Everything in "/client" and "/server" is eagerly loaded (automatically loaded) [Read more](https://guide.meteor.com/structure.html#load-order) 

Pretty straight forward right ?

Let's move forward.

## Importing from NPM

For now, we won't get into much details, but the idea is that with Meteor you have access to the full NPM ecosystem, allowing you to import
modules from http://www.npmjs.com

```
// moment is a library you will most likely use in order to manipulate dates however you want
meteor npm install --save moment
```

Use it:
```
import moment from 'moment';

moment(new Date()).format('YYYY-MM-DD')
```

## Importing from Atmosphere

You can access it on: https://atmospherejs.com

Atmosphere is for Meteor only packages. You can find great resources to help you with Collections, Server-Side Routes, Mailing, etc.
The sky is the limit.

Adding a package is as simple as:
https://atmospherejs.com/twbs/bootstrap

```
meteor add twbs:bootstrap
```

That's it. You now have bootstrap in your application loaded. There are many cool packages out there that we'll explore during these tutorials.

Ok. Let's dive into more details now.

You can use the modular approach with Meteor package also:

```
// in your console
meteor add aldeed:simple-schema
```
```
// in any file
import { SimpleSchema } from 'meteor/aldeed:simple-schema';

// use SimpleSchema object
```

If the package exports the objects, you will be able to access them by importing from the package, but prefixing it with "meteor/".

## Importing from local files

As you saw in the example for Folder Structure, we have the ability to import from local files by using an "absolute path"
```
import X from /imports/something.js
```

```
// relative path
// given you have two files: /imports/api/X.js and /imports/api/Y.js
// in Y.js file you can do:
import X from './X.js'
// but you could also do
import X from '/imports/api/X.js'
// or you can use ".." to specify the previous folder like:
import X from '../api/X.js'

// you can even omit the ".js" part
import X from './X';

// if you have a folder that contains "index.js", by importing the folder you will actually import "index.js" from that folder
import Z from './Z';
// if Z is a file (Z.js) it will import it, if Z is a folder it will try to import Z/index.js
```

Pretty easy right ?

Find out more about exports and imports:
- http://exploringjs.com/es6/ch_modules.html
- https://stackoverflow.com/questions/25494365/es6-module-export-options/34842087#34842087

We believe that is critical for you to understand how we define our "requirements" in a file, because this will be used very often.

Side note: For meteor packages, you don't actually have to import them, because they are pseudo-globals, which means, you can use what the package exports
as a global variable. For the example above with simple-schema we could easily have in a file:

```
// no importing SimpleSchema
new SimpleSchema(config);
```

However, it is not recommended, because if you use a code-linter (a thingie that will check your code syntax and what you use) it will burst out with failures,
because it found no reference to SimpleSchema, because you do not imported. Hard-core programmers are against globals, and for a good reason.

Meteor comes packed with MongoDB, Node, Npm, so you don't have to worry about any dependencies.




