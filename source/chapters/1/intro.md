---
title: 'Intro'
description: Getting started with Meteor
disqusPage: 'Chapter 1: Intro'
---

## Installing the framework
In order to be able to develop with Meteor, you will need three basic components: Nodejs, NPM and the Meteor framework itself.
So, to get what you need, run the following command in your system terminal:

```
curl https://install.meteor.com/ | sh
```

This command will work on an Ubuntu based distribution, such as Linux Mint, Elementary OS and so on.
This will also work on a Mac!
To check that you really have installed what you needed, check the software versions of the components:
```
nodejs -v
npm -v
meteor --version
```
If you don't have errors after running these commands, congratulations! You can move on! 
However, if you have errors, check if you ran the commands correctly. 
Afterwards, Google is your best friend! 

On Windows, simply download the installer and follow the basic Windows installation procedure that most software has.
Here, instead of Bash, Terminator or iTerm, you can use Powershell or the good old Command Prompt.
The commands are mostly the same.

##Tools
The tool that works best for us, and the one which we recommend you use, would be WebStorm, from Jetbrains.
Students can get it for free, and other people can get a free 30-day evaluation trial.
You can download Webstorm from [here](https://goo.gl/gqmqwt).

To learn more about Webstorm go [here](https://www.jetbrains.com/webstorm/documentation/).

Another tool which is quite easy to learn and use would be Sublime Text. 
This is a code editor with a lot of features that will make your life a whole lot easier, 
if you're looking to get in the coding business.
You can download it from [here](https://www.sublimetext.com/3).

To learn how to use it properly,go [here](https://scotch.io/bar-talk/the-complete-visual-guide-to-sublime-text-3-getting-started-and-keyboard-shortcuts).

## Creating a project

After you have installed Meteor, you can easily create a new project by running the following command in your terminal:

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
Now, to view your work, type *http://localhost:3000* in your browser and hit Enter.
Yes ! It's **THAT** easy!

To stop the project from running, just stop the process in your terminal with Ctrl+c!

## Official resources

The [Meteor Guide](https://guide.meteor.com) contains a lot of projects recipes.
Don't dive directly into it, as it requires you to know a little bit about how Meteor works.

The [Meteor Docs](https://guide.meteor.com) describe the API and functionalities of the modules included with Meteor.

## Application folder structure

This is the basic folder structure that an application, such as the one you've just created, should be made up of:

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
// to create it quickly, run the commands (they should work just fine on Mac/Linux)
mkdir -p client server imports/ui imports/startup/client imports/startup/server imports/api
echo "import '/imports/startup/client';" > client/main.js
echo "import '/imports/startup/server';" > server/main.js
```


Everything in "/client" and "/server" is automatically loaded, as it is explained [here.](https://guide.meteor.com/structure.html#load-order)

That's all great ! However, we need more control over our application, and in order to gain that control,
we will refrain from using auto-loading in our projects.

Everything in "/imports" must be explicitly imported so Meteor can "gain knowledge" about it.
This gives us the ability to build modular applications.
[Read more about this here.](https://danmartensen.svbtle.com/build-better-apps-with-es6-modules)


## Importing from NPM

For now, we won't get into a lot of details, but the basic idea is that with Meteor you have access to the full NPM ecosystem, allowing you to import
modules from http://www.npmjs.com
As an idea, to get you to realise just how many resources you have at your disposal, the NPM system contained roughly 445 797 packages on May 1st of 2017.

```
// "moment" is a library you will most likely use when working with dates
// This is how you install it:
meteor npm install --save moment
```
We use save here because it will save it in our packages.json, meaning that if we work in collaboration with other developers, 
 when they download the project to start working on it,they will have the same package with the version you specified, making their life much easier. 

To use "moment" in your project, at a basic level, use this code snippet:
```
import moment from 'moment';

moment(new Date()).format('YYYY-MM-DD')
```

## Atmosphere

Atmosphere is a package manager which is specific to Meteor only. As such, it will help you manage project dependencies with ease.
You can find great resources to help you with Collections, Server-Side Routes, Mailing, etc.

You can learn more about it on the project webpage: https://atmospherejs.com

Adding a package is as simple as:
https://atmospherejs.com/twbs/bootstrap

```
meteor add twbs:bootstrap
```

That's it. You now have added bootstrap in your application. And it's ready for you to use it!
There are a lot of cool and useful packages out there that we'll explore during these tutorials.

## Importing from Atmosphere
You also can use the modular approach with Meteor packages:

As a sample, we are using SimpleSchema, a package that allows us to easily validate objects.
This is the github page of the project: https://github.com/aldeed/meteor-simple-schema

```
//type into your console
meteor add aldeed:simple-schema
```

```js
//to use SimpleSchema, add this snippet to any of your source files:
import { SimpleSchema } from 'meteor/aldeed:simple-schema';

const schema = new SimpleSchema({
    title: {
        type: String
    }
})
```

If the package exports the objects, you will be able to access them by importing from the package, but note that we are prefixing it with "meteor/".

## Importing from local files

As you saw in the example for creating the project folder structure, we have the ability to import from local files by using an "absolute path"
```
import X from /imports/something.js
```

```
// relative path
// given you have two files: /imports/api/X.js and /imports/api/Y.js
// in Y.js file you can use a relative path (not a lot of details in that file path) :
import X from './X.js'

// but you could also use an absolute path ( all the details, no room to wiggle out of it :-) )
import X from '/imports/api/X.js'

// or you can use ".." to specify the previous folder
import X from '../api/X.js'

// you can even omit the ".js" part
import X from './X';

// if you have a folder that contains "index.js", by importing the folder you will actually import "index.js" from that folder
import Z from './Z';

// if Z is a file (Z.js) it will import it, if Z is a folder it will try to import Z/index.js
```

Pretty easy right ?

Would you like to find more about importing and exporting ? Read more here:
- http://exploringjs.com/es6/ch_modules.html
- https://stackoverflow.com/questions/25494365/es6-module-export-options/34842087#34842087


## Need Help ?

If you need help, or you get stuck you can ask other Meteor evangelists out there:
- https://forums.meteor.com
- #meteor channel on FreeNode Server: [https://webchat.freenode.net/](https://webchat.freenode.net/)

Make sure you Google your questions first, to find out if somebody had the same problem as well ( and most likely has found an answer to that problem ), before asking the community!


