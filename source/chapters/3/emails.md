---
title: 'Emails'
description: How to work with emails
disqusPage: 'Chapter 3: Emails'
---

Most of the apps send out emails, but what is the best way to do it ?

Well, I don't know about the best, but let's explore this solution.

We want emails to:
- Be able to be found easily in a centralized place. It's very important they are not spread all over the app so we have control over them.
- Be able to have a nice layout for everyone of them
- Be easy to develop and easy to maintain

So, if we want to keep them centralized, it's clear that we must use [Events](/chapters/3/events.html). Therefore,
whenever we want to send an email we need to dispatch an event.

So, how do we render our emails ? React baby!

So we want to create a service that sends an email based on a React template. Let's see how it would look like:

```js
// file: /imports/api/emails/send.js
import {Email} from 'meteor/email';
import {Meteor} from 'meteor/meteor';
import React from 'react';
import ReactDOM from 'react-dom/server';

const debug = Meteor.isDevelopment;
// const debug = false;

/**
 * React Email Sender
 */
export default function (mailConfig, Component, props) {
    const element = React.createElement(Component, props);
    
    let options = _.extend({}, {
        from: 'notifications@app.com', // or get it from Meteor.settings
        html: ReactDOM.renderToString(element)
    }, mailConfig);

    if (debug) {
        console.log(options);
    } else {
        // so it won't lag your methods
        Meteor.defer(() => {
            Email.send(options);
        })
    }
}
```

Cool I think it's self descriptive what this function does, here is how it would look in a real-life scenario:

```bash
// file: /imports/api/emails/templates/NewItemCreated.jsx
import React from 'react';
import PropTypes from 'prop-types';

const NewItem = ({item}) {
    return (
        <div>This is your new {item.title}</div>
    )
}

NewItem.propTypes = {
    item: PropTypes.object,
};

export default NewItem;
```

And to actually send it:

```js
// file: /imports/api/emails/listeners.js
import send from './send';
import {Items} from '/imports/db';
import {Emitter, Events} from '/imports/api/events';

import ItemNewEmail from '/imports/api/emails/templates/NewItemCreated.jsx'

Emitter.on(Events.ITEM_NEW, function ({itemId}) {
    const item = Items.findOne(itemId);
    
    send({
        to: 'rick.astley@never-gonna-give-you-up.com',
        subject: 'Never gonna let you down!'
    }, ItemNewEmail, {item})
});
```

Now if you want to create a layout enveloping your template it's quite easy, you would do something like:

```bash
// ...
const Layout = function({children, header}) {
    return (
        <table>
            <tbody>
                <tr>
                    <td>{headerText}</td>
                </tr>
                <tr>
                    <td>{children}</td>
                </tr>
            </tbody>
        </table>
    )
}
```

And because we want to keep the send email as flexible as possible we will use the layout inside our templates:

```bash
// ...
const ItemNewEmail = ({item}) {
    return (
        <Layout headerText={'You received a new item'}>
            This is your new item: <strong>{item.title}</strong>
        </Layout>
    );
}
```

Both `Layout` and `ItemNewEmail` can change in anyway you like, depending on your context. 
Of course you can have multiple layouts, or no layout, it's up to the **email template to decide, not the send function**, don't forget this.

Isn't this easy and clean ?