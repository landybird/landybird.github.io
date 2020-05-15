---
title: nginx和uwsgi超时                        
description: nginx和uwsgi超时
categories:
- python
tags:
- python   
---

#### `uwsgi` 的设置

- http-timeout和socket-timeout（连接时间）

http-timeout=60 # 就是60秒
一般情况下，我们的uwsgi都是配合nginx使用的，所以用的都是socket-timeout参数。
这两者的区别简单说就是：uwsgi单独使用就用http, 配合nginx就用socket

解释下这两个时间的意义：
举个例子：
	前端（客户端）访问后端（服务器），服务器需要运行5分钟才能给前端返回响应，
	但是http-timeout或者socket-timeout设置的是60，那么一分钟后，我的前端和后端
	就断开连接了， 
	！但是！我的服务器还是会坚持把这5分钟的活干完，只不过没有办法给前端返回
	响应了！
	（顾客去餐厅吃饭，做饭需要10分钟才能上菜，顾客等了1分钟就跑路了！）
1
2
3
4
5
6
7
8
2.harakiri(服务器响应时间）
harakiri=60 # 就是60秒

和http-timeout有点类似，举个例子：
	前端（客户端）向后端（服务器）发送到一个请求，等待服务器响应，服务器
	需要1分钟来计算数据，但是我的harakiri就设置了10秒，那么10秒一到，
	我们的服务器就强制终止了计算，前端肯定就得不到响应了。
	（老板给员工发了一个任务，这个任务需要5天完成，这个员工干了一天
	就撂挑子了！）
1
2
3
4
5
6
3.buffer-size（前后端传输数据大小）
buffer-size=1024 # 就是1024k，1M

这个容易理解，比如前段（客户端）向后端（服务器）发了一个请求，这个
请求的大小是5M，那么buffer-size的大小就得大于1024*5，不然就报错了
