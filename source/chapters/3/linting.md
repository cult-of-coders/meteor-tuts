---
title: 'Linting'
description: Keeping your code standards high
disqusPage: 'Chapter 3: Linting'
---

## Why?

Because it takes you 2 minutes to set up, and the code quality respects the standards. Also you make it easier for code-reviewers, to spend time
on thinking about what you did, rather than your coding standards mistakes.

The thing is we need to automate this process, so we need to run the linting *before we commit*.
And not allow a commits with bad code, this will force you to write beautiful code.

## Install

Also, try integrating it with your IDE, for WebStorm look here: https://www.jetbrains.com/help/webstorm/eslint.html 

```js
meteor npm i --save-dev babel-eslint eslint eslint-plugin-import eslint-plugin-meteor eslint-plugin-react eslint-import-resolver-meteor lint-staged pre-commit
```

## Config 

This is an opionated configuration.

Create `.eslintrc.js` file inside your project root:
```
module.exports = {
    "parser": "babel-eslint",
    "parserOptions": {
        "allowImportExportEverywhere": true,
        "ecmaFeatures": {
            "jsx": true
        }
    },
    "env": {
        "es6":     true,
        "browser": true,
        "node":    true,
    },
    "plugins": [
        "meteor",
        "react"
    ],
    "extends": ["eslint:recommended", "plugin:meteor/recommended", "plugin:react/recommended"],
    "settings": {
        "import/resolver": "meteor"
    },
    "rules": {
        "react/jsx-filename-extension": [1, {
            "extensions": [".jsx"]
        }],
        "react/jsx-no-bind": [2, {
            "ignoreRefs": false,
            "allowArrowFunctions": false,
            "allowFunctions": false,
            "allowBind": false
        }],
        "max-len": [0, {code: 100}],
        "import/no-absolute-path": [0],
        "meteor/audit-argument-checks": [0],
        "indent": ["error", 4],
        "switch-colon-spacing": [0],
        "no-invalid-this": [0],
        "new-cap": [1],
        "no-trailing-spaces": [2, {
            skipBlankLines: true
        }],
    },
    "overrides": {
        files: "*.js,*.jsx",
    }
};
```

In your `package.json` file add the following:

```
{
  ...
  "scripts": {
    "start": "meteor run --settings settings.json",
    "test": "meteor test --driver-package practicalmeteor:mocha --settings settings.json",
    "lint": "eslint . --ext .jsx --fix"
  },
  "lint-staged": {
    "*.js": "eslint --fix",
    "*.jsx": "eslint --fix"
  },
  "pre-commit": "lint-staged"
}
```

Nice, now if you want to commit, it will try to lint your code first. And if any errors happen, it will not allow you to do so.

If by any chance you are in a rush and you have a production bug and the whole world is burning! Try running:
```
git commit -m "xxx" -n
```

Using `-n` will bypass the linting and it will allow you to commit directly.