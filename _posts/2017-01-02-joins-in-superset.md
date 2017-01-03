---
layout: post
comments: true
title: Joins in Airbnb's superset
date: 2017-01-02 20:20:30
categories: python
short_description: How to join in Airbnb's superset
image_preview: /images/charlie6.gif
---

Airbnb has an awesome open source data viz tool called [superset](http://airbnb.io/superset/index.html).

I've been working on getting this deployed to heroku. Getting it deployed was easy by cloning this repo [here](https://github.com/dugjason/superset-on-heroku).

After I deployed it an and started experimenting, a read the [FAQ](http://airbnb.io/superset/faq.html)
which had a **bombshell** that joins aren't possible. Without joins, since my datasource is postgres,
the utility of the tool is pretty low.

I dug through some issues, searching for *joins* and came across this [issue](https://github.com/airbnb/superset/issues/875). By adding a column and a subquery, you can achieve a join.

Here's a screenshot of adding the column:

![superset](/images/superset-joins.png)
