---
title: uvloop加速asyncio      
description: uvloop加速asyncio   
categories:
- python
tags:
- python   
---
    

based on `libuv` [libuv](https://github.com/libuv/libuv)

`libuv`是一种高性能的、跨平台异步的 `I/O 类库`，nodejs也使用到了它。

    由于nodejs是如此的广泛和流行,可以知道libuv是快速且稳定的。
    
    利用uvloop可以写出在单CPU内核下每秒钟能够发出上万个请求的Python网络代码
    

#### 基础概念


[uvloop](https://uvloop.readthedocs.io/)

TCP, HTTP 请求的性能比较

[uvloop-blazing-fast-python-networking](http://magic.io/blog/uvloop-blazing-fast-python-networking/)

uvloop makes asyncio fast. In fact, it is at least 2x faster than nodejs, gevent, as well as any other Python asynchronous framework.

The performance of uvloop-based asyncio is close to that of Go programs.


Uvloop最终目的使得Asyncio更加快速


#### 使用


```python


pip install --upgrade pip 

pip install uvloop


# 1
# To make asyncio use the event loop provided by uvloop, you install the uvloop event loop policy:

import asyncio
import uvloop
asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())

# 2 
# Alternatively, you can create an instance of the loop manually, using:

import asyncio
import uvloop
loop = uvloop.new_event_loop()
asyncio.set_event_loop(loop)
```
