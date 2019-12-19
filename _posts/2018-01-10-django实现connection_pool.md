---
title: django 数据库连接池 Django and SQLAlchemy
description: Django and SQLAlchemy 数据库连接池
categories:
- python
tags:
- django
---

<br>


# 数据库连接池 `Django` and `SQLAlchemy`

<br>

在连接线程进行query之后， Django会自动的断开连接

例如在执行

    q = Model.objects.all() 

后

当变量 "q" 回收的同时数据库也会关闭，我们可以使用 `SQLAlchemy`
来缓存 db connection。

    使用 SQLAlchemy, 只有当前进程程序退出时， db connection 才会关闭


使用的步骤:

>1 安装 `Install SQL Alchemy`

>2 拷贝原始的 mysql backend的package, 用来修改

    一般位置在 django.db.backends.mysql
    
拷贝之后，把这个pacakge加到 `PYTHONPATH` 中, 命名为 `db_pool`

    DATABASE_ENGINE = 'db_pool' 

>3 修改相关的db配置 Update settings.py and database settings

settings.py
    
    这个是默认的超时时间
    DATABASE_WAIT_TIMEOUT = 120
    
    settings.py 中的设置与 db保持一致
    
>4 修改package `db_pool`中的内容

```python

try:
    from settings import DATABASE_WAIT_TIMEOUT
except ImportError:
    print u'DATABASE_WAIT_TIMEOUT not in settings.py, defaulting to 120.'
    DATABASE_WAIT_TIMEOUT = 120

import sqlalchemy.pool as pool 

Database = pool.manage(Database, recycle=DATABASE_WAIT_TIMEOUT-1) # must match or be less than wait_timeout in mysql 

if settings.DATABASE_HOST.startswith('/'):
    self.connection = Database.connect(port=kwargs['port'], unix_socket=kwargs['unix_socket'], user=kwargs['user'], db=kwargs['db'], passwd=kwargs['passwd'], use_unicode=kwargs['use_unicode'], charset='utf8')
else:
    self.connection = Database.connect(host=kwargs['host'], port=kwargs['port'], user=kwargs['user'], db=kwargs['db'], passwd=kwargs['passwd'], use_unicode=kwargs['use_unicode'], charset='utf8') 
```
  
