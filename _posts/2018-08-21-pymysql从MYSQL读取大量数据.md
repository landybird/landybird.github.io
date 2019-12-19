---
title: pymysql从MYSQL读取大量数据
description: pymysql从MYSQL读取大量数据
categories:
- python
tags:
- python基础
---

<br>


### pymysql从MYSQL读取大量数据：

一次性读取大量的数据，默认情况下会读取所有的数据待内存中去，可能会使得进程被杀掉.

解决这个问题的方式是使用`流式的cursor`, 在`python`中 可以使用 `MySQLdb.cursor.SSCursor`

```python

import MySQLdb
conn = MySQLdb.connect(...)
cursor = MySQLdb.SSCursor(conn)
cursor.execute(...)
while True:
    row = cursor.fetchone()
    if not row:
        break
    ...
```

需要注意的是:

> 1 不要使用 `fetchall`
 
 需要使用 `fetchone` ,或者`fetchmany` (实际也是调用的多次的fetchone) 而不要使用 `fetchall`

> 2 SSCursor 不是 mysql server 端的 cursor
 
 它只是一个`streaming result set`
 
 
> 3 必须读取所有的数据 `You must read ALL records. `

      The rational is that you send one query, and the server replies with one answer, albeit a really long one. 
    Therefore, before you can do anything else, even a simple ping, you must completely finish this response.



> 4 每一条数据的处理必须快速 `you must process each row quickly.` 

mysql默认的处理时间是60s `by default MySQL will wait for a socket write to finish in 60 seconds`
 
        If your processing takes even half a second for each row, you will find your connection dropped unexpectedly with error 2013, 
     "Lost connection to MySQL server during query." The reason is by default MySQL will wait for a socket write to finish in 60 seconds. 
     The server is trying to dump large amount of data down the wire, yet the client is taking its time to process chunk by chunk.
     So, the server is likely to just give up. You can increase this timeout by issuing a query 
     SET NET_WRITE_TIMEOUT = xx where xx is the number of seconds that MySQL will wait for a socket write to complete. 
     But please do not rely on that to be a workable remedy. Fix your processing instead. Or if you cannot reduce processing time any further, 
     you can quickly chuck the rows somewhere local to complete the query first, and then read them back later at a more leisure rate.

> 5 如果需要不等待， 并行的操作mysql 需要再建立一个连接

    If you need to run another query in parallel, do it in another connection. 
    Otherwise, you will get error 2014, "Commands out of sync; you can't run this command now."
    
        




