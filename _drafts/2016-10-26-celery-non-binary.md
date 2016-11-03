---
layout: post
comments: true
title: Celery MongoDB backend binary results
date: 2016-10-26 20:20:23
categories: python celery
short_description: How to store non-binary Celery results in MongoDB
image_preview:https://media.giphy.com/media/3o7TKDxSkF9iGiCJri/giphy.gif
---

By default, when you use MongoDB as your result backend for <a target='_blank'  href='http://www.celeryproject.org/'> Celery</a> the result and traceback are stored as a Binary data type. This is very efficient since you can potentially have large results and tracebacks. However, if you're storing the results in MongoDB, it's probably because you want to query against them.

It's easy to have your tasks return dictionaries, which in turn will be stored as BSON in MongoDB, making them easy to retrieve. Also, MongoDB's <a target='_blank' href='http://docs.mongodb.org/manual/core/aggregation-pipeline/'>aggregation pipeline</a> can be used to run analytic queries on your data. You should also consider using MongoDB's <a target='_blank' href='http://docs.mongodb.org/manual/core/capped-collections/'>capped collections</a> for your logs. This will allow you to limit the disk space you'll be using in your logging without having to add any additional logic. MongoDB makes for a really powerful Celery backend.

To disable storing these fields as Binary objects, the relevant functions are on the on the `MongoBackend` object
in `celery.backends.mongodb`. The easiest thing to do is create a new backend subclassed from the `MongoBackend`
object. We need to override three functions on our new `MongoBackend`. Those are: `_store_result`, `_save_group`,
and `_get_task_meta_for`. From these methods, we simply have to remove the functions wrapped in the Binary function.

The updated backend should look like this:

```python
from celery.backends import mongodb
from celery import states
from datetime import datetime
from celery.app import task

class CustomMongoBackend(mongodb.MongoBackend):
    """
    This backend removes the default storing of result data as Binary.
    """
    def _store_result(self, task_id, result, status,
                      traceback=None, request=None, **kwargs):
        """Store return value and status of an executed task."""
        meta = {'_id': task_id,
                'status': status,
                'result': result, 'date_done': datetime.utcnow(),
                'task': request.task,
                'traceback': self.encode(traceback),
                'children': self.encode(
                    self.current_task_children(request),
                )}
        self.collection.save(meta)
        return result

    def _save_group(self, group_id, result):
        """Save the group result."""
        meta = {'_id': group_id,
                'result': self.encode(result),
                'date_done': datetime.utcnow()}
        self.collection.save(meta)
        return result

    def _get_task_meta_for(self, task_id):
        """Get task metadata for a task by id."""
        obj = self.collection.find_one({'_id': task_id})
        if not obj:
            return {'status': states.PENDING, 'result': None}
        meta = {
            'task_id': obj['_id'],
            'status': obj['status'],
            'result': obj['result'],
            'date_done': obj['date_done'],
            'traceback': self.decode(obj['traceback']),
            'children': self.decode(obj['children']),
        }
        return meta
```

Stick this class in a module in your project, like `backends.py`. You can then use this class as a backend by
configuring Celery in the following way:

```python
CELERY_RESULT_BACKEND = 'backends:CustomMongoBackend'
CELERY_MONGODB_BACKEND_SETTINGS = {
    'host': '127.0.0.1',
    'user': 'admin',
    'password': 'password',
    'port': 27017,
    'database': 'celery',
    'taskmeta_collection': 'log'
}
```

You should be all set now to look at your MongoDB collection and view the results directly, without having to decode them.

Again, if you are storing large results and tracebacks and rarely viewing the logs, it's probably best to store the results as Binary.
