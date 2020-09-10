---
title: Django的async支持 `Django3.1+`    
description: Django 3.1 Async   
categories:
- python
tags:
- django   
---
    

[ >> django-31-async](https://wersdoerfer.de/blogs/ephes_blog/django-31-async/)

Django 3.1 中加入了对 `async` views, middlewares and test 的支持

    noticce: 数据库操作还没有被支持
    

这个新特性可以用于 `同时`处理大量任务



### async视图示例

```python

python -m venv mysite_venv && source mysite_venv/bin/activate

django-admin startproject mysite .  # create django project in current directory
python manage.py migrate            # migrate sqlite
python manage.py runserver          # should start the development server now


# views 
import time

from django.http import JsonResponse
from django.urls import path

def api(request):
    time.sleep(1)
    payload = {"message": "Hello World"}
    if "task_id" in request.GET:
        payload["task_id"] = request.GET["task_id"]

    return JsonResponse(payload)


import httpx
import asyncio

def get_api_urls(num=10):
    base_url = "http://127.0.0.1:9008/api/"
    return [
        f"{base_url}?task_id={task_id}" for task_id in range(num)
    ]

async def api_aggregated(request):
    s = time.perf_counter()
    responses = []
    urls = get_api_urls(num=10)
    async with httpx.AsyncClient() as client:
        responses = await asyncio.gather(*[
            client.get(url) for url in urls
        ])
        responses = [r.json() for r in responses]

    elapsed = time.perf_counter() - s
    result = {
        "message": "Hello Async World",
        "responses": responses,
        "debug_message": f"fecth executed in {elapsed:0.2f} seconds.",
    }

    return JsonResponse(result)

#{"message": "Hello Async World", "responses": [{"message": "Hello World", "task_id": "0"}, {"message": "Hello World", "task_id": "1"}, {"message": "Hello World", "task_id": "2"}, {"message": "Hello World", "task_id": "3"}, {"message": "Hello World", "task_id": "4"}, {"message": "Hello World", "task_id": "5"}, {"message": "Hello World", "task_id": "6"}, {"message": "Hello World", "task_id": "7"}, {"message": "Hello World", "task_id": "8"}, {"message": "Hello World", "task_id": "9"}],
# "debug_message": "fecth executed in 1.08 seconds."}

def api_aggregated_sync(request):
    s = time.perf_counter()
    responses = []
    urls = get_api_urls(num=10)
    for url in urls:
        r = httpx.get(url)
        responses.append(r.json())
    elapsed = time.perf_counter() - s
    result = {
        "message": "Hello Sync World!",
        "aggregated_responses": responses,
        "debug_message": f"fetch executed in {elapsed:0.2f} seconds.",
    }
    return JsonResponse(result)

# {"message": "Hello Sync World!", "aggregated_responses": [{"message": "Hello World", "task_id": "0"}, {"message": "Hello World", "task_id": "1"}, {"message": "Hello World", "task_id": "2"}, {"message": "Hello World", "task_id": "3"}, {"message": "Hello World", "task_id": "4"}, {"message": "Hello World", "task_id": "5"}, {"message": "Hello World", "task_id": "6"}, {"message": "Hello World", "task_id": "7"}, {"message": "Hello World", "task_id": "8"}, {"message": "Hello World", "task_id": "9"}],
#  "debug_message": "fetch executed in 10.38 seconds."}



# urls 
from django.contrib import admin
from django.urls import path
from . import views

urlpatterns = [
    path('api/', views.api),
    path('api/aggregated/', views.api_aggregated),
    path('api/aggregated_sync/', views.api_aggregated_sync),
]
```


Django(内置的开发服务器)支持`async`视图, 可以在async视图中完成对第三方api的并发请求

但是WSGI服务下的应用, 每个view请求仍然是运行各自的线程上的



- `ASGI` 服务器 ([uvicorn](https://www.uvicorn.org/))


```python
python -m pip install uvicorn
uvicorn --workers 18 --reload untitled11.asgi:application --host 10.0.0.206 --port 9006
# 需要把访问的函数写成 async 的形式
```


- `sync_and_async_middleware` 中间件

```python

# middleware.py
import json
import time
import asyncio

from django.http import JsonResponse
from django.utils.decorators import sync_and_async_middleware

def add_elapsed_time(response, start):
    data = json.loads(response.content)
    data["elapsed"] = time.perf_counter() - start
    response = JsonResponse(data)
    return response

@sync_and_async_middleware
def timing_middleware(get_response):
    if asyncio.iscoroutinefunction(get_response):
        async def middleware(request):
            start = time.perf_counter()
            response = await get_response(request)
            response = add_elapsed_time(response, start)
            return response
    else:
        def middleware(request):
            start = time.perf_counter()
            response = get_response(request)
            response = add_elapsed_time(response, start)
            return response

    return middleware



# settings.py
MIDDLEWARE = [
    ...
    "mysite.middleware.timing_middleware",
]
```



- 为什么要使用 `async` -- 解决高并发的问题 


应用程序必须同时执行的任务数量增加时

    1 启动更多的机器
    2 一台计算机上使用更多的内核
    3 使用更多的操作系统进程
    4 每个进程启动更多的操作系统线程
    5 在单个线程中使用async / await安排多个任务

CPython 使用多核只能通过更多的操作系统进程


当任务是IO密集的时候:

    向数据库提交sql查询并获取结果
    
    汇总来自不同API端点或微服务的答案
    
    从elasticsearch或类似的全文本搜索服务
    
    从缓存中获取结果
    
    获取网站片段从磁盘加载一些图像
    

 Python线程是`操作系统线程`，而OS内核可以决定计划运行哪些线程。 
 因此，你根本不知道何时将控制流转移到另一个线程。 它可能随时发生
 

而`异步任务async`是不同的。他们使用协作式多任务处理，每个任务都可以在放弃控制权时自行决定。
只要不使用await关键字来表示现在可以同时执行其他任务，代码就可以像正常的同步程序一样顺序运行



-  `Async Adapter` 给ORM和Django其他尚未具备异步功能的函数加上适配器


[Async adapter functions¶](https://docs.djangoproject.com/en/3.1/topics/async/#async-adapter-functions)

从异步上下文中调用同步代码时，有必要调整调用样式，反之亦然。 
为此，从`asgiref.sync模块`中有两个适配器函数：`async_to_sync（）`和`sync_to_async（）`。
它们用于在调用样式之间进行转换，同时保持兼容性。 这些适配器功能已在Django中广泛使用。
 `asgiref`包本身是Django项目的一部分，当您使用pip安装Django时，它将自动作为依赖项安装。

`async_to_sync()` which will take care of setting up the event loop but also makes sure threadlocals will work.

    负责设置事件循环
    
    而且确保threadlocals可以正常工作
    
```python
from asgiref.sync import async_to_sync

# 1
async def get_data(...):
    ...

sync_get_data = async_to_sync(get_data)

# 2 
@async_to_sync
async def get_other_data(...):
    ...


# 3 
from asgiref.sync import sync_to_async

async_function = sync_to_async(sync_function)
async_function = sync_to_async(sensitive_sync_function, thread_sensitive=True)

@sync_to_async
def sync_function(...):
    ...

```

`sync_to_async（）`具有两种线程模式：

`thread_sensitive = False（默认值）`：同步函数将在全新的线程中运行，一旦调用完成，该线程将关闭。 

`thread_sensitive = True`: 同步函数将与所有其他thread_sensitive函数在同一线程中运行。

如果主线程是同步的并且您正在使用async_to_sync（）包装器，则它将是主线程。


### thread or async  线程 和 协程 


`linux`和`macOS`上新线程的`默认堆栈大小（ulimit -s）为8MB`。 
但这并不意味着这是启动附加线程的实际内存开销。 

首先，它是虚拟内存而不是驻留内存，其次对32位计算机（可用虚拟内存只有3GB）施加了硬性和较低的线程数量限制，
但在64位计算机上，此限制不再适用

`async tasks` 开销 2KB memory 

`Python 2`上的锁争用问题。Python解释器每100个滴答声就会检查另一个线程是否应该能够获取GIL。
 即使在单个CPU上，这也会导致性能降低，但是在具有多核的计算机上，这尤其糟糕，因为现在线程将争夺并行地在不同CPU上获取GIL。
  
 这些问题已通过`Python 3.2`中引入的`新GIL修复`，现在每隔5毫秒调用一次检查（可通过`sys.setswitchinterval`对其进行配置）。



如果您只想同时汇总一些请求结果，则`线程池`可能就足够了。 
您无需使用其他支持`ASGI`的应用服务器，这也适用于较早的Django版本


