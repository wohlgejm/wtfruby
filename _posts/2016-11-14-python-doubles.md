---
layout: post
comments: true
title: Uber's Python test doubles are awesome
date: 2016-11-14 20:20:30
categories: python testing
short_description: Use doubles instead of mock
image_preview: https://media4.giphy.com/media/N1RuMazLHPqDK/200.gif#86
---

One reason Ruby makes me happy is `rspec`. It reads like documentation and it's really easy to mock and stub.
Python has the `mock` library, but this always felt clunky to me.

A common test case is interacting with an external API.
Here's an example of mocking a get request in Python, using `py.test`.

```python
from collections import namedtuple
import json

import mock

from my_api_service import ApiService


@mock.patch('requests.get')
def test_api_service(mock_get):
    fake = namedtuple('Request', 'status_code content'])
    mock_get.return_value = fake(200, json.dumps({'foo': 'bar'}))
    assert ApiService().data == {'foo': 'bar'}
```

Here's the same spec in rspec:

```ruby
RSpec.describe ApiService do
  describe '.data' do
    let(:fake) { double(:fake, status_code: 200, content: { foo: bar }.to_json }
    subject { described_class }

    it 'returns foo bar' do
      allow(Net::HTTP).to receive(:get).and_return(fake)
      expect(subject.data).to eq({ foo: 'bar' })
    end
  end
end
```

I just came across Uber's doubles library, which appears to be about two years old and I wish I discovered this
earlier.

Here's that same spec, but using doubles:

```python
from doubles import allow, InstanceDouble
import requests

from my_api_service import ApiService


def test_api_service():
    fake = InstanceDouble('requests.Response')
    fake.status_code = 200
    allow(fake).content.and_return(json.dumps({'foo': 'bar'})
    allow(requests).get.and_return(fake)
    assert ApiService().data == {'foo': 'bar'}

```
It might look like a lot of boiler plate to allow methods on the `InstanceDouble`, but this is much safer
than the other spec where I just used a namedtuple. From the [docs](http://doubles.readthedocs.io/en/latest/usage.html#pure-doubles):

> The double that’s created will verify itself against an instance of that class.
> The example above will fail with a VerifyingDoubleError exception, assuming foo is not a real instance method.

`doubles` makes it a whole lot easier, in my opinion to, to write tests in Python.
