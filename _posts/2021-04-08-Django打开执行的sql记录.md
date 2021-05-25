---
title: Django打开执行的sql记录
description: Django打开执行的sql记录
categories: 
- python    
tags:
- django   
---

#### 背景: 

排查数据库执行sql, 打开django orm 执行过程中执行的sql


[connection.queries](https://docs.djangoproject.com/en/3.2/faq/models/)

`Debug=True`模式下可用


非 Debug 模式 

[How to print sql query from django as Dubug is False](https://stackoverflow.com/questions/67683964/how-to-print-sql-query-from-django-as-dubug-is-false)

```python

from django.db import connection
from django.db import reset_queries

reset_queries()  #  clear the query
with transaction.atomic():
    r = FbHandleTask.objects.create(
    ...
    )
    
    accounts.filter(id__in=handle_ids_).update(status=4)
    
    fb_batch_operate_logger.info("[sql_execute]: {}".format(connection.queries))

```