---
title: 'React'
description: Integrating React with Meteor
disqusPage: 'Chapter 2: React'
---

## Introduction

This chapter shows you how to integrate React into Meteor. However, this is not a React tutorial! In order
to fully understand what's going on here, check the following resources:

- [9 things every ReactJS beginner should know](https://camjackson.net/post/9-things-every-reactjs-beginner-should-know)
- [The React way - getting started](https://blog.risingstack.com/the-react-way-getting-started-tutorial/)
- [React fundamentals](https://egghead.io/courses/react-fundamentals)
- [Smart and dumb components in React](http://jaketrent.com/post/smart-dumb-components-react/)

## The environment
Before we go through some React code, let's set up the environment for the project with these terminal commands:

```
meteor add cultofcoders:meteor-react-routing
meteor npm install --save react react-mounter react-dom 
```

## The code structure

As we discussed in the first chapter, we want to build modular apps that do not rely on auto loading.
We will store all of our React components in the `/imports/ui` folder.

A router decides what to show to the user, and because it is a critical component
 in our app, we will place it in `/imports/routing`.

## Routing and application flow

To better understand what we are about to do and look at some code samples, please go through the 
[README.md](https://github.com/cult-of-coders/meteor-react-routing) file that we maintain for our projects. 

After the setup you did previously with those 2 terminal commands, you will need to create a few files:
- `/imports/routing/router.js`
- `/imports/routing/index.js`
- `/imports/ui/App.jsx` 

Now include the routing in your client/startup folder, and you should be good to go!

Let's start with some code in `/imports/ui/Home.jsx` :
```js
import React from 'react';

export default class Home extends React.Component {
    render() {
        return <h1>Hello Reactive World!</h1>
    }
}
```
Now in `/imports/routing/index.js`:

```js
import route from './router.js'

route('/', Home);
```

And let's access [our work](http://localhost:3000/)!

## Methods

By default, React is already reactive, so by using `this.setState` it will
render again your component in a very efficient manner.
 
Let's see how we would call a method and show the response in our component with some code in `/imports/ui/Home.jsx`:

```js
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
            <div>
                {
                    this.state.donuts.map(donut => {
                        return <div key={donut._id}>{donut._id}</div>
                    })
                }
            </div>
        )
    }
}
```

## Publications

To use publications, we need Meteor's reactivity, the Tracker, about which we discussed in Chapter 1 of this tutorial. 
However,  Meteor has created a package that allows us to easily use [it](https://guide.meteor.com/react.html#using-createContainer).
This terminal command will add it to our project:
```
meteor add react-meteor-data
```

Let's assume you have a publication named 'donuts', that will of course return...Donuts.find().
In `/imports/ui/Home.jsx`, let's use:
```js
import { createContainer } from 'meteor/react-meteor-data';
import Donuts from '/imports/api/donuts/collection.js'

class Home extends React.Component {
    render() {
        const {loading, donuts} = this.props;
        
        if (loading) {
            return <div>Waiting for the method</div>
        }
        
        return (
            <div>
                {
                    donuts.map(donut => {
                        return <div key={donut._id}>{donut._id}</div>
                    })
                }
            </div>
        )
    }
}

export default createContainer((props) => {
    const handle = Meteor.subscribe('donuts');
    // the reactive data sources will get tracked here
    // any change will result into a re-render of "Home" component
    
    return {
        loading: !handle.ready(),
        donuts: Donuts.find().fetch()
    }
}, Home)
```

Now try adding a new donut from the Meteor shell. You will see it updated live.

This is it. It's that simple. You now have the knowledge that allows you to integrate React with Meteor. 
Now, you need to master it.

In the homework you'll most likely be dealing with forms, so here are some recommendations:
- [Material UI](http://www.material-ui.com/)
- [Formsy React](https://github.com/christianalfoni/formsy-react)
- [Formsy material UI](https://github.com/mbrookes/formsy-material-ui) 

There is a [package](https://github.com/meteor-utilities/react-list-container) that contains a lot of components and will do a lot of the work for you!

## Homework

#### 1. Log me in!
Create a route `/login` and a `Login.jsx` file. In it you will have a form. When the form is submitted, 
it will call `Meteor.loginWithPassword` and redirect him to "/" [Hint 1](/)

#### 2. Private Donuts!
Only allow logged in users to view the donuts list, which is private. [Hint]()

#### 3. Register
Create a registration. Use just email and password for now.

#### 4. Create a donut
As a logged in user, present a form and create a donut via the Meteor.call(). You can do it with a separate route, or
where you list the donuts. Whatever you choose, create another component. Always think in components.

## Over-achievers

#### 1. Filters
In the donuts list, add a checkbox, when you click it it will only show the donuts with price smaller than 200. Make sure the filter
is in another component, so that when something changes, your subscription changes as well. This means you may need a container on top of your container!

#### 2. Pagination (By Methods) (From scratch)
Implement a pagination strategy (You can use [React paginator](https://www.npmjs.com/package/react-paginator) where you display
all the donuts, 3 items per page. *Quick tip*: You may need to create another method that retrieves the count of all the donuts.

#### 3. Pagination (By Subscription) (From scratch)
This one is a bit tricky. But not impossible, if you want true reactive pagination (which in most cases you don't) but you want 
to know about it because you're an over-achiever. Check out [this](https://github.com/percolatestudio/publish-counts) package, which can be used for reactive counting.

