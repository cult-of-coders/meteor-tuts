---
title: Accounts
description: Handling users with Meteor
disqusPage: 'Chapter 1: Accounts'
---

Congratulations for making in this far with this Meteor tutorial! Now we're going to talk about a very important element in you application: users!
That's right! Without users, your application cannot become useful.
Meteor already has implemented some of the core functionality for dealing with users, and it offers simple yet very powerful APIs for helping you to implement user interaction in you application:

- User creation
- Login (via a password, or with Facebook, Google)
- Password recovery
- Change the user password
- Reset a user's password

In this section of the tutorial we focus more on teaching you how to handle passwords, but we'll also show you how easy it is to integrate the framework with other systems.
Run this in your command line interface, and then we can get started:
```
meteor add accounts-base accounts-password
```

In your server-side shell type this snippet of code:

```js
Accounts.createUser({
    email: 'donut@lover.com', 
    password: '12345'
})
```

Users get stored in a collection, which you can access via `Meteor.users`. 
It's the exactly like the collection that we have learned about until now.

Now use this snippet in your browser console:

```js
Meteor.loginWithPassword('donut@lover.com', '12345', function (err) {
    if (!err) {
        console.log('You see this because the authentication process was a success')
    } else {
        console.log(err);
    }
})
```

And, just like that, you're logged in!
Now go back to your browser console and let's see the user that is currently logged in and its ID:
```js
Meteor.user()
Meteor.userId()
```

Since `Meteor.user()` is a reactive data source, if you use it in a Tracker, you can benefit from its reactivity,
meaning you can access information about the users, logged in, or anonymous.

Another thing you may notice is how the `emails` key is followed by the *"verified"* check:
```js
[
    {
        address: 'donut@lover.com',
        verified: true|false
    }
]
```

This may seem to be redundant and complicated, but the Meteor developers decided to stick with this, maybe because they wanted to cater to the people
that want multiple email addresses linked to their account. To get the email of your user, you would have to use the following snippet:
```js
Meteor.user().emails[0].address
```

But you should not worry about this right now, because, by the end of this tutorial we'll learn how to easily implement this in any application, 
and simplify the steps you'll need to take in order to use it, so you won't have to repeat yourself whenever you will need a user's email.
You might think that '12345' is not a very secure password! Since you are correct, let's go ahead and change it:

```js
Accounts.changePassword('12345', 'My1337L333Tpassword%', function (err) {
    if (!err) {
        console.log('Changing the password was a success!')
    } else {
        console.log(err);
    }
})
```

Very nice! Now, since the password wasn't abiding to the basic guidelines that the user should respect when choosing his/her password, we made them change it.
 Now let's get the user to try out his/her new shiny password by logging them out, in order to get them to log back in with the new password:

```js
Meteor.logout(function (err) {
    if (!err) {
        console.log('The user was logged out successfully !')
    } else {
        console.log(err);
    }
});
```
Now, if you check their values, *Meteor.user()* and *Meteor.userId()* will be null, because there is no user that's currently logged in.

The [callbacks](https://en.wikipedia.org/wiki/Callback_(computer_programming)) we used previously in `loginWithPassword`, 
`changePassword` and `logout` are optional. So you can make the application work without them.

But wait! Your user's new password is so complex, he/she already forgot it(bad user...really bad user).
Since we're good people, let's reset their password:
```js
Accounts.forgotPassword({ email: 'donut@lover.com' })
```

Now if we check the output in the terminal, where we started Meteor, we should see a link that looks like this:
```
http://localhost:3000/#/reset-password/eNqDzCvx0F3OA6B0dzmx4i6kLs4-veJ36j3X2Rhxui7
```

The last part, the one after the last '\', is your token.
Now let's use that token:

```js
Accounts.resetPassword('eNqDzCvx0F3OA6B0dzmx4i6kLs4-veJ36j3X2Rhxui7', 'NewPassword123', function (err) {
    if (!err) {
        console.log('Password reset was a success!')
    } else {
        console.log(err);
    }
})
```
Now we can get back to the user and send our regards, together with the new password.

But what if you want the user to confirm the registration by e-mail ?

In your meteor shell type this:
```js
Accounts.createUser({
    email: 'user@withoutPassword.com'
})

//this will return the newly created _id

Accounts.sendEnrollmentEmail(_id);
```

Now go to where you started meteor and check out the email.
Get the token like we just did and set a new password with `Accounts.resetPassword()`.

**If you'd be interested to find out more:**

Meteor allows you to [customize](http://docs.meteor.com/api/passwords.html#Accounts-emailTemplates) the messages that you send to your users.

Did you like playing around with passwords ? Read more about them [here](http://docs.meteor.com/api/passwords.html).

Wouldn't you like to know how you can allow your users to log in with Facebook, Google, Twitter, etc ? 
Find out how to do it [here](https://guide.meteor.com/accounts.html#supported-login-services).
