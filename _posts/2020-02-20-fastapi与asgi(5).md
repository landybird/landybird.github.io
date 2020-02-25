---
title: fastapi与asgi(4)                                              
description: fastapi 中 的中间件 Middleware
categories:
- python
tags:
- python   
---


> custom proprietary headers can be added using the `'X-' prefix` 自定义的header 应该使用 `'X-' prefix`





在 `FastAPI applications` 中可以增加 `Middleware`

    
    接受应用程序中的每个请求
    
    可以对该请求执行某些操作或运行任何所需的代码
    
    传递要由应用程序其余部分处理的请求（通过某些路径操作）
    
    将获取应用程序生成的响应（通过某些路径操作）
    
    可以对响应做出响应或运行任何需要的代码。 然后返回响应。



#### 创建中间件 `@app.middleware("http") `


```python
import time

from fastapi import Depends, FastAPI
from starlette.requests import Request

app = FastAPI()

@app.get("/test/")
async def read_users_me():
    return {"a": 111}



@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    print(request.__dict__)

    # before response
    response = await call_next(request)

    # after response
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response

```




#### 复用 `Starlette's Middleware`


[Starlette Middleware](https://www.starlette.io/middleware/)


        
    CORSMiddleware 
    
    GZipMiddleware.
    
    SentryMiddleware.
    
    HTTPSRedirectMiddleware
    
    TrustedHostMiddleware
    
    ..
    
    

#### `CORSMiddleware` 与 `CORS` (Cross-Origin Resource Sharing) 跨资源共享



`CORS`或 `“跨源资源共享”`是指 浏览器中运行的前端具有与后端进行通信的JavaScript代码，并且后端与前端具有不同的“来源”的情况


`origin` is the combination of 


    protocol (http, https)
    
    domain (myapp.com, localhost, localhost.tiangolo.com)
    
    port (80, 443, 8080).


different origins:
    
    http://localhost
    https://localhost
    http://localhost:8080


> 前后端跨域请求过程



    前端 http://localhost:8080  -->> 后端 http://localhost:80 


浏览器会发送一个  `HTTP OPTIONS request ` 到后端, 如果后端返回`适当的headers (允许通讯进行)`, 这样 浏览器会允许`前端向跨域的后端发送请求`


后端需要设置一个 `allowed origins` list


> 使用通配符号 `*`有局限性 (不支持cookie等凭证信息)

    简便
    
    仅允许某些类型的通信
    
    但是不支持涉及凭据的所有内容：Cookie，授权标头（如与Bearer Token一起使用的那些标头等）




> 使用 `CORSMiddleware`


```python

from fastapi import FastAPI
from starlette.middleware.cors import CORSMiddleware

app = FastAPI()

origins = [
    "http://localhost.tiangolo.com",
    "https://localhost.tiangolo.com",
    "http://localhost",
    "http://localhost:8080",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,   
    allow_credentials=True,  # Credentials (Authorization headers, Cookies, etc)
    allow_methods=["*"],     # Specific HTTP methods (POST, PUT) or all of them with the wildcard "*".
    allow_headers=["*"],     # Specific HTTP headers or all of them with the wildcard "*".
)


```