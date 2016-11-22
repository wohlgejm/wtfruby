---
layout: post
comments: true
title: Partial indices can be a bad idea
date: 2016-11-22 20:20:30
categories: postgres rails
short_description: Why you shouldn't use partial indices
image_preview: https://media.giphy.com/media/XeAiPVAmHl8Wc/giphy.gif
---

You might be tempted to use a partial index in Postgresql for a common scope you have.

An example of a migration looks like this is:

```ruby
class AddPartialIndexToFoo < ActiveRecord::Migration
  add_index(:foo, :active, where: 'active IS true')
end
```

The scope could look like:

```ruby
class Foo ActiveRecord::Base
  scope :active, -> { where(active: true) }
end
```

The partial index will not evolve with any code changes.
A feature request might change the `active` scope to include another attribute.
Once that scope is changed, the index is worthless and it's hard to remember
that it must be changed.

This bit us hard recently and it was hard to pin down because the query timed out only *some times*
and on a variety of pages where the scope was used.

If you have a default scope, it would probably make matters even worse.

Because partial indexes can't keep up with scopes, I think they are *generally* a bad idea, like default scopes.
