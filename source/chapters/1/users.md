---
title: Accounts
description: How Meteor handles users
disqusPage: 'Chapter 1: Accounts'
---

In this part of the tutorial we will discuss about the friendly and simple APIs for:

- Creating an user
- Login (with Password, Facebook, Google and others)
- Forgot Password
- Change Password
- Reset Password

In this part we'll focus more on registering and logging in the regular way, but we will also guide you on how you can integrate it with other authentication mechanism.

```
meteor add accounts-base accounts-password
```
Create a server side method where we register an user

```js
// file: /imports/api/users/methods.js
'user.register' (data) {
    Accounts.createUser({
        email: data.email,
        password: data.password
    });
}

// client side, in the browser try this:
const data = {email: 'test@test.test', password: '12345'}
Meteor.call('user.register', email, password, (err, result) => {
    console.log(err, result) // in case you try it twice, it will throw an exception that email already exists
});

```

Users are stored in a collection. You can access this collection via `Meteor.users`. 
It's the same kind of collection that we learned about in the past chapters.

Now go to your browser's console:

```js
Meteor.loginWithPassword('test@test.test', '12345', function (err) {
    if (!err) {
        console.log('I was called because authentication was a success')
    } else {
        console.log(err);
    }
})
```
You should be logged in now!

```js
// in browser console:
Meteor.user() // will return the current logged in user
Meteor.userId() // will return the _id of the current logged in user
```

`Meteor.user()` is a reactive data source, so if you use it in a Tracker, then you will benefit from it's reactivity.

Another thing you may notice is how `emails` key is structured:
```js
[
    {
        address: 'meteor@cultofcoders.com',
        verified: true|false
    }
]
```

This may seem a bit complicated, but they decided to stick with this, maybe because they wanted to easily satisfy  the people
who want multiple email addresses on their account. To get the email of your user, you would have to do:
```js
Meteor.user().emails[0].address
```

But don't worry about this now, when we'll learn how to make this easy, so you won't have to type this everywhere you need an user's email.

You think '12345' is not a very secure password, and you are correct, let's change it:

```js
// browser's console
Accounts.changePassword('12345', 'My1337L333Tpasswurt%', function (err) {
    if (!err) {
        console.log('Change password was a success!')
    } else {
        console.log(err);
    }
})
```

Very nice, now let's try a logout:

```js
//browser's console
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

But wait, your new password is so complex, you already forgot it.

```js
Accounts.forgotPassword({ email: 'donut@lover.com' })
```

Now check your terminal, where you started Meteor, you should see the email you got, you should have a link like:
```
http://localhost:3000/#/reset-password/eNqDzCvx0F3OA6B0dzmx4i6kLs4-veJ36j3X2Rhxui7
```

The last part is your token.

```js
Accounts.resetPassword('eNqDzCvx0F3OA6B0dzmx4i6kLs4-veJ36j3X2Rhxui7', 'NewPassword123', function (err) {
    if (!err) {
        console.log('Password reset was a success!')
    } else {
        console.log(err);
    }
})
```

If you want to read more about this package, you can find documentation for it [here](https://docs.meteor.com/api/accounts.html)

# Important
In order to see this in real action, you can access on [github](www.github.com) and clone the project.