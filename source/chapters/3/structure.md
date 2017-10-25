---
title: 'Overall Structure'
description: Let's bring it to a degree of order
disqusPage: 'Chapter 3: Structure'
---

So, we discussed about many concepts and we described where to put them. But the thing is,
it doesn't really matter that much, what we describe here are a set of practices, there's no such thing
as the best way, or the way. Whatever you choose, be consistent and try to document it.

To give you an overall image, it will look something like this:

## Standalone

```bash
server
    # This is where you import everything you need on server to start
    # Most likely it will be just: import '/imports/api/server'
    main.js
client
    # This is where you import everything you need on client to start
    # Most likely it will be just: import '/imports/client/init'
    main.js
imports
    # This is where our logic belongs
    api
        # We import all the "/server" folders from our modules
        server.js
        # For emitter and events (you can also make it a folder with index.js if you need decoupling)
        events.js
        # For securing your app (you can also make it a folder with index.js if you need decoupling)
        security.js
        emails
            send.js
            listeners.js
            templates
                {Module}NewEmail.jsx
        # We couple code based on different modules
        {module}
            # Here we store things that need to run only on the server
            server
                exposures
                    get{Module}.expose.js
                    index.js # Here we import them all
                listeners.js
                methods.js
                publications.js
                # Here we import all of them (including ./exposures)
                index.js
            # Here we decouple our logic
            services
                {Module}Service.js
            testing
                server
                    {Module}Service.test.js
            
    client # Outside this scope (This will be treated in another chapter)
    # This is where our persistence layer is. Including external APIs
    db
        # Here we import and export all collections for easy usage
        index.js
        {module}
            # Nice grapher queries
            queries
                get{Module}.js
            aggregations
                getGroupedElementsByEmail.js
                testing
                    server
                        getGroupedElementsByEmail.test.js
            collection.js
            schema.js
            hooks.js
            links.js
    # For dummy data
    fixtures
        createUsers.js
        config.js
        index.js
```

## Microservice

```bash
core
    # For emitter and events (you can also make it a folder with index.js if you need decoupling)
    events.js
    # For securing your app (you can also make it a folder with index.js if you need decoupling)
    security.js
    emails
        send.js
        listeners.js
        templates
            {Module}NewEmail.jsx
    services
        {module}
            {Module}Service.js
            testing
                server
                    {Module}Service.test.js
    # This is where our persistence layer is. Including external APIs
    db
        # Here we import and export all collections for easy usage
        index.js 
        {module}
            # Nice grapher queries
            queries
                get{Module}.js
            aggregations
                getGroupedElementsByEmail.js
                testing
                    server
                        getGroupedElementsByEmail.test.js
            collection.js
            schema.js
            links.js
    # For dummy data
    fixtures
        createUsers.js
        config.js
        index.js
{microservice}
    server
        main.js
    client
        main.js
    imports
        api
            # We import all the "/server" folders from our modules
            server.js
            # We have the bridge between Meteor and our core
            {module}
                # Here we store things that need to run only on the server
                server
                    exposures
                       get{Module}.expose.js
                       index.js # Which imports all
                    listeners.js
                    methods.js
                    publications.js
                    # Here we import all the files (including ./exposures)
                    index.js
        client # Outside this scope (This will be treated in another chapter)
```

