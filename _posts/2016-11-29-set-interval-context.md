---
layout: post
comments: true
title: Watch out for `setInterval`'s context
date: 2016-11-29 20:20:30
categories: javascript
short_description: Be wary of `setInterval`'s context.
image_preview: http://i39.tinypic.com/1znyi6p.gif
---

I had a WTF moment pairing today on a bug in some javascript. There was an interval function
being called that should've been disabled. The code looked something like this:

```javascript
foo = {};
foo.timer = function() { if(!this.disabled) { console.log(this); } }

setInterval(foo.timer, 100);
```

`this` was not what we thought it was, `foo`, but was `window`.

Apparently, this is well [documented](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval),
but I had no idea.

> Code executed by setInterval() runs in a separate execution context than the function from which it was called.
> As a consequence, the this keyword for the called function is set to the window (or global) object,
> it is not the same as the this value for the function that called setTimeout

You can get around this by calling `foo` directly, or binding the function to `foo`.

```javascript
foo = {};
foo.timer = function() { if(!this.disabled) { console.log(this); } }

i = setInterval(foo.timer.bind(foo), 100);
```
