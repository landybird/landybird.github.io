---
title: django + celery + rabbitMQ
description: django+celery+rabbitMQ
categories:
- python
tags:
- django
---

<br>


# django + celery + rabbitMQ：

<br>

django 自带的` signals` 信号机制是`同步的`

<br>

##  应用场景

`耗时的储存过程` 或者是其他`数据库操作`，可以放在`消息队列中处理`, 加上`多个worker`进程，可以`分布式的处理任务`，更有效率

先把结果返回给用户, 处理放在后台处理


`django`：web框架；

`RabbitMQ`：消息队列系统，负责`存储消息`；

`celery`：worker进程，同时提供在webapp中`创建任务`的功能



<br>

## 下载安装 rabbitMQ ,celery

（1）rabbitMQ (需要erlang环境)

`window` [rabbitMQ地址](http://www.rabbitmq.com/install-windows.html)
        [erlang地址](http://www.erlang.org/downloads)
        
> 安装运行遇到问题（1）：

运行命令 rabbitmq-server出现如下问题
  
    *********************************
    ERLANG_HOME not set correctly.
    *******************************
    
    Please either set ERLANG_HOME to point to your Erlang installation or place the
    RabbitMQ server distribution in the Erlang lib folder.

vim rabbitmq-server.bat 发现
    
    if not exist "!ERLANG_HOME!\bin\erl.exe" (
        echo.
        echo ******************************
        echo ERLANG_HOME not set correctly.
        echo ******************************


需要配置  ERLANG_HOME 环境变量 指定erl.exe 的位置


> 运行遇到问题 ( 2 ):

    ERROR: node with name "rabbit" already running on  ....
    

查看erl运行状态
```ini

tasklist | findstr erl
    
erl.exe                      22196 Services                   0     76,276 K

```

终止任务

```ini
taskkill /pid 22196 /f 

```

重新启动

 rabbitmq-plugins enable rabbitmq_management  安装插件


 rabbitmq-server
 
    "WARNING: Using RABBITMQ_ADVANCED_CONFIG_FILE: C:\Users\Domob\AppData\Roaming\RabbitMQ\advanced.config"
    
      ##  ##
      ##  ##      RabbitMQ 3.7.9. Copyright (C) 2007-2018 Pivotal Software, Inc.
      ##########  Licensed under the MPL.  See http://www.rabbitmq.com/
      ######  ##
      ##########  Logs: C:/Users/Domob/AppData/Roaming/RabbitMQ/log/RABBIT~1.LOG
                        C:/Users/Domob/AppData/Roaming/RabbitMQ/log/rabbit@DESKTOP-41HH3R2_upgrade.log


访问 http://127.0.0.1:15672 

 ![](https://landybird.github.io/landybird.github.io/assets/images/rq1.png)


`linux` 

    apt-get install rabbitmq-server
    

> 简单配置使用

    λ rabbitmqctl add_user jimmy jimmy
    
            Adding user "jimmy" ...
    
    λ rabbitmqctl.bat  list_users
    
            Listing users ...
            user    tags
            guest   [administrator]
            jimmy   []
    
    
    λ rabbitmqctl  set_user_tags jimmy administator
    
            Setting tags for user "jimmy" to [administator] ...
    
    λ rabbitmqctl add_vhost vhost
        
            Adding vhost "vhost" ...

    λ rabbitmqctl set_permissions -p vhost jimmy ".*" ".*" ".*"
    
            Setting permissions for user "jimmy" in vhost "vhost" ...
            


(2) celery 安装  

`pip install django-celery`

注意最新的`celery >=4.0` 版本中 `win 环境下会有问题` ,添加任务的时候： `Task handler raised error: ValueError('not enough values to unpack (expected 3, got 0)',)`


解决办法：
    
    pip install eventlet
    celery -A <mymodule> worker -l info -P eventlet
    


<br>

##  django 中的配置， 以及使用

注意 django-celery 不再更新同步 最新的celery

```python 

1 testCelery/celery.py 
    
    from __future__ import absolute_import, unicode_literals
    import os
    from celery import Celery
    from django.conf import settings
    
    # set the default Django settings module for the 'celery' program.
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'testCelery.settings')
    
    app = Celery('testCelery')
    
    # Using a string here means the worker doesn't have to serialize
    # the configuration object to child processes.
    # - namespace='CELERY' means all celery-related configuration keys
    #   should have a `CELERY_` prefix.
    app.config_from_object('django.conf:settings')
    
    # Load task modules from all registered Django app configs.
    app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)
    
    
    @app.task(bind=True)
    def debug_task(self):
        print('Request: {0!r}'.format(self.request))


2  testCelery/settings.py
    
    
    INSTALLED_APPS = [
        ...,
        'djcelery',
        'task'
    ]

    
    import djcelery
    djcelery.setup_loader()
    #
    # BROKER_HOST = "localhost"
    # BROKER_PORT = 5672
    # BROKER_USER = "guest"
    # BROKER_PASSWORD = "guest"
    # BROKER_VHOST = "/"
    BROKER_URL = 'redis://127.0.0.1:6379/0'
    CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/0'
    CELERY_ACCEPT_CONTENT = ['json']
    CELERY_TASK_SERIALIZER = 'json'
    CELERY_RESULT_SERIALIZER = 'json'
    CELERYBEAT_SCHEDULER = 'djcelery.schedulers.DatabaseScheduler'
    CELERY_TIMEZONE = 'Asia/Shanghai'



3  testCelery/__init__.py
    
        from __future__ import absolute_import, unicode_literals
        
        # This will make sure the app is always imported when
        # Django starts so that shared_task will use this app.
        from .celery import app as celery_app
        
        __all__ = ('celery_app',)



4  task/tasks.py
    
        from __future__ import absolute_import, unicode_literals
        from celery import shared_task, task
        from testCelery import celery_app
        
        
        @task
        def add(x, y):
            return x + y
        
        
        @task
        def mul(x, y):
            return x * y
        
        
        @task
        def xsum(numbers):
            return sum(numbers)
    

5  views.py 实际业务处理中
    
     from task.tasks import add
     
     def myview(request):
         add(1, 2)  使用异步任务
    
    
```
    
    
    
## 查看celery任务队列的状态 `flower`

Celery提供了一个工具flower，将各个任务的执行情况、各个worker的健康状态进行监控并以可视化的方式展现

```python 

pip install flower 

python manage.py celery flower  

# 注意django-celery 与 celery 的版本兼容性 可能导致 flower不能使用


```




    

