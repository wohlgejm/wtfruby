---
layout: post
comments: true
title: Bust github badges cache
date: 2016-11-10 21:20:30
categories: ruby rails
short_description: How to bust a cached badge on github
image_preview: https://img.buzzfeed.com/buzzfeed-static/static/2014-11/26/20/enhanced/webdr10/anigif_enhanced-25333-1417052342-10.gif
---
I wanted to see the updated coverage badge for my Jira cli, [Accountable](https://github.com/wohlgejm/accountable).

Github caches these badges according to this [issue](https://github.com/lemurheavy/coveralls-public/issues/116).

Credit to courtney-miles whose answer worked for me: `curl -X PURGE "https://camo.githubusercontent.com/..."`.
