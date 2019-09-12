---
layout: post
comments: true
title: Testing sqs in python
date: 2017-03-10 20:20:30
categories: python aws sqs
short_description: Mocking and testing sqs in python
---

AWS's [SQS](https://aws.amazon.com/sqs/) (simple queue service) is a nice way of not having to manage your own
[RabbitMQ](https://www.rabbitmq.com/), [Redis](https://redis.io/), etc., message broker. Testing it can be a
pain point however, since you can't have your CI start an instance for testing.

This is where [Moto](https://github.com/spulec/moto) comes in. Moto is a python library that mocks AWS service
endpoints, including SQS. Here is an example of testing that SQS receives a message using [py.test](http://doc.pytest.org/en/latest/). Moto works great with boto3 since it is mocking the http calls. We'll first create the queue since
SQS does not lazily create them. Whatever object or function you have that interacts with the queue, you'll need
to mock to post to the test queue. Here's a simple example of an object that creators messages.

```python
import boto3


client = boto3.client('sqs', 'us-east-1')


class MyMessageCreator(object):
    queue_url = client.get_queue_url(QueueName='your-queue-name')['QueueUrl']

    def send_message(self, message):
        client.send_message(
          QueueUrl=self.queue_url,
          MessageBody=message
        )
```
And here is the spec:
```python
import json

import pytest
from moto import mock_sqs
import boto3

import MyMessageCreator


@pytest.fixture
def sqs(scope='session', autouse=True):
    mock = mock_sqs()
    mock.start()

    sqs_client = boto3.client('sqs', 'us-west-1')
    queue_name = 'connect-responses-{}'.format(stage)
    queue_url = sqs_client.create_queue(
        QueueName='test'
    )['QueueUrl']

    yield (sqs_client, queue_url)

    mock.stop()

def test_message_received(sqs):
    client, queue_url = sqs
    MyMessageCreator.queue_url = queue_url
    creator = MyMessageCreator()
    creator.send_message({'key': 'value'})
    response = client.receive_messsage(QueueUrl=queue_url)
    assert response['Messages'][0]['Body'] == json.dumps({'key':  'value'})
```
