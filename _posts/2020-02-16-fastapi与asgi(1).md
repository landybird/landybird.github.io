---
title: fastapi与asgi(1)                                              
description: fastapi的概述
categories:
- python
tags:
- python   
---





`FastAPI`  +  `Starlette`  +  `Pydantic`



### FastAPI 概述

`py3.6+`  + `type hint`




> 基于 Starlette

`FastAPI` is actually a sub-class of `Starlette`

[Starlette](https://www.starlette.io/)


> 数据结构基于 Pydantic


[pydantic](https://pydantic-docs.helpmanual.io/)



> 下载  `pip install fastapi[all]` 

下载`fastapi`所有的依赖 



> 特性



    Fast:            Very high performance, on par with NodeJS and Go (thanks to Starlette and Pydantic). 
                     One of the fastest Python frameworks available.
    
    Fast to code:    Increase the speed to develop features by about 200% to 300% *.
    
    Fewer bugs:      Reduce about 40% of human (developer) induced errors. *
    
    Intuitive:       Great editor support. Completion everywhere. Less time debugging.
    
    Easy:            Designed to be easy to use and learn. Less time reading docs.
    
    Short:           Minimize code duplication. Multiple features from each parameter declaration. Fewer bugs.
    
    Robust:          Get production-ready code. With automatic interactive documentation.
    
    Standards-based: Based on (and fully compatible with) the open standards for APIs: OpenAPI (previously known as Swagger) and JSON Schema.
    
    可以自动生成文档       server addr/docs     基于  Swagger UI   https://github.com/swagger-api/swagger-ui
                         server addr/redoc    基于   ReDoc       https://github.com/Rebilly/ReDoc
                 
                         
    定义好参数的数据类型之后， 支持数据的验证，
    
                             数据的序列化,  
                             
                             自动生成带 交互的api文档


> Example 


demo.py

```python


# 同步服务

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "world"}



@app.get("/items/{item_id}")
def read_item(item_id: int, q:str = None):
    return {"item_id": item_id, "q": q}



# >> 启动

#  uvicorn  demo:app --port 8001 

# http://127.0.0.1:8001/items/1     {"item_id":1,"q":null}
# http://127.0.0.1:8001/            {"Hello":"world"}




# 异步服务



app1 = FastAPI()


@app1.get("/")
async def read_root():
    return {"Hello": "world"}



@app1.get("/items/{item_id}")
async def read_item(item_id: int, q:str = None):
    return {"item_id": item_id, "q": q}



# uvicorn demo:app1 --port 8001 --reload







# 可以自动生成文档       server addr/docs     基于  Swagger UI   https://github.com/swagger-api/swagger-ui
#                      server addr/redoc    基于   ReDoc       https://github.com/Rebilly/ReDoc



```



>  接收 PUT 的 request 数据， 并验证数据


```python

from pydantic import BaseModel


class Item(BaseModel):
    name: str
    price: int
    is_offer: bool = None



@app1.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_name": item.name, "item_id": item_id}



#  PUT 会以 json 格式 读取 request body 

#  在输入输出时候自动转换json格式   Convert from and to JSON automatically.



```



> summary  


    # 定义好参数的数据类型之后， 支持数据的验证，
    
                               数据的序列化,  
                             
                              自动生成带 交互的api文档
    
    #  在输入输出时候自动转换json格式   Convert from and to JSON automatically.


### 参数

<br>

####  1 path参数


path 参数会 传递给函数参数


>  不限制类型

```python

@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}

```

> 限制数据类型

```python


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}

```


> 使用 Enum, 来分配返回数据


```python

from enum import Enum 


class ModelName(str, Enum):
    alpha = "alpha"
    beta  = "beta"
    celta =  "celta"




@app1.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name == ModelName.alpha:
        return {"model_name": model_name, "msg": "alpha"}
    return {"error"}
```



> 数据验证 基于[Pydantic](https://pydantic-docs.helpmanual.io/)

```python
{
    "detail": [
        {
            "loc": [
                "path",
                "item_id"
            ],
            "msg": "value is not a valid integer",
            "type": "type_error.integer"
        }
    ]
}

```


#### 2 param 参数


>  布尔值的处理


    on. yes, true, True, 1

```python

@app1.get("/items/{item_id}")
async def read_item(item_id: str, q: str = None, short: bool = False):
    item = {"item_id": item_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item



# 注意这里的  short 可以是 on. yes, true, True, 1
# 都会被处理为 True

```

> 多path参数 和 param参数

```python


@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: str = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item


```


#### 3 请求体 request body


> 建议 
        
    To send data, you have to use one of: POST (the more common), PUT, DELETE or PATCH
    

发送数据使用  `pydantic `


```python

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None


app = FastAPI()


@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item, q: str = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result

```


> 不使用`pydantic`的对象， 自定义`body`数据


```python

from fastapi import Body 

@app1.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User, importance: int):

    # 这里的importance 是一个param参数
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    return results
    
 
 
@app1.put("/items/{item_id}")
async def update_item( item_id: int, item: Item, user: User, importance: int = Body(..., gt=2)):
    # 自定义body对象
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    return results   

    
    {
      "item": {
        "name": "string",
        "description": "string",
        "price": 0,
        "tax": 0
      },
      "user": {
        "username": "string",
        "full_name": "string"
      },
      "importance": 0
    }

```


### 参数验证

####  1 param 与 query validation


>  默认的类型检测 `param: str`

```python

@app.get("/items/")
async def read_items(q: str = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results

```


>  使用 `Query` 长度, 正则, 设置默认值

```python


from fastapi import Query

@app1.get("/items/")
async def read_items(q: str = Query(None, max_length=50, description="匹配信息", min_length=2, regex="^requery.*$")):
    # 这里的默认值为 None 表示param q可以为空
    # q: str = Query("fixedquery", min_length=3)  默认值为demo
    
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
    



```

> 使`param`变得`required` 和 `not required`

    Not required
    q: str = Query(None, min_length=3)
    
    Required  但是没有默认值
    q: str = Query(..., min_length=3)
    

> 使用`List` Query parameter list



```python
@app1.get("/items/")
async def read_items(q: list = Query(None)):
    query_items = {"q": q}
    return query_items_


from typing import List
@app1.get("/items/")
async def read_items(q: List[str] = Query(None)):
    query_items = {"q": q}
    return query_items


# http://localhost:8000/items/?q=foo&q=bar


```


```python


from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
    q: str = Query(
        ...,
        alias="item-query",    # request param 参数名, 后端用q接收
        title="Query string",
        description="Query string for the items to search in the database that have a good match",
        min_length=3,
        max_length=50,
        regex="^fixedquery$",
        deprecated=True,      # 标识api接口已经弃用
    )
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results

```

#### 2 path参数和验证


```python

from fastapi import FastAPI, Path

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    *,
    #  * 标识关键字参数
    #  Python won't do anything with that *, but it will know that all the following parameters should be called as keyword arguments (key-value pairs), also known as kwargs. Even if they don't have a default value.
    item_id: int = Path(..., title="The ID of the item to get", gt=0, le=1000),
    q: str,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results


```
    
    
    gt: greater than
    
    ge: greater than or equal
    
    lt: less than
    
    le: less than or equal


###  请求体 `Body` (`application/json`)



```python


from fastapi import Body, FastAPI
from pydantic import BaseModel

app1 = FastAPI()


class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None


class User(BaseModel):
    username: str
    full_name: str = None


@app1.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int,
    item: Item,
    user: User,
    importance: int = Body(..., gt=0),
    q: str = None
):
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    if q:
        results.update({"q": q})
    return results



# body 体


{
  "item": {
    "name": "string",
    "description": "string",
    "price": 0,
    "tax": 0
  },
  "user": {
    "username": "string",
    "full_name": "string"
  },
  "importance": 0
}



# ===========================

from fastapi import Body, FastAPI
from pydantic import BaseModel

app1 = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None


@app1.put("/items/{item_id}")
async def update_item(*, item_id: int, item: Item = Body(..., embed=False)):
    results = {"item_id": item_id, "item": item}
    return results


# body 体

{
  "name": "string",
  "description": "string",
  "price": 0,
  "tax": 0
}




@app1.put("/items/{item_id}")
async def update_item(*, item_id: int, item: Item = Body(..., embed=True)):
    results = {"item_id": item_id, "item": item}
    return results



# body 体

{
  "item": {
    "name": "string",
    "description": "string",
    "price": 0,
    "tax": 0
  }
}

```



#### 1 pydantic 中的 `BaseModel` 与 `Field`  

```python

from fastapi import Body, FastAPI
from pydantic import BaseModel, Field

app1 = FastAPI()

class Item(BaseModel):
    name: str
    description: str = Field(None, title="The description of the item", max_length=300)
    price: float = Field(..., gt=0, description="The price must be greater than zero")
    tax: float = None


@app1.put("/items/{item_id}")
async def update_item(*, item_id: int, item: Item = Body(..., embed=True, example={
            "name": "Foo",
            "description": "A very nice Item",
            "price": 35.4,
            "tax": 3.2
})):
    results = {"item_id": item_id, "item": item}
    return results


```


#### 2 嵌套的Model


```python


from typing import List, Set

from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()


class Image(BaseModel):
    url: HttpUrl
    name: str


class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None
    tags: Set[str] = []
    images: List[Image] = None


class Offer(BaseModel):
    name: str
    description: str = None
    price: float
    items: List[Item]


@app.post("/offers/")
async def create_offer(*, offer: Offer):
    return offer



# body 

{
  "name": "string",
  "description": "string",
  "price": 0,
  "items": [
    {
      "name": "string",
      "description": "string",
      "price": 0,
      "tax": 0,
      "tags": [
        "string"
      ],
      "images": [
        {
          "url": "string",
          "name": "string"
        }
      ]
    }
  ]
}



```


#### 3 其他可用数据类型


    
    UUID    一般用于item_id
    
    
    datetime.datetime 
    
             str in ISO 8601 format, like: 2008-09-15T15:53:00+05:00.
        
    datetime.date
    
             str in ISO 8601 format, like: 2008-09-15
             
    datetime.time
    
             str in ISO 8601 format, like: 14:23:55.003
             
    datetime.timedelta
    
    
    frozenset
    
    bytes
    
            str with binary "format"
            
    Decimal
    
    
    
    
```python

from datetime import datetime, time, timedelta
from uuid import UUID

from fastapi import Body, FastAPI

app1 = FastAPI()


@app1.put("/items/{item_id}")
async def read_items(
    item_id: UUID,
    start_datetime: datetime = Body(...),
    end_datetime: datetime = Body(...),
    repeat_at: time = Body(...),
    process_after: timedelta = Body(...),
):
    start_process = start_datetime + process_after
    duration = end_datetime - start_process
    return {
        "item_id": item_id,
        "start_datetime": start_datetime,
        "end_datetime": end_datetime,
        "repeat_at": repeat_at,
        "process_after": process_after,
        "start_process": start_process,
        "duration": duration,
    }




# body

{
  "start_datetime": "2020-02-21T06:32:24.223Z",
  "end_datetime": "2020-02-22T06:32:24.223Z",
  "repeat_at": "14:23:55",
  "process_after": 0
}

# response 


{
    "item_id": "2ef05342-5474-11ea-a2e3-2e728ce88125",
    "start_datetime": "2020-02-21T06:32:24.223000+00:00",
    "end_datetime": "2020-02-22T06:32:24.223000+00:00",
    "repeat_at": "14:23:55",
    "process_after": 0,
    "start_process": "2020-02-21T06:32:24.223000+00:00",
    "duration": 86400
}


```


###  从`表单 Form `中获取数据 `(application/x-www-form-urlencoded)`




    Info
        
        To use forms, first install python-multipart.
        
        E.g. pip install python-multipart.
        
        
```python

from fastapi import FastAPI, Form

app = FastAPI()


@app.post("/login/")
async def login(*, username: str = Form(...), password: str = Form(...)):
    return {"username": username}


# curl
#  -X POST "http://127.0.0.1:8001/login/"
#  -H "accept: application/json"
#  -H "Content-Type: application/x-www-form-urlencoded"
#  -d "username=user&password=aaaaa"



```


### 请求`File`文件 `(multipart/form-data)`

```python
from fastapi import FastAPI, File, UploadFile

app = FastAPI()


@app.post("/files/")
async def create_file(file: bytes = File(...)):
    return {"file_size": len(file)}

# curl -X POST "http://127.0.0.1:8001/files/" -H "accept: application/json" -H "Content-Type: multipart/form-data" -F "file=@6666666666.png;type=image/png"

@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile = File(...)):
    print(file.__dict__)
    # {'filename': '__init__.py', 'content_type': 'text/plain', 'file': <tempfile.SpooledTemporaryFile object at 0x000001779CDF1EB0>}
    return {"filename": file.filename}

# curl -X POST "http://127.0.0.1:8001/uploadfile/" -H "accept: application/json" -H "Content-Type: multipart/form-data" -F "file=@2.png;type=image/png"
```


使用 `UploadFile` 比 `bytes`的优点


    存储在内存中的文件大小上限为最大，超过此限制后，文件将存储在磁盘中
    
    意味着它可以很好地用于大型文件，例如图像，视频，大型二进制文件等，而不会占用所有内存
    
    You can get metadata from the uploaded file.
    
    It has a file-like async interface.
    
    It exposes an actual Python SpooledTemporaryFile object that you can pass directly to other libraries that expect a file-like object.
    
    {'filename': '__init__.py', 
    'content_type': 'text/plain', 
    'file': <tempfile.SpooledTemporaryFile object at 0x000001779CDF1EB0>
    }



`async methods`,` FastAPI` runs the file methods in a `threadpool` and awaits for them




> Multiple file uploads 

```python
from typing import List

from fastapi import FastAPI, File, UploadFile
from starlette.responses import HTMLResponse

app = FastAPI()


@app.post("/files/")
async def create_files(files: List[bytes] = File(...)):
    return {"file_sizes": [len(file) for file in files]}


@app.post("/uploadfiles/")
async def create_upload_files(files: List[UploadFile] = File(...)):
    return {"filenames": [file.filename for file in files]}


@app.get("/")
async def main():
    content = """
<body>
<form action="/files/" enctype="multipart/form-data" method="post">
<input name="files" type="file" multiple>
<input name="files" type="file" multiple>
<input type="submit">
</form>
<form action="/uploadfiles/" enctype="multipart/form-data" method="post">
<input name="files" type="file" multiple>
<input name="files" type="file" multiple>
<input type="submit">
</form>
</body>
    """
    return HTMLResponse(content=content)
```



#### 读取文件 


> `async`


    contents = await myfile.read()


> `sync`


    contents = myfile.file.read()
    
    #  UploadFile.file




```python
from fastapi import FastAPI, File, Form, UploadFile

app = FastAPI()


@app.post("/files/")
async def create_file(
    file: bytes = File(...), fileb: UploadFile = File(...), token: str = Form(...)
):
    return {
        "file_size": len(file),
        "token": token,
        "fileb_content_type": fileb.content_type,
    }


# curl -X POST "http://127.0.0.1:8001/files/"
#      -H "accept: application/json"
#      -H "Content-Type: multipart/form-data"
#      -F "token=token_a" 
#      -F "file=@2.png;type=image/png"
#      -F "fileb=@1.png;type=image/png"

```


###  `Cookie` 以及其他 `HEADER`

使用 `Cookie` 需要在 header中添加



```python


from fastapi import Cookie, FastAPI

app1 = FastAPI()


@app1.get("/items/")
async def read_items(*, session_id: str = Cookie(...)):
    return {"session_id": session_id}



# curl -X GET "http://127.0.0.1:8001/items/" -H "accept: application/json" -H "Cookie: session_id=3123123213wdsadsa"

{
  "session_id": "3123123213wdsadsa"
}

```


默认情况下 `Header` 会转换` underscore (_) `to `hyphen (-) `


```python

from fastapi import FastAPI, Header

app = FastAPI()


@app1.get("/items/")
async def read_items(*, user_agent: str = Header(None)):
    return {"user_agent": user_agent}


{
  "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36"
}



@app.get("/items/")
async def read_items(*, strange_header: str = Header(None, convert_underscores=False)):
    return {"strange_header": strange_header}

{
  "strange_header": "dasdsa"
}




from typing import List

from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(x_token: List[str] = Header(None)):
    return {"X-Token values": x_token}
    
    
#X-Token: foo
#X-Token: bar   


    {
    "X-Token values": [
        "bar",
        "foo"
    ]
}


```


### 返回值 Response


    @app.get()
    @app.post()
    @app.put()
    @app.delete()


<br>


#### 使用`response_model` 渲染，序列化


```python

from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app1 = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: str = None



# 不输出 pwd
class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: str = None


@app1.post("/user/", response_model=UserOut) # 使用 outUser
async def create_user(*, user: UserIn):
    return user


```

> `response_model_include` and `response_model_exclude`  控制渲染的字段

```python

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = 10.5


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The Bar fighters", "price": 62, "tax": 20.2},
    "baz": {
        "name": "Baz",
        "description": "There goes my baz",
        "price": 50.2,
        "tax": 10.5,
    },
}


@app.get(
    "/items/{item_id}/name",
    response_model=Item,
    response_model_include=["name", "description"],
)
async def read_item_name(item_id: str):
    return items[item_id]


@app.get("/items/{item_id}/public", response_model=Item, response_model_exclude=["tax"])
async def read_item_public_data(item_id: str):
    return items[item_id]

```


#### `status_code`



    200   Successful
    
    201   Created
    
    300   
    
    400   Client error
    
    500   server errors
    


> 可以直接使用 `starlette.status`


```python


from fastapi import FastAPI
from starlette.status import HTTP_201_CREATED

app = FastAPI()


@app.post("/items/", status_code=HTTP_201_CREATED)
async def create_item(name: str):
    return {"name": name}

```


#### 使用 `jsonable_encoder`转换成 可以序列化为json的dict



```python

from datetime import datetime

from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

fake_db = {}


class Item(BaseModel):
    title: str
    timestamp: datetime
    description: str = None


app = FastAPI()


@app.put("/items/{id}")
def update_item(id: str, item: Item):
    json_compatible_item_data = jsonable_encoder(item)
    
    # json_str = json.dumps(json_compatible_item_data)
    
    fake_db[id] = json_compatible_item_data


```


### `client` 异常处理 



    status codes in the 400 range mean that there was an error from the client.

应用场景

    
    The client doesn't have enough privileges for that operation.
    操作权限
    
    The client doesn't have access to that resource.
    资源数据权限
    
    The item the client was trying to access doesn't exist.
    不存在的访问路径
    
    etc.


#### `HTTPException`



```python

# not found 

from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}


@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}


```


> 增加 `response header`


```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}


@app.get("/items-header/{item_id}")
async def read_item_header(item_id: str):
    if item_id not in items:
        raise HTTPException(
            status_code=404,
            detail="Item not found",
            headers={"X-Error": "There goes my error"},
        )
    return {"item": items[item_id]}


# content-length: 27
# content-type: application/json
# date: Fri, 21 Feb 2020 10:17:34 GMT
# server: uvicorn
# x-error: There goes my error
```


#### 改写`HTTPException`


```python
from fastapi import FastAPI, HTTPException
from starlette.responses import PlainTextResponse

app = FastAPI()


@app.exception_handler(HTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```


#### 用`exception_handler`自定义异常处理


```python

from fastapi import FastAPI
from starlette.requests import Request
from starlette.responses import JSONResponse


class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name


app = FastAPI()


@app.exception_handler(UnicornException)
async def my_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )


@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}

```


### 关于`路由 path`的一些概念 (大部分配合docs显示)



#### `tags 路由分类`


```python
from typing import Set

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None
    tags: Set[str] = []


@app.post("/items/", response_model=Item, tags=["items"])
async def create_item(*, item: Item):
    return item


@app.get("/items/", tags=["items"])
async def read_items():
    return [{"name": "Foo", "price": 42}]


@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]



# items

#    GET /items​/  Read Items
    
#    POST /items​/ Create Item



# users

#   GET   users​/  Read Users

```



#### `docstring` 文档解释信息


```python
from typing import Set

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None
    tags: Set[str] = []


@app.post("/items/", response_model=Item, summary="Create an item")
async def create_item(*, item: Item):
    """
    Create an item with all the information:

    - **name**: each item must have a name
    - **description**: 描述
    - **price**: required
    - **tax**: if the item doesn't have tax, you can omit this
    - **tags**: a set of unique tag strings for this item
    """
    return item


```


#### `Deprecate` 显示弃用  

```python

from fastapi import FastAPI

app = FastAPI()


@app.get("/items/", tags=["items"])
async def read_items():
    return [{"name": "Foo", "price": 42}]


@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]


@app.get("/elements/", tags=["items"], deprecated=True)
async def read_elements():
    return [{"item_id": "Foo"}]
```