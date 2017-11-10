---
title: 'Cascade Style Sheet (CSS)'
description: Styling your Meteor application
disqusPage: 'Chapter 2: CSS'
---

## Introduction


The style is an important aspect of an application. Anyone can write css, but there are some subtle things that makes the difference between a good written css and a bad one.

We choose not to use a framework, but we are going to guide you throughout the process of building the base css you'll need in most of your projects.

For this tutorial, we are going to use SASS (SCSS), because it's an amazing tool that offers some features that plain css misses that could improve your productivity right away and help you modularize your code. For example: variables, mixins, import, functions and many other cool features. To read more about it, [click here](http://sass-lang.com/)


## Setting Up

```bash
meteor add fourseven:scss
meteor remove standard-minifier-css // meteor installs this by default to optimize the css
meteor add juliancwirko:postcss // for a better optimization of css
meteor npm install --save-dev autoprefixer css-mqpacker postcss-csso
```

We are using [PostCSS](http://postcss.org/) because it makes our life easier by using some of it's plugins:
- [Autoprefixer](https://github.com/postcss/autoprefixer) taking out our responsability to prefix our css rules for each browser.
- [CSS MQPacker](https://github.com/hail2u/node-css-mqpacker) pack some CSS media query rules into one using PostCSS into one and adding them at the end of css file
- [CSSO](https://github.com/css/csso) is a CSS minifier that compresses and cleans our code

To configure your newly installed plugins, you need to add this in package json:

``` json
"postcss": {
    "plugins": {
      "autoprefixer": {
        "browsers": [
          "last 2 versions",
          "> 2%"
        ]
      },
      "css-mqpacker": {},
      "postcss-csso": {}
    }
  }

```

## Styles Folder Structure
This is the basic folder structure we are using and recommend

<pre>
├── client 
│   ├── styles
│   │   ├── cc-app.scss // Main file where we import all the other scss files
│   │   └── _helpers // in this folder we add all files for variables, functions, mixins 
|   │   │    └── _functions.scss // functions that we use in mixins 
|   │   │    └── _helpers.scss // general rules all over the website
|   │   │    └── _mixins.scss // a group of declarations that you want to re-use
|   │   │    └── _variables.scss
│   │   └── base 
|   │   │    └── _normalize.scss // css reset for the browser 
|   │   │    └── _font.scss // rules for importing external fonts 
|   │   │    └── _general.scss // general rules for elements (html, body, links, a, etc)
│   │   └── elements
|   │   │    └── grid.scss
│   │   └── form  
|   │   │    └── _button.scss
|   │   │    └── _checkbox.scss 
|   │   │    └── _radio.scss
|   │   │    └── _text.scss
|   │   │    └── _textarea.scss
|   │   │    └── _select.scss
|   │   │    └── _form-helpers.scss
│   │   └── layout 
|   │   │    └── header
|   │   │    └── footer
|   │   │    └── main
│   │   └── pages // a folder in which we add the css files for each page of application
│   │   └── plugins // // in this folder we add the files from our external components that we use in the app
│   ├── ...
├── imports
│   ├── ...
└── server 
    └── ...
</pre>

You can find a pre-installed meteor project with this folder structure, [here](wwww.github.com)


## Use of Flexbox
To structure our layout model, we are using CSS3 Flexbile Box because it solves some problems that the layout float model have:
- vertical and horizontal centering
- same height columns

You can learn more about it, [here](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)


## Writing clean css


Using !important in your CSS usually means you're narcissistic & selfish or lazy. Respect the devs to come...
