---
title: 'Intro'
description: Getting Meteor to Enterprise Level
disqusPage: 'Chapter 3: Intro'
---

## Welcome

In this chapter we are going to talk about what does it take to write a solid back-end and how we solve common issues in a very elegant manner.

We are not going to work on how to structure the front-end part. We are going to address topics such as how do we structure the code, how we test it, how we lint it, and all sorts of elements
that make an app great behind the scenes and simply a joy to work with.

## Why?

We at [Cult of Coders](https://www.cultofcoders.com) believe in the future of Meteor, making things easy for development is the key of success. But even if you have a powerful tool like Meteor in your hands, you can still make many mistakes and spend time learning from them.
We made these mistakes so you don't have to. 

So why are we sharing them for free? When we could easily create a book and make some good bucks? Because money is not our objective. A thriving community is more valuable to us.

The main reason is that we love and support the community, the next is by engaging the community in this, we are able to gain more insights and improve on our knowledge.
 
That being said, we hope you enjoy this chapter and that it will open your mind.

May the code be with you.

## Beginning the adventure

We will begin with a story, a young developer joining a team of Meteor developers, and he receives his first task:

`
I want as a user to create a post.
After creation, I want to send an email to the admin so he can approve it
After approval, find the users that are interested in this post by their interests, and notify them.
`

From the client I would craft a form, and do something like, but since our focus is backend, so we won't get too much on the frontend
side of things.

```js
Meteor.call('posts.create', {
    title: 'I wanna learn Meteor!',
    tags: ['philosophy'],
})
```

In the server you start coding:
```js
import Posts from '/some/where/collection.js';

Meteor.methods({
    'posts.create'(post) {
        Posts.insert(post);
        
        Email.send({
            to: 'admin@app.com',
            from: 'notifications@app.com',
            subject: 'New Post',
            html: `
                You can access it here: ${Meteor.absoluteUrl(`/posts/${post._id}`)}
            `
        })
    },
    'posts.approve'(postId) {
        const post = Posts.findOne(postId);
        Posts.update(postId, {
            $set: {
                isApproved: true
            }
        });
        
        const users = Meteor.users.find({
            interests: {$in: post.tags}
        }).fetch();
        
        users.forEach(user => {
            Email.send({
                to: user.emails[0].address,
                from: 'notifications@app.com',
                subject: 'New Post With Your Interest',
                html: `
                    You can access it here: ${Meteor.absoluteUrl(`/posts/${post._id}`)}
                `
            })
        })
    },
})
```

This looks like a simple innocent way of doing things, the code is relatively clean, and it does the job.

But then the *Bigshot Code Reviewer* comes to you and says the following:

1. You need to validate the user when creating a post
2. What prevents the user of setting `isApproved: true` in the post object ? Never trust the client!
3. What if I want to change `notifications@app.com` from one place.
4. Post approval is not secured, you need an Admin role to do that
5. What if the `/posts/:_id` route changes I want to be able to change it from one place
6. What if I have multiple admins and want to notify them all ?
7. The tags need to be validated as well, we don't allow all possible tags

Because you are a good developer you listen to all of his requests and you start coding, then it will look something like this:

```js
import Posts from '/some/where/collection.js';

const FROM = Meteor.settings.emails.from; // You decide to put your "from" in settings, which is an ok approach.
const TAGS = ['psychology', 'philosophy'];

function generatePostUrl(_id) {
    return Meteor.absoluteUrl(`/posts/${_id}`);
}

Meteor.methods({
    'posts.create'(post) {
        if (!this.userId) {
            throw new Meteor.Error('not-allowed');
        }
        
        if (post.isApproved) {
            delete post['isApproved'];
        }
        
        if (post.tags) {
            let cleanedTags = [];
            post.tags.forEach(tag => {
                if (_.find(TAGS, tag)) {
                    cleanedTags.push(tag);
                }
            })
            
            post.tags = cleanedTags;
        }
        
        Posts.insert(post);
        
        const admins = Meteor.users.find({
            roles: {$in: ['ADMIN']}
        }).fetch();
        
        admins.forEach(admin => {
            Email.send({
                to: admin.emails[0].address,
                from: FROM,
                subject: 'New Post',
                html: `
                    You can access it here: ${generatePostUrl(post._id)}
                `
            })
        })
    },
    'posts.approve'(postId) {
        if (!Roles.userIsInRole(this.userId, 'ADMIN')) {
            throw new Meteor.Error('not-allowed');
        }
        
        const post = Posts.findOne(postId);
        
        if (!post) {
            throw new Meteor.Error('not-found');
        }
        
        Posts.update(postId, {
            $set: {
                isApproved: true
            }
        });
        
        const users = Meteor.users.find({
            interests: {$in: post.tags}
        }).fetch();
        
        users.forEach(user => {
            Email.send({
                to: user.emails[0].address,
                from: FROM,
                subject: 'New Post With Your Interest',
                html: `
                    You can access it here: ${generatePostUrl(post._id)}
                `
            })
        })
    },
})
```

## Implementation

By this time your code has grown a lot, and you send it for review being optimistic... but our **Bigshot Code Reviewer** comes with a new set of requests:

1. I need to be able to validate these tags at User level as well
2. I want to have Emails centralized somewhere so I can have a nice layout
3. When you find users you fetch a lot of data you don't need!
4. ... 

So what's the problem here? You want to do your job, and receiving so many comments makes you feel like you don't know jack. Then after you implement them you realize
that your code grows and grows and if another developer wants to read your methods he will have a hard time because it's simply too much. What if later on you want to send some Push Notifications,
or other stuff, a method can grow to 200 lines? Unacceptable.

```bash
Q: Is there any way in this lonesome world?
A: Yes. Use services.
```

Let's begin first with our Emailing service:
```js
// file: '/some/where/services/EmailService.js'
const FROM = Meteor.settings.emails.from;

function generatePostUrl(_id) {
    return Meteor.absoluteUrl(`/posts/${_id}`);
}

class EmailService {
    sendPostForApproval(userId, postId) {
        Email.send({
            to: this._getEmailForUser(userId),
            from: FROM,
            subject: 'New Post',
            html: `
                You can access it here: ${generatePostUrl(post._id)}
            `
        })
    }
    
    sendPostForInterest(userId, postId) {
        Email.send({
            to: this._getEmailForUser(userId),
            from: FROM,
            subject: 'New Post With Your Interest',
            html: `
                You can access it here: ${generatePostUrl(post._id)}
            `
        })
    }
    
    _getEmailForUser(userId) {
        const user = Meteor.users.findOne(userId, {
            fields: {emails: 1}
        })
        
        if (!user) {
            throw new Meteor.Error('not-found');
        }
        
        return user.emails[0].address;
    }
}

export default EmailService;
```
 
We already decouple mail sending in a nice manner, and we aren't fetching from database things we don't need. Nice!
 
```js
// file: '/some/where/services/PostService.js'
const TAGS = ['psychology', 'philosophy'];

class PostService {
    static createPost(userId, post) {
        // the services is not your security layer, so we shouldn't do the check here
        this._validateCreationPost(post)
        post.userId = userId;
        
        const postId = Posts.insert(post);
        this.notifyAdminForApprovalOfPost(post);
        
        return postId;
    }
    
    static approvePost(postId) {
        const post = this._getPost(postId);
        Posts.update(postId, {
            $set: {isApproved: true}
        });
        
        this.notifyUsersOfPosts(post);
    }
    
    static notifyAdminForApprovalOfPost(post) {
        const admins = Meteor.users.find({
            roles: {$in: ['ADMIN']}
        }, {
            fields: {_id: 1}
        }).fetch();
        
        admins.forEach(admin => {
            EmailService.sendPostForApproval(admin._id, post._id)
        });
    }
    
    static notifyUsersOfPosts(post) {
        const users = Meteor.users.find({
            interests: {$in: post.tags}
        }, {
            fields: {_id: 1}
        }).fetch();
        
        users.forEach(user => {
            EmailService.sendPostForInterest(user._id, post._id);
        })
    }
    
    static _getPost(postId) {
        const post = Posts.findOne(postId);
        
        if (!post) {
            throw new Meteor.Error('not-found');
        }
        
        return post;
    }
    
    static _validateCreationPost(post) {
        if (post.isApproved) {
            delete post['isApproved'];
        }
        
        if (post.tags) {
            let cleanedTags = [];
            post.tags.forEach(tag => {
                if (_.find(TAGS, tag)) {
                    cleanedTags.push(tag);
                }
            })
            
            post.tags = cleanedTags;
        }
    }
}

export default PostService;
```

Ok, nice, now as you can see the code can be read as poetry. By decoupling functions you understand what they do without seeing the code, this brings verbosity and makes the code a pleasure to work with.
So how will our methods look after these changes?

```js
Meteor.methods({
    'posts.create'(post) {
        if (!this.userId) {
            throw new Meteor.Error('not-allowed');
        }
        
        return PostService.createPost(post);
    },
    'posts.approve'(postId) {
        if (!Roles.userIsInRole(this.userId, 'ADMIN')) {
            throw new Meteor.Error('not-allowed');
        }
        
        return PostService.approvePost(postId);
    }
})
```

Wow! So clean and so sexy! Much more readable. But are we there yet? Is this how the code should look?
Not by far. 

But we did some good progress already, good job!
We understood that separation of logic, makes the code manageable, easy to understand, and easier to test.

So why do we say *not by far* ?

Well, the **Bigshot** will still see some problems:
1. If I want to write a test for PostService but not send emails how would I do that ?
2. Tags need to be centralized somewhere and apply same validation when User updates interests.
3. What if the user adds some extra fields to post object ? How do I stop that ?
4. What if that absolute path for seeing posts needs also to be decoupled and used somewhere ?
5. Emails still don't have a nice re-usable layout
6. PostService should be about Posts, not about notifying others, it has too much logic.
7. ...

Ok, how do we close this **Bigshot**'s mouth? We continue learning the principles behind quality code.

Let the adventure begin! But first, you must understand some basic stuff regarding programming in javascript:

This is the best resource I've found:
https://github.com/ryanmcdermott/clean-code-javascript 

It's based on Robert Martin's Clean Code book but tailored for our love, JavaScript.

Please don't treat it as just another link, you must absorb the teachings there, and make sure that until you master them,
you read them daily or at least weekly for 2 months.

Even I, read it from time to time, to refresh my memory.

`After all this time? Always.`