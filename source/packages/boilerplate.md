---
title: Grapher Boilerplate
description: A meteor bootstrap project with Grapher and Grapher-Live and Demo Data
---

### Install

```
git clone https://github.com/cult-of-coders/grapher-boilerplate
cd grapher-boilerplate
meteor npm install
meteor
```

Or check it live:
http://grapher-live.cultofcoders.com

### Fixtures

Currently we have demo data and it is linked like this:

Users has many posts
Posts has many comments
Posts belong in a Group
Users belong in multiple Groups
Comment has one User

### Sample Queries

To fetch all posts with their comments and their authors in Grapher Live:

```js
{
    posts: {
        title: 1,
        comments: {
            text: 1,
            user: {
                profile: 1
            }
        }
    }
}
```

### Fetch only "Good" comments:

```
{
    posts: {
        comments: {
            $filters: {text: "Good"}
        }
    }
}
```

### Use sorting options

```
{
    posts: {
        $options: {
            $sort: {title: -1}
        },
        title: 1,
        owner: {
            profile: 1
        }
    }
}
```