---
layout: post
comments: true
title: Private keys in dokku
date: 2016-12-12 20:20:30
categories: dokku
short_description: How to set a private key environment variable in dokku
image_preview: /images/macboat.gif
---

I've been playing around with [dokku](https://github.com/dokku/dokku) which is a pretty awesome heroku replacement
if you only need a single instance. It's really easy to set up a rails app on a Digital Ocean micro instance
for $5/month. I went with this over heroku's free tier because I needed way more database capacity than 10k rows,
which heroku caps you at.

If you need to set a private key as an environment variable, you'll run into issues with the way the returns are
escaped. I hopped into the dokku IRC channel and their recommendation was to base64 encode the key and decode in
the app.

Simple enough.

Here's how do it from the command line, getting rid if the extra lines when encoded:

```bash
dokku config:set appname PRIVATE_KEY_BASE64=$(cat privatekey | base64 | sed ':a;N;$!ba;s/\n//g')
```
