---
layout: post
comments: true
title: Fake `open` with Python doubles
date: 2016-11-20 15:20:30
categories: python testing
short_description: How to test `open` with the doubles library
image_preview: https://media4.giphy.com/media/4BtwJzwVJBCP6/200.gif#38
---

I've been strugging a bit testing an `open` call with the `doubles` library. As is the case many times, most
of my struggle has revolved aroudn writing python2 and 3 compatible code/tests.
I'm trying to assert the built in `open` function is called with the right file name argument.

The first thing you need to do is disable builtin verification in the `doubles` library, using a context manager
they [provide](https://doubles.readthedocs.io/en/latest/faq.html). Next, you need to handle the change in modules
where open is provided from 2 to 3, `__builtin__` to `builtins`. More documentation available
[here](https://wiki.python.org/moin/PortingToPy3k/BilingualQuickRef). And finally, the differences in `StringIO` -
see more [here](http://python-future.org/compatible_idioms.html#stringio).

The final version of my test to check `open` is called with the correct arguments, looks like this:

```python
def test_io():
    with no_builtin_verification():
        i = __import__(('builtins' if sys.version_info >= (3,)
                        else '__builtin__'))
        expect(i).open.with_args(Config().config_file, 'w+').and_return(
            io.BytesIO()
        )
```
