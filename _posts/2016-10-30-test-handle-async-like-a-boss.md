---
layout: post
title: Testing `handle_asynchrously` like a boss
date: 2016-10-30 15:20:30
categories: ruby rails
short_description: How to test DelayedJob's `handle_asynchrously`. Like a boss.
image_preview: https://media2.giphy.com/media/wjNzcOiXJ1xPW/200.gif#73
---

If you're using `Delayed::Job` you can conveniently define background tasks like:

```ruby
class Foo
  def takes_long
    sleep 10
  end
  handle_asynchrously :takes_long
end
```

This saves you the trouble of setting up a worker class, defining `perform` on that class, then explicitly
calling `takes_long_without_delay`.

To test this, you can mock `Delayed::PerformableMethod`, which is a `Struct` that is initialized in
`Delayed::MessageSending.handle_asyncrously`.

```ruby
RSpec.describe 'Foo' do
  describe '#takes_long' do
    let(:job) { double(:job, perform: nil) }
    subject { Foo.new }

    it 'enqeues the function' do
      expect(Delayed::PerformableMethod).to receive(:new) do |obj, method, _args|
        expect(method).to eq(:takes_long_without_delay)
        expect(obj).to be(obj)
      end.and_return(job)
      subject.takes_long
    end
  end
end
```

You need to stub a job that implements `perform` and return it so DelayedJob can enqueue that stub
and not raise an error. This doesn't test the functionality of `takes_long`, so you should use
another [technique](http://code.dblock.org/2015/11/02/how-to-test-delayed-jobs.html) for that.
