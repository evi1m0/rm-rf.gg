---
layout: post
title: Python-Django Celery BUG01
---

#### Celery

> Celery is an asynchronous task queue/job queue based on distributed message passing.
> 
> It is focused on real-time operation, but supports scheduling as well.
>
> The execution units, called tasks, are executed concurrently on a single or more worker servers using multiprocessing, Eventlet, or gevent. Tasks can execute asynchronously (in the background) or synchronously (wait until ready).

Celery是Python开发的分布式任务调度模块，今天抽空看了一下，果然接口简单，开发容易，5分钟就写出了一个异步发送邮件的服务。

Celery本身不含消息服务，它使用第三方消息服务来传递任务，目前，Celery支持的消息服务有RabbitMQ、Redis甚至是数据库，当然Redis应该是最佳选择。

#### Install Celery

pip: 

    $ sudo pip install Celery

easy_install:

    $ sudo easy_install Celery

#### Install Redis

Ubuntu:

    wget http://download.redis.io/releases/redis-2.8.9.tar.gz
    tar xvfz redis-2.8.9.tar.gz
    cd redis-2.8.9
    sudo make
    sudo make install
    
Celery-Redis:

    $ pip install -U celery[redis]
    
Setting password (redis):

    vim /etc/redis/redis.conf
    
    requirepass passwordxxx

#### Python-Django BUG 01 - Fixed

我将要把 Django 项目的一个模块使用 Celery 进行调度，但使用过程中发现了恼火的问题：

    Received unregistered task of type 'tasks.add'.
    The message has been ignored and discarded.

    Did you remember to import the module containing this task?
    Or maybe you are using relative imports?
    Please see http://bit.ly/gLye1c for more information.

    The full contents of the message body was:
    {'retries': 0, 'task': 'tasks.add', 'utc': False, 'args': (4, 4), 'expires': None, 'eta': None, 'kwargs': {}, 'id': '841bc21f-8124-436b-92f1-e3b62cafdfe7'}

    Traceback (most recent call last):
      File "/usr/local/lib/python2.7/dist-packages/celery/worker/consumer.py", line 444, in receive_message
        self.strategies[name](message, body, message.ack_log_error)
    KeyError: 'tasks.add'
    
按照官方的推荐在当前目录下新建了 Celerycibfug.py:

    BROKER_URL = 'redis://:linux@localhost:6379//'
    CELERY_TASK_SERIALIZER = 'json'
    CELERY_RESULT_SERIALIZER = 'json'
    CELERY_ENABLE_UTC = True
    CELERY_IMPORTS = ("scanner",)
    
RUN:

    celeryd --config=celeryconfig --loglevel=INFO
    
仍然会报错，最后发现仅仅是需要给 @app.task 修饰器定义 name 值：

    @app.task(name='sxs-scanner')
    def main(test, data=None)
        ...

就这样，这个问题成功的修复并 RUN 了起来，真是累心：
        
<center>
<img src="http://ww3.sinaimg.cn/large/c334041btw1eqk59nu98lj20rp08zdll.jpg" />
</center>