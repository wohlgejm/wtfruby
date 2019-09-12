---
layout: post
comments: true
title: Flask CORS on Chrome
date: 2017-06-15 20:20:30
categories: python flask
short_description: Getting CORS to work on Chrome
---

The creator of Flask, Armin Ronacher, has a snippet [here](http://flask.pocoo.org/snippets/56/)
to handle cross origin requests in Flask. This works great if you are not using Chrome and testing from `localhost`.
To get this decorator to work on Chrome from `localhost`, add the following header.
```python
h['Access-Control-Allow-Headers']  = (
    'Cache-Control, Pragma, Origin, Authorization,'
    'Content-Type, X-Requested-With'
)
```
