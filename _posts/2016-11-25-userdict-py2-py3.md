---
layout: post
comments: true
title: Using `UserDict` in Py2 and Py3
date: 2016-11-25 20:20:30
categories: python
short_description: How to use `UserDict` in Python 2 and 3
image_preview: /images/frank1.gif
---

If you want to create a dict like object using `UserDict` in Python 2 and 3, there's
a couple of things to keep in mind.

1. `UserDict` has moved to the collections module in Py3.
2. `UserDict` in Py2 is an "old-style" class. This will cause problems if using `super`.

I ran into a couple issues trying to create a dict like object that is kept in sync
with a yaml file. Here's how I made it compatible with both Python versions
(not the full implementation here):

```python
try:
    from UserDict import UserDict
except ImportError:
    from collections import UserDict

import yaml


class YamlDict(UserDict, object):
    def update(*args, **kwargs):
        super(YamlDict, self).update(*args, **kwargs)
        self._save()

    def _save(self):
        with open('path_to_file', 'w+') as f:
            f.write(yaml.dump(self.data)
```

Mixing in `object` allows you to use `super`.
