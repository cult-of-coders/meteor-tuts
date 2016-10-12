---
title: Grapher Live
description: A meteor package that allows you to test your queries live.
---

### Install

```
meteor add cultofcoders:grapher-live
meteor npm install --save-dev react react-ace
```

### Server Side Setup
```
// in your /imports/startup/server
// don't initialize it in your production environment, or it will lead to unwanted data exposure.

import { initialize } from 'meteor/cultofcoders:grapher-live';

initialize(); // exposes a method "grapher_live", used by the React Component
```

### Client Side Setup
```
// client side expose a route using your router
// example with FlowRouter
// don't initialize this in production environment

import { render } from 'react-dom';
import { GrapherLive } from 'meteor/cultofcoders:grapher-live';

FlowRouter.route('/grapher', () => {
    render(<GrapherLive />, document.getElementById('app')); 
    // where app is your main id to render stuff (you can also use "render-target")
})
```

### Usage

Grapher Live uses *createQuery*:

```
// query
{
    tasks: {
        $filter({filters, params}) {
            filters.isChecked = params.isChecked;
        }
    }
}

// params example
{
    isChecked: true
}