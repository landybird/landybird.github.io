---
title: filedescriptor超過限制
description: filedescriptor超過限制
categories:
- linux
tags:
- linux
---

<br>

#### filedescriptor超出限制



`select的最大處理能力為1024` 

    当 select 中监听的 fd 个数超过 1024 的时候，就会出现这个异常


錯誤出現的位置 
```
Traceback (most recent call last):
  File "/home/xxx/xxx/src/smb_thrift_server.py", line 2200, in <module>
    server.serve()
  File "/home/xxx/xxx/venv3/lib/python3.5/site-packages/thrift/server/TNonblockingServer.py", line 350, in serve
    self.handle()
  File "/home/xxx/xxxx/venv3/lib/python3.5/site-packages/thrift/server/TNonblockingServer.py", line 310, in handle
    rset, wset, xset = self._select()
  File "/home/luban/smb_thrift_server/venv3/lib/python3.5/site-packages/thrift/server/TNonblockingServer.py", line 302, in _select
    return select.select(readable, writable, readable)
ValueError: filedescriptor out of range in select()
```

可能原因 fd泄漏问题：

    单进程的 fd 个数升高，超出了 1024 限制


/proc/pid/fd/ 目录提供 了有关打开文件的信息
`/proc/pid/fd/`

`proc 是一个伪文件系统, 被用作内核数据结构的接口`

查看某進程下所以的fd
`lsof -p pid|wc -l`

要查看被进程使用的输入文件
`ls –l /proc/pid/fd/0`


查看被进程使用socket
`ls –l /proc/pid/fd|sed –n `/socket/{s/.*`
