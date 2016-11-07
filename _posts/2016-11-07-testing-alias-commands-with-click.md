---
layout: post
comments: true
title: Testing aliased commands with Click
date: 2016-11-07 21:20:30
categories: ruby rails
short_description: How to test aliased commands with click
image_preview: http://gifrific.com/wp-content/uploads/2013/01/iasip-dance.gif
---

[Click](http://click.pocoo.org/5/) is an awesome library that helps to quickly build command line apps.
I've been using it build a command line tool, [Accountable](https://github.com/wohlgejm/accountable), that
integrates Jira and git.

Testing is well [documented](http://click.pocoo.org/5/testing/), but it took me a minute to figure out how
to test a command I aliased.

Aliasing commands is documented [here](http://click.pocoo.org/5/advanced/). My class ended up looking almost exactly
the same expect I used a dictionary to look up available aliases since I want to try to match commands.

In the docs, a `Command` is passed directing into `CliRunner.invoke`. `invoke` can also take your entire cli
and then the alias command can be passed in the list of arguments.

`invoke` just calls `main` on the first argument, which is implemented by the individual `Command` and
your cli with your alias subclass.

So your tests (using py.test) can end up looking something like this:

```python
from click.testing import CliRunner

from mycli import cli


def test_full_name():
    runner = CliRunner()
    result = runner.invoke(cli.full_name)
    assert result.output = 'invoked full name'


def test_alias():
    runner = CliRunner()
    result = runner.invoke(cli.cli, ['alias'])
    assert result.output = 'invoked full name'
```
