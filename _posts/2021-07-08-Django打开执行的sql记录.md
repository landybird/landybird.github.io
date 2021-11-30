---
title: Django打开执行的sql记录
description: Django打开执行的sql记录, 获取可执行的SQL
categories: 
- python    
tags:
- django   
---

### 查看执行的SQL

#### 背景: 

排查数据库执行sql, 打开django orm 执行过程中执行的sql


#### debug模式 `Debug == True` 

[connection.queries](https://docs.djangoproject.com/en/3.2/faq/models/)

`Debug=True`模式下可用

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

#### 非debug模式 `Debug == False`


[How to print sql query from django as Dubug is False](https://stackoverflow.com/questions/67683964/how-to-print-sql-query-from-django-as-dubug-is-false)

[Database instrumentation](https://docs.djangoproject.com/en/3.2/topics/db/instrumentation/)

```python
class QueryLogger:

    def __init__(self):
        self.queries = []
        self.errored = False

    def __call__(self, execute, sql, params, many, context):
        current_query = {'sql': sql, 'params': params, 'many': many}
        try:
            result = execute(sql, params, many, context)
        except Exception as e:
            self.errored = True
            current_query['status'] = 'error'
            current_query['exception'] = e
            raise
        else:
            current_query['status'] = 'ok'
            return result
        finally:
            self.queries.append(current_query)


from django.db import connection


ql = QueryLogger()

with connection.execute_wrapper(ql):
    with transaction.atomic():
        r = Amodel.objects.create(
            ...
        )
        Bmodel.objects.filter(id__in=handle_ids_).update(status=4)

if not ql.errored:
    for query in ql.queries:
        print(query)
else:
    print("Some error occured")

```

### 2 获取可以执行的SQL


```
class QueryTransforms:
    """
    Django 不提供原始的SQL语句， 分别将query + param传入数据库adapter
    利用 cursor.execute EXPLAIN 截取 原始SQL 
    tricks (https://code.djangoproject.com/ticket/17741)
    """

    def __init__(self, queryset):
        self.queryset = queryset

    @property
    def query(self):
        query, params = self.queryset.query.sql_with_params()
        with connection.cursor() as cursor:
            cursor.execute('EXPLAIN ' + query, params)
            res = str(cursor.db.ops.last_executed_query(cursor, query, params))
            assert res.startswith('EXPLAIN ')
        return res[len('EXPLAIN '):]



```
