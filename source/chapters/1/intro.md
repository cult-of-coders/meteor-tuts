---
title: 'Intro'
description: Getting started with Meteor
disqusPage: 'Chapter 1: Intro'
---

## Installing

https://www.meteor.com/install

## Creating a project

After you have installed Meteor, you can easily create a new project:

```
meteor create myProjectName
```

## Start Meteor

```
cd myProjectName
meteor run
```

That's it, now access http://localhost:3000
This is how easy it is to get up and running with Meteor.

## Resources

[Meteor Guide](https://guide.meteor.com) contains recipes for a lot of stuff, don't dive directly into it, it requires you to know a bit about how Meteor works

[Meteor Docs](https://guide.meteor.com) describes the API and functionalities of the modules offered by Meteor

## Folder Structure

This is the default folder structure that we will use.

<pre>
├── client 
│   └── main.js // this file contains: import /imports/startup/client
├── imports
│   ├── ui // contains anything User-Interface related (We won't cover UI in this Chapter)
│   ├── startup
│   │   ├── client
│   │   │   └── index.js // loads everything that is needed for the client to function (ex: routes, jQuery plugins, css, anything concerning the client)
│   │   └── server
│   │       └── index.js // loads everything that is needed for the server to function
│   ├── api // contains the rest
└── server 
    └── main.js // this file contains: import /imports/startup/server
</pre>

```
// prototype it quickly (works on Mac/Linux)
mkdir -p client server imports/ui imports/startup/client imports/startup/server imports/api
echo "import /imports/startup/client;" > client/main.js
echo "import /imports/startup/server;" > server/main.js
```


Everything in "/client" and "/server" is eagerly loaded (automatically loaded) [Read more](https://guide.meteor.com/structure.html#load-order) 

We won't use auto-loading in order to gain more control about the file order.

Everything in "/imports" must be explicitly imported so Meteor can know about it.
The advantage of this is that it offers the ability to build modular applications.
[Read more](https://danmartensen.svbtle.com/build-better-apps-with-es6-modules)


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

## Atmosphere

You can access it on: https://atmospherejs.com

Atmosphere is for Meteor only packages. You can find great resources to help you with Collections, Server-Side Routes, Mailing, etc.
The sky is the limit.

Adding a package is as simple as:
https://atmospherejs.com/twbs/bootstrap

```
meteor add twbs:bootstrap
```

That's it. You now have bootstrap in your application loaded. There are many cool packages out there that we'll explore during these tutorials.

## Importing from Atmosphere
You can use the modular approach with Meteor package also:

Sample we are using SimpleSchema, a package that allows us to easily validate objects:
https://github.com/aldeed/meteor-simple-schema

```
// in your console
meteor add aldeed:simple-schema
```

```
// in any file
import { SimpleSchema } from 'meteor/aldeed:simple-schema';

const schema = new SimpleSchema({
    title: {
        type: String
    }
})
```

If the package exports the objects, you will be able to access them by importing from the package, but note that we are prefixing it with "meteor/".

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


## Need Help ?

If you need help, or you get stuck you can ask other Meteor evangelists out there:
- https://forums.meteor.com
- #meteor channel on FreeNode Server: [https://webchat.freenode.net/](https://webchat.freenode.net/)

Make sure you Google your questions first, before asking the community!
