---
title: 'Hello'
description: Getting started with Meteor
disqusPage: 'Chapter 1: Intro'
---

## Installing the framework

To install the framework, please follow the steps from the meteor official website: [meteor.com](https://www.meteor.com/install)

## Tools
The tool that works best for us, and the one which we recommend you use, would be WebStorm, from Jetbrains.
Students can get it for free, and other people can get a free 30-day evaluation trial.
You can download Webstorm from [here](https://www.jetbrains.com/webstorm/download).

To learn more about Webstorm, [go here](https://www.jetbrains.com/webstorm/documentation/).

A free alternative to Webstorm would be VSCode. This is an extensible, feature-rich code editor, that is
relatively easy to learn.  
It will make your life a whole lot easier, if you're looking to get in the coding business.
You can download it from [here](https://code.visualstudio.com/).

## Creating a project

After you have installed Meteor, you can easily create a new project by running the following command in 
your terminal:

```
meteor create myProjectName
```

## Start Meteor

Now you have to start up the project you've just created by running the following commands:

```
cd myProjectName
meteor run
```

That's it! You've just created your first project in Meteor!
Now, to view your work, type *http://localhost:3000* in your browser's address bar and hit Enter.
Yes ! It's **THAT** easy!

## Application folder structure

This is the basic folder structure that an application, such as the one you've just created, should be made up of:



```bash
client
    main.js # this file contains: /imports/startup/client
db # This is where our persistence layer is. Including external APIs
imports
    ui # contains anything User-Interface related
    startup
        client
            index.js # loads everything that is needed for the client to function (ex: routes, css, anything concerning the client)
        server
            index.js # loads everything that is needed for the server to function
    api # contains the rest
    server
        main.js # this file contains: import /imports/startup/server
```
As a reminder, you could always use the github repository. For that, go [here](www.github.com)

Everything in "/client" and "/server" is automatically loaded, as it is explained [here.](https://guide.meteor.com/structure.html#load-order)

That's all great, however, we need more control over our application, and in order to gain that control,
we will refrain from using auto-loading in our projects.

This compiles all the modules we specified into our project with the "import" keyword, when we type **"meteor run"** 
into the command line, but does not load all of them.  

Because of this, everything in "/imports" must be explicitly imported so Meteor can "gain knowledge" about it.
This also gives us the ability to build modular applications.
[Read more about this here.](https://danmartensen.svbtle.com/build-better-apps-with-es6-modules)


## Importing from NPM

For now, we won't get into a lot of details, but the basic idea is that with Meteor you have access to the
 full NPM ecosystem, allowing you to import modules from http://www.npmjs.com.

"moment" is a library you will use when working with dates in your projects.

Now, open a new terminal (leave the one with server opened) and in the root directory of the project we are working on, write

```
meteor npm install --save moment
```

We use save here because it will save it in our packages.json, meaning that if we work in collaboration with
 other developers, when they download the project to start working on it, they will have the same package 
 installed, with the version you specified, making their life much easier. 

To use "moment" in your project, at a basic level, use this code snippet:

```
// file: (for example) /imports/startup/server/index.js
import moment from 'moment';

const date = moment(new Date()).format('YYYY-MM-DD')
console.log(date)
```

Now, you should see the current date printed on your terminal in case we added this code server side, otherwise on browser's console on client side

## Atmosphere

Atmosphere is a package manager which is specific to Meteor only, just like npm is for NodeJs. 
It will help you manage project dependencies with ease.
You can find great resources to help you with Collections, Server-Side Routes, Mailing, etc.

You can learn more about it on the project [webpage](https://atmospherejs.com).

Adding a package is as simple as typing this into the console:
```
meteor add accounts-base accounts-password

```

That's it. You now have added an user management package in in your application. And it's ready for you to use it!
There are a lot of cool and useful packages out there that we'll explore during these tutorials.

## Importing from Atmosphere
You also can use the modular approach with Meteor packages!

To import an atmosphere package we prefix it with `meteor/`

```js
import { Accounts } from 'meteor/accounts-base'

```

Don't worry too much about this, we are going to explore them later in this tutorial.

## Importing from local files

As you saw in the example for creating the project folder structure, we have the ability to import from 
local files by using an "absolute path":
```
import myService from '/imports/startup/server/myService';
```

You can also use a relative path:
```
import {sum} from './helpers';
```
In order to see how this exactly works in an example, you can go on the github repository, [here](www.github.com)
Pretty easy right ?

Would you like to find more about importing and exporting ? You can read more here:
- http://exploringjs.com/es6/ch_modules.html
- https://stackoverflow.com/questions/25494365/es6-module-export-options/34842087#34842087


## Need Help ?

If you need help, or you get stuck you can ask other Meteor evangelists out there:
- https://forums.meteor.com
- Go to the #meteor channel on FreeNode Server: [https://webchat.freenode.net/](https://webchat.freenode.net/)
- Contact Cult of Coders for consultancy, [here](https://www.cultofcoders.com/contact)

Make sure you Google your questions first, to find out if somebody had the same problem as well 
(and most likely has found an answer to that problem), before asking the community!


