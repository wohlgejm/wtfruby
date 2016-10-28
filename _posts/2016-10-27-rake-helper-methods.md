---
layout: post
comments: true
title: Don't use helper methods in rake tasks
date: 2016-10-27 19:15:23
categories: ruby rails
short_description: Defining a helper method in a rake task can have unintended side effects.
image_preview: https://media.giphy.com/media/9YuLKEMFcxs88/giphy.gif
---

I recently made the mistake of writing a rake task and, when I had duplicate logic across two of the tasks,
decided to pull it out into a helper method inside the namespace block. I also gave it a general name,
`update_foo`. This first task ran and worked fine.

```ruby
namespace :foo do
  desc 'Updates foo'
  task :update, [:foo] => [:environment] do |_, args|
    foo = Foo.find_by_bar(args[:foo])
    update_foo(foo)
  end

  desc 'Updates all foos'
  task update: [:environment] do
    Foo.all.each { |foo| update_foo(foo) }
  end

  def update_foo(foo)
    do_something
  end
end
```

Then, I had another set of tasks that also shared similar logic and had an `update_foo` helper method.
I ran the second task and was baffled when it didn't work.

After some debugging, it became clear that the helper method of the same name, *in the other rake task* was
being called. I should have realized this since this is all happening inside a block. `update_foo` was being
defined globally and was overwritten by definition inside the other rake task. Instead of doing this, I
should've extracted this common logic into its own class.
