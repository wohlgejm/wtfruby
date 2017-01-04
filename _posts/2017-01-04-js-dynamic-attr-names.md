---
layout: post
comments: true
title: Javascript dynamic key names in ES6
date: 2017-01-04 20:20:30
categories: javascript
short_description: Setting a dynamic key name on a javascript object
image_preview: /images/dee1.gif
---

Today I was looking into creating a javascript object with a dynamic key name.
I came to these [docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer)
and thought there was hope, but alas, new object literal features are only available in ES6.
I was fooled by using the Chrome console to think it was an ES5 feature. I always forget that Chrome supports
ES6.

In ES5 you have to do this.

```javascript
var key = 'dynamicKey';
var o = {};
o[key] = 'value';
```

But in ES6, when creating an object literal, you can do this:

```javascript
var key = 'dynamicKey';
var o = { [key]: 'value' };
```

A little funky, but the world will be a better place when ES5 doesn't need to be supported.
