---
title: Accounts
description: How Meteor handles users
---

Oh, hello again, so Meteor has a very opinionated way on doing this stuff, and it offers very friendly and dead simple APIs for:

- Creating an user
- Login (Password, Facebook, Google, ...)
- Forgot Password
- Change Password
- Reset Password

In this episode we'll focus more on using it with passwords, but we'll also show you how easy it is to integrate it with other systems.

```
meteor add accounts-base accounts-password
```

Open your server-side shell and type:

```
Accounts.createUser({
    email: 'donut@lover.com', 
    password: '12345'
})
```

Users are stored in a collection. You can access this collection via `Meteor.users`. 
It's the same kind of collection that we learned about in the past chapters.

Now go to your browser's console:

```
Meteor.loginWithPassword('donut@lover.com', '12345', function (err) {
    if (!err) {
        console.log('I was called because authentication was a success')
    } else {
        console.log(err);
    }
})
```

Bam! You're logged in.

```
// in browser console:
Meteor.user() // will return the current logged in user
Meteor.userId() // will return the _id of the current logged in user
```

`Meteor.user()` is a reactive data source, so if you use it in a Tracker, then you will benefit from it's reactivity,
meaning you can show some stuff for logged in users, and hide for anonymous ones.

Another thing you may notice is how `emails` key is structured:
```
[
    {
        address: 'donut@lover.com',
        verified: true|false
    }
]
```

This may seem a bit complicated, but they decided to stick with this, maybe because they wanted to satisfy easily the people
who want multiple email addresses on their account. To get the email of your user, you would have to do:
```
Meteor.user().emails[0].address
```

But don't worry about this now, when we'll learn how to make this easy, so you won't have to type this everywhere you need an user's email.

You think '12345' is not a very secure password, and you are correct, let's change it:

```
Accounts.changePassword('12345', 'My1337L333Tpasswurt%', function (err) {
    if (!err) {
        console.log('Change password was a success!')
    } else {
        console.log(err);
    }
})
```

Very nice, now let's try a logout:

```
Meteor.logout(function (err) {
    if (!err) {
        console.log('Logout was a success!')
    } else {
        console.log(err);
    }
});
// now Meteor.user() and Meteor.userId() will be null
```

Next time you login, you'll login with your new password.

Btw, the callbacks we used in `loginWithPassword`, `changePassword` and `logout` are optional, you can simply not use it.

But wait, your new password is so complex, you already forgot it.

```
Accounts.forgotPassword({ email: 'donut@lover.com' })
```

Now check your terminal, where you started Meteor, you should see the email you got, you should have a link like:
```
http://localhost:3000/#/reset-password/eNqDzCvx0F3OA6B0dzmx4i6kLs4-veJ36j3X2Rhxui7
```

The last part is your token.

```
Accounts.resetPassword('eNqDzCvx0F3OA6B0dzmx4i6kLs4-veJ36j3X2Rhxui7', 'NewPassword123', function (err) {
    if (!err) {
        console.log('Password reset was a success!')
    } else {
        console.log(err);
    }
})
```

Bam!
What if you want registration, but with mail confirmation ?

Go to your meteor shell:
```
Accounts.createUser({
    email: 'user@withoutPassword.com'
})

// it returns the newly created _id

Accounts.sendEnrollmentEmail(_id);
```

Now go to where you started meteor and check the email out.
Get the token and set a new password using `Accounts.resetPassword()`

Emails are customizable:
http://docs.meteor.com/api/passwords.html#Accounts-emailTemplates

Read more about other cool stuff here:
http://docs.meteor.com/api/passwords.html

That's a short intro into the account system. Isn't it super-duper easy ?

Login with Facebook, Google, Twitter, etc:
https://guide.meteor.com/accounts.html#supported-login-services