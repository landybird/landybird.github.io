---
title: MySQL server has gone away
description: MySQL server has gone away
categories:
- python
tags:
- python基础
---

<br>

# MySQL server has gone away

<br>

`mysql server has gone away` 的 原因

<table summary="MySQL server has gone away error codes and a description of each code."><colgroup><col width="35%"><col width="65%"></colgroup><thead><tr>
              <th scope="col">Error Code</th>
              <th scope="col">Description</th>
            </tr></thead><tbody><tr>
              <td scope="row"><a class="link" href="client-error-reference.html#error_cr_server_gone_error"><code class="literal">CR_SERVER_GONE_ERROR</code></a></td>
              <td>The client couldn't send a question to the server.</td>
            </tr><tr>
              <td scope="row"><a class="link" href="client-error-reference.html#error_cr_server_lost"><code class="literal">CR_SERVER_LOST</code></a></td>
              <td>The client didn't get an error when writing to the server, but it didn't
                get a full answer (or any answer) to the question.</td>
</tr></tbody></table>

<br>
> (1) 默认情况下 mysql server 会在没有响应的 8个小时后 关闭:

```sql
SHOW GLOBAL VARIABLES LIKE '%timeout';

# wait_timeout	28800s  #8hs
```

这样需要程序在断开连接后再重新获取请求链接, 进行 `reconnection `


> (2) 这种情况是 DB管理员 杀掉了 mysql的进程

> (3) 是在 链接关闭之后 进行 数据库查询操作


> (4) 连接超时， 可以通过`mysql_ping()`进行判断

> (5) `max_allowed_packet` 默认`64MB` 请求的query过大, 例如 `BLOB column`  

> (6) 子进程fork公用同一个mysql connection, 再一次关闭后会断开连接
  
    #需要每个进程单独建立链接


数据库的链接是不可fork的， Basically connections 'are not forkable'... Each process should have it own connection


    Multiprocessing copies connection objects between processes because it forks processes, and therefore copies all the file descriptors of the parent process. 
    That being said, a connection to the SQL server is just a file, you can see it in linux under /proc//fd/.... any open file will be shared between forked processes.
    
解决的方法在子进程 或者程序中重新连接数据库
 
使用`apscheduler` 调用任务中用到了 `django`的orm，所以每次在调用task的时候，在task里面需要执行

`django.db.connections.close_all()`