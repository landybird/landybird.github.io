---
title: fastapi与asgi(6)                                              
description: 多个router, background task, 静态文件, 其他
categories:
- python
tags:
- python   
---


#### 多router文件


    .
    ├── app
    │   ├── __init__.py
    │   ├── main.py
    │   └── routers
    │       ├── __init__.py
    │       ├── items.py
    │       └── users.py
    


`users.py`

```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "Foo"}, {"username": "Bar"}]


@router.get("/users/me", tags=["users"])
async def read_user_me():
    return {"username": "fakecurrentuser"}


@router.get("/users/{username}", tags=["users"])
async def read_user(username: str):
    return {"username": username}
```

`items.py`

```python
from fastapi import APIRouter, HTTPException

router = APIRouter()


@router.get("/")
async def read_items():
    return [{"name": "Item Foo"}, {"name": "item Bar"}]


@router.get("/{item_id}")
async def read_item(item_id: str):
    return {"name": "Fake Specific Item", "item_id": item_id}


@router.put(
    "/{item_id}",
    tags=["custom"],
    responses={403: {"description": "Operation forbidden"}},
)
async def update_item(item_id: str):
    if item_id != "foo":
        raise HTTPException(status_code=403, detail="You can only update the item: foo")
    return {"item_id": item_id, "name": "The Fighters"}
```

`main.py`


```python

from fastapi import Depends, FastAPI, Header, HTTPException

from .routers import items, users

app = FastAPI()


async def get_token_header(x_token: str = Header(...)):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")


app.include_router(users.router)
app.include_router(
    items.router,
    prefix="/items",
    tags=["items"],
    dependencies=[Depends(get_token_header)],
    responses={404: {"description": "Not found"}},
)

```


#### `Background task` 异步任务 run after returning a response
    
    Email notifications sent after performing an action:
        As connecting to an email server and sending an email tends to be "slow" (several seconds), 
        you can return the response right away and send the email notification in the background.
        
    Processing data:
        For example, let's say you receive a file that must go through a slow process, 
        you can return a response of "Accepted" (HTTP 202) and process it in the background.


```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()


def write_notification(email: str, message=""):
    with open("log.txt", mode="w") as email_file:
        content = f"notification for {email}: {message}"
        email_file.write(content)


@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_notification, email, message="some notification")
    return {"message": "Notification sent in the background"}


# 或 加上依赖 depndency


from fastapi import BackgroundTasks, Depends, FastAPI

app = FastAPI()


def write_log(message: str):
    with open("log.txt", mode="a") as log:
        log.write(message)


def get_query(background_tasks: BackgroundTasks, q: str = None):
    if q:
        message = f"found query: {q}\n"
        background_tasks.add_task(write_log, message)
    return q


@app.post("/send-notification/{email}")
async def send_notification(
    email: str, background_tasks: BackgroundTasks, q: str = Depends(get_query)
):
    message = f"message to {email}\n"
    background_tasks.add_task(write_log, message)
    return {"message": "Message sent"}

```


#### 静态文件

>  need to install `aiofiles`

`pip install aiofiles`


```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
# You could also use from starlette.staticfiles import StaticFiles

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"), name="static")

# /static ， 任何以“ / static”开头的路径都将由它处理
# directory =“ static”是指包含您的静态文件的目录的名称
# name =“ static”为其提供了一个名称，FastAPI可以在内部使用它




```


#### `FastAPI application` 整体的配置

```python

from fastapi import FastAPI

app = FastAPI(
    title="My Super Project",
    description="This is a very fancy project, with auto docs for the API and everything",
    version="2.5.0",
    docs_url="/documentation", 
    # default /docs
    redoc_url=None,
    openapi_url="/api/v1/openapi.json"
    # By default, the OpenAPI schema is served at /openapi.json
)


@app.get("/items/")
async def read_items():
    return [{"name": "Foo"}]
```
