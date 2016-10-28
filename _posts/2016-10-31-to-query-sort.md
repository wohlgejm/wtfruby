---
layout: post
title: to_query does something, maybe, weird?
date: 2016-10-27 00:15:23
categories: ruby rails
short_description: Beware when testing ActiveSupport's `to_query`
image_preview: https://media2.giphy.com/media/j7F8n3f5IYays/200.gif#66
---

`ActiveSupport` provides a helpful `to_query` method that can be called on a `Hash` to build
url query parameters in a readable fashion. If you use it and then test it, be aware that
`sort` is called before the string concatenation begins.

Another implementation detail to be aware of is that `CGI.escape` on the key, value pairs.
This is different than `URI.escape`. `URI.escape` has been deprecated, but it's something
to be aware of if you're writing a spec since, for instance, this can happen:

```ruby
URI.escape('+')
> "+"
CGI.escape('+')
> "%2B"
```
