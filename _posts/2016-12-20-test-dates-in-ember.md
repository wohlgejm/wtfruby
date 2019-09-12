---
layout: post
comments: true
title: Testing `Date` in ember
date: 2016-12-20 20:20:30
categories: ember javascript
short_description: How to test `new Date()` in ember
---

I recently added a bunch of timestamps to an ember application and wanted to assert that they were being saved
in an acceptance test.  To do this, I added an ember-cli plugin,
[ember-sinon](https://github.com/csantero/ember-sinon), that adds the
[sinon](http://sinonjs.org/docs/) library to the app.

Sinon has fake timers which can control the use of `new Date()` and `Date.now()`, which is exactly what I want
to test since I'm using `new Date()` to set the attribute on the ember model.

I ended up with an acceptance test that looked like this:

```javascript
import { module, test } from 'qunit';
import sinon from 'sinon';

let now = new Date();

module('Acceptance: ToDos', {
  beforeEach: function() {
    this.clock = sinon.useFakeTimers(now.getTime(), 'Date');
  }
  afterEach: function() {
    this.clock.restore();
  }
}

test('sets updated_at', function(assert) {
  assert.expect(1);
  visit('/');

  andThen(function() {
    server.put('/', function(db, request) {
      let response = JSON.parse(reqest.reqestBody).response;
      assert.equal(response.updated_at, now.toISOString(), 'updated_at is set correctly');
    });
  });
  click('.submit-button');
}
```
