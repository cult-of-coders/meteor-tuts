---
title: 'React'
description: Integrating React with Meteor
disqusPage: 'Chapter 2: React'
---

## Introduction

This chapter shows you how to integrate React into Meteor with absolute ease. But be warned! This is not a React tutorial, in order
to fully understand what's going on here check the following resources:

- https://camjackson.net/post/9-things-every-reactjs-beginner-should-know
- https://blog.risingstack.com/the-react-way-getting-started-tutorial/
- https://egghead.io/courses/react-fundamentals
- http://jaketrent.com/post/smart-dumb-components-react/

## Setting Up
```
meteor add cultofcoders:meteor-react-routing
meteor npm install --save react react-mounter react-dom 
```

## Architecture

As we discussed in the first chapter, we want to build modular apps and not rely on auto loading.
We will store all our react components in the `/imports/ui` folder.

Routing is a general concept. A router decides what to show to the user, and because we regard it
as a very important piece in our app, we shall put it in `/imports/routing`

## Routing

Please follow the README.md from: 
https://github.com/cult-of-coders/meteor-react-routing

After you have setup your files, created `/imports/routing/router.js`, `/imports/routing/index.js`, and you included the routing in your client/startup folder,
created your `/imports/ui/App.jsx` file, and now it's time to see this baby in action!

```
// file: /imports/ui/Home.jsx
import React from 'react';

export default class Home extends React.Component {
    render() {
        return <h1>Hello Reactive World!</h1>
    }
}
```

```
// file: /imports/routing/index.js
import route from './router.js'

route('/', Home);
```

Now access it: http://localhost:3000/

## Methods

By default, if you read the tutorials on React, you realized that React is already reactive, by doing `this.setState` it will
rerender efficiently your Component.
 
Let's see how we would call a method and show a response in our component:

```
// file: /imports/ui/Home.jsx

export default class Home extends React.Component {
    constructor() {
        super();
        this.state = {loading: true, donuts: []}
    }
    
    componentDidMount() {
        Meteor.call('donuts.list', (err, res) => {
            this.setState({
                loading: false,
                donuts: res // assuming the method returns an array of donuts
            })
        })
    }
    
    render() {
        if (this.state.loading) {
            return <div>Waiting for the method</div>
        }
        
        return (
            {this.state.donuts.map(donut => <div>{donut._id}</div>)}
        )
    }
}
```

Try it out, see if it works.

## Publications

For publications we need Meteor's reactivity, the Tracker, that we discussed about it in Chapter 1. However,
fortunately we don't have to reinvent the wheel for it, because Meteor has created a package that allows us
to easily do it:

https://guide.meteor.com/react.html#using-createContainer

```
// terminal
meteor add react-meteor-data
```

```
// assuming you have a publication 'donuts' that returns Donuts.find()
// file: /imports/ui/Home.jsx

import { createContainer } from 'meteor/react-meteor-data';
import Donuts from '/imports/api/donuts/collection.js'

class Home extends React.Component {
    render() {
        const {loading, donuts} = this.props;
        
        if (loading) {
            return <div>Waiting for the method</div>
        }
        
        return (
            {donuts.map(donut => <div>{donut._id}</div>)}
        )
    }
}

export default createContainer((props) => {
    const handle = Meteor.subscribe('donuts');
    // reactive data sources will get tracked here
    // any change will result into a re-render of "Home" component
    
    return {
        loading: !handle.ready(),
        donuts: Donuts.find().fetch()
    }
}, Home)
```

Go ahead, try adding a new donut from the Meteor's shell. You'll see it updated live.

This is it. It's just this simple. You now have the knowledge of integrating Meteor & React. Now, we need to
master it.

In the homework you'll most likely be dealing with forms, here are some starting points:
- http://www.material-ui.com/
- https://github.com/christianalfoni/formsy-react
- https://github.com/mbrookes/formsy-material-ui 

But you can use everything you want.

There is a package that leverages work for you:
https://github.com/meteor-utilities/react-list-container

## Homework

#### 1. Log me in!
Create a route `/login` and a `Login.jsx` file. In it you will have a form. When the form is submitted, 
it will call `Meteor.loginWithPassword` and redirect him to "/" [Hint 1](/)

#### 2. Private Donuts!
Only allow logged in users to view the donuts list, it is very private. [Hint]()

#### 3. Register
Create a registration, use email and password for now.

#### 4. Create a donut
As a logged in user, present a form and createa donut via the Meteor.call(). You can do it in a separate route, or
where you list the donuts. Whatever you choose, create another component. Always think in components.

## Over-achievers

#### 1. Filters
In the donuts list, add a checkbox, when you click it it will only show the donuts with price < 200. Make sure filter
is in another component, when something changes, your subscription should change. This means you may need a container
over on top of your container!

#### 2. Pagination (By Methods) (From scratch)
Implement a pagination strategy (You can use: https://www.npmjs.com/package/react-paginator). Where you display
all the donuts, 3 items per page. *Quick tip*: You may need to create another method that retrieves the count for all donuts.

#### 3. Pagination (By Subscription) (From scratch)
This one is a bit tricky. But not impossible, if you want a true reactive pagination (which in most cases you don't) but you want 
it know because you're an over-achiever, check this meteor package for reactive counting:

- https://github.com/percolatestudio/publish-counts

