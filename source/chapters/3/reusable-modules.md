---
title: 'Reusable Modules'
description: How to create pieces of code designed for reusability
disqusPage: 'Chapter 3: Reusable Modules'
---

## Why?

Let's say you want to create a chat thingie, and you want to decouple it, because
you think that in the future you will definitely use it in some other project.

You need to treat it separately as a module:

Create a new folder `/imports/modules/chat/`

```js
// file: /imports/modules/chat/client.js
// Do things related to client initialization (if necessary)

// file: /imports/modules/chat/server.js
// Do things related to server initialization, methods, listeners, etc (if necessary)

// file: /imports/modules/chat/db
// Apply the same principles

// file: /imports/modules/chat/api
// Apply the same principles

// file: /imports/modules/chat/client
// Store frontend stuff here.
```

## Imports

Inside your re-usable module, never use absolute paths, only relative paths when importing:

```js
import {Threads, Messages} from ../../db
```

## README.md

Provide a README that specifies the module this uses, so it can be nicely reused.

```markdown
// file: /imports/modules/chat/README.md
// Store here the installation details and npm dependencies.
// Describe in general terms how to use this module
```

## Event handling

Use an independent event handler. Not the global one.

```js
// file: /imports/modules/chat/api/events.js

import EventEmitter from 'event-emitter';

const Events = {
    CHAT_NEW: 'chat.new'
}

export default new EventEmitter();
```

If you want to unify emitters you can do so simply:
https://www.npmjs.com/package/event-emitter#unifyemitter1-emitter2-event-emitterunify

But if you want to do app specific logic when something in this module happens, I recommend you use the EventEmitter from `chat` module to listen for events.


## A git submodule

Ofcourse now that you have it as a module, you can make it a git-submodule, and make it re-usable in multiple projects and maintainable.

Read more here:
https://git-scm.com/book/en/v2/Git-Tools-Submodules 

## Caution

The problem is that this app is not so independent, so you may find yourself conflicting with some npm dependencies,
so keep that in mind. It depends on your whole structure and how you use it.

You can avoid this by properly versioning your git submodule.


