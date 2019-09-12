---
layout: post
comments: true
title: Don't use API Gateway mapping templates
date: 2016-10-31 20:20:23
categories: aws api-gateway
short_description: You shouldn't put any business logic in API Gateway for your own good.
---

I've been using AWS's API Gateway for around a year and can't agree more with Thoughtwork's assessment of
[Over Ambitious API Gateways](https://www.thoughtworks.com/radar/platforms/over-ambitious-api-gateways).

AWS's allows you to transform the incoming request and subsequent response with what it calls "mapping templates."
These mapping templates are written in [Velocity Template Language](https://velocity.apache.org/engine/1.7/vtl-reference.html).

Once you have a mapping template that details all the fields you expect in the request payload, you have to
update it **any time you want to add another field**. I seem to always forget this part and you end up
having expected fields duplicated in your code bases. API Gateway should be as dumb as possible and just
pass on the payload it receives. Let it stick to its concerns - rate limiting and checking API keys.

I've also encountered weird issues with escaping around backslashes in templates. It's unclear WTF is going on,
but I'll report back when the issue is solved.
