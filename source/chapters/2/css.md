---
title: 'Cascade Style Sheet (CSS)'
description: Styling your Meteor application
disqusPage: 'Chapter 2: CSS'
---

## Introduction


The style is an important aspect of an application. Anyone can write css, but there are some subtle things that makes the difference between a good written css and a bad one.

We choose not to use a framework, but we are going to guide you throughout the process of building the base css you'll need in most of your projects.

For this tutorial, we are going to use [SASS (SCSS)](http://sass-lang.com/), because it's an amazing tool that offers some features that plain css misses that could improve your productivity right away and help you modularize your code. For example: variables, mixins, import, functions and many other cool features.


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

Remeber that you can find a pre-installed meteor project with this folder structure, [here](https://github.com/cult-of-coders/meteor-tuts-boilerplate)


## Use of Flexbox
To structure our layout model, we are using CSS3 Flexbile Box because it solves some problems that the layout float model have:
- vertical and horizontal centering
- same height columns

You can learn more about it, [here](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

## Use of Rem for font size
We are using rem size for our fonts because it offers responsiveness, scalability, improved reading experience, and greater flexibility in defining components. To read more about it, click [here](https://www.sitepoint.com/understanding-and-using-rem-units-in-css/) 

## Writing clean css
It is extremely important to write clean css code so that we have some recommendations for you:
- When you create an application, prefix your classes with something specific and short. For example, we chose for our company "cc" (Cult of Coders). The reason behind it is that if we get an external library, we won't any have any conflicts
- Use 3 or less levels of css nesting. Most of the time you don't need more than one level because if you want to override those styles only on a specific elements, you don't have to recreate the whole nesting levels
- Using !important in your CSS usually means you're narcissistic & selfish or lazy. Respect the devs to come and don't use it!
- For each component (or page / section) create a separate file in which you'll store only the specific styles for that section. This will make the code much cleaner, offering better readability and very easy to modify later 
- Naming standard convention. Our current recommendation is [BEM](http://getbem.com/naming/)
- Be specific in naming your classes. The class names should represent your elements

```scss
/* BAD */
.box {
  background-color: green !important;
  .box2 {
    .cc-box3 {
      font-size: 5px;
      .cc-button-x {
        color: red; 
      }
    }
  }
} 
.cc-div {}
```

```scss
/* GOOD */
.cc-donut {
  &--bg-main{
    background-color: $color-main;
    color: $color-white;
  }
}
.cc-donuts-list {
  height: 100%;
  display: flex;
  
  &__title {
    @include font-size(16px);
  }
}
```

## Use of Variables
Variables are simply amazing because you can store some general rules that you want to apply in your stylesheet.

```scss
$font-roboto: 'Roboto', sans-serif;
$base-font-size: 62.5%;
$color-success: #4CAF50;
$shadow-dp0: 0px 0px 0px 0px rgba(black, 0.2), 0px 0px 0px 0px rgba(black, 0.14), 0px 0px 0px 0px rgba(black, 0.12);

/* How to use them? */
.cc-donuts-list {
  font-family: $font-roboto;
  color: $color-success;
  box-shadow: $shadow-dp0;
}
```

## Use of mixins
A mixin lets you make groups of CSS declarations that you want to reuse throughout your site
```scss
@mixin breakpoint ($value, $min-value: false) {
  @if $value == 'phone-small' {
    @media only screen and (max-width : 330px) { @content; }
  }

  @if $value == 'phone' {
    @media only screen and (max-width : 750px) { @content; }
  }

  @else if $value == 'tablet' {
    @media only screen and (min-width: 751px) and (max-width: 1199px) { @content }
  }

  @else if $value == 'desktop' {
    @media only screen and (min-width: 1601px) { @content }
  }

  @else {
    @media only screen and (max-width: $value) { @content; }
  }
}

/* How to use it? */
.cc-notification {
  @include breakpoint("tablet") {
    font-size: 50px;
  }
}

```
## Homework

#### 1. Donut colors
Style the section where we display the donuts. Add to each element from the section, a specific class. Each type of donut, will have a different color.

#### 2. Responsive donuts
The creation of a donut needs to be responsive so we can add it from each device we would like 
