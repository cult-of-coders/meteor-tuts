---
title: 'Static Type Checking'
description: Maintaining large projects
disqusPage: 'Chapter 3: Static Type Checking'
---

## Why?

Probably you have noticed it's a lot easier to develop new projects than support old ones. That's because of complexity that tends to grow fast as we add new features to the program. 

So, you might start breaking up your code into smaller modules, then add [testing](/chapters/3/testing.html)and maybe add some [linting](/chapters/3/linting.html). But what about the way those modules interact with each other? Wouldn't be great if we can define a contract that sets clear boundaries of what to except and enforce it? And that's where static type checking comes in. 

Also, Meteor has [some plans](https://github.com/meteor/meteor/pull/10522#issuecomment-511506265) to move on with TypeScript and is encouraging developers to help out with the [conversion to TypeScript](https://github.com/meteor/meteor/pulls?utf8=%E2%9C%93&q=is%3Apr+label%3AProject%3ATypeScript-conversion). But we'll be using [FlowType](https://flow.org/) which is supported by the [ecmascript package](https://atmospherejs.com/meteor/ecmascript). Flow is much easier to pick up and can be incrementally adopted. Don't worry if you can use one you won't face much trouble using the other. 

## Install

```js
meteor npm i --save-dev flow-bin eslint-plugin-flowtype
```

## Config

This is an opionated configuration.

Create `.flowconfig` file inside your project root:

```
[options]
# Absolute path support:
# e.g. "/collections/todos"
module.name_mapper='^\/\(.*\)$' -> '<PROJECT_ROOT>/\1'

# Meteor none core package support
# e.g. "meteor/kadira:flow-router"
module.name_mapper='^meteor\/\(.*\):\(.*\)$' -> '<PROJECT_ROOT>/.meteor/local/build/programs/server/packages/\1_\2'

# Meteor core package support
# e.g. "meteor/meteor"
module.name_mapper='^meteor\/\(.*\)$' -> '<PROJECT_ROOT>/.meteor/local/build/programs/server/packages/\1'

# Fallback for Meteor core client package
# e.g. "meteor/meteor"
module.name_mapper='^meteor\/\(.*\)$' -> '<PROJECT_ROOT>/.meteor/local/build/programs/web.browser/packages/\1'

# Structural mapping
module.name_mapper='^modules\/\(.*\)$' -> '<PROJECT_ROOT>/src/modules/\1'
module.name_mapper='^core\/\(.*\)$' -> '<PROJECT_ROOT>/src/modules/core/\1'
module.name_mapper='^ui\/\(.*\)$' -> '<PROJECT_ROOT>/src/client/\1'

[ignore]
# Ignore the `.meteor/local` packages that aren't important
.*/.meteor/local/build/programs/server/app/.*
.*/.meteor/local/build/programs/server/assets/.*
.*/.meteor/local/build/programs/server/npm/.*
.*/.meteor/local/build/programs/server/node_modules/.*
.*/.meteor/local/build/programs/web.browser/app/.*
.*/.meteor/local/build/main.js
.*/.meteor/packages/.*
```

If you're using vscode there're few [adjustments](https://github.com/flowtype/flow-for-vscode) to be made. And that's it, enjoy!