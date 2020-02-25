---
title: fastapi与asgi(3)                                              
description: fastapi中 Security问题
categories:
- python
tags:
- python   
---


In `many frameworks` and systems just handling `security` and `authentication` takes a big amount of effort and code (in many cases it can be `50% or more` of all the code written).

`FastAPI` provides several tools to help you deal with Security `easily`, `rapidly`, in a standard way




### 几个概念 



#### `OAuth2`

`OAuth2`是一个规范，定义了几种处理`身份验证`和`授权`的方式。 

它是一个相当广泛的规范，涵盖了几个复杂的用例。
 
它包括使用“第三方”进行身份验证的方法。
 
OAuth2没有指定如何加密通信，它希望您使用HTTPS为您的应用程序提供服务。
 
    这就是所有带有“使用Facebook，Google，Twitter，GitHub登录”的系统的基础。
    

#### `OAuth1`


与OAuth2完全不同，并且`更为复杂`，因为它`直接包含有关如何加密通信的规范`。
 
它现在不是很流行或使用。

    OAuth2没有指定如何加密通信，它希望您使用HTTPS为您的应用程序提供服务。
    
    
#### `OpenID Connect`


`OpenID Connect`是OAuth 2.0（授權框架）之上的`身份驗證層`

它只是扩展了OAuth2，以指定一些在OAuth2中相对模糊的内容，以尝试使其更具互操作性。


    例如，Google登录使用OpenID Connect（在下面使用OAuth2）。 
    
    但是，Facebook登录不支持OpenID Connect。 它具有自己的OAuth2风格。


#### `OpenID (not "OpenID Connect")`

`OpenID 规范`

试图解决与OpenID Connect相同的问题，但`不是基于OAuth2`。 

因此，它是一个完整的附加系统。 它现在不是很流行或使用



#### `OpenAPI規範`

`OpenAPI規範`（最初稱為`Swagger規範`）是一種`機器可讀接口文件`的規範，用於`描述，生成，使用和可視化RESTful Web服務`。

它最初是`Swagger框架的一部分`，但在2016年成為一個獨立的項目，由Linux基金會的開源協作項目OpenAPI Initiative監督



> `FastAPI` is based on `OpenAPI`.

OpenAPI 具有定义多个安全 “方案”的方法:  `multiple security "schemes"`.



`apiKey`: an application specific key that can come from:

    A query parameter.
    
    A header.
    
    A cookie.


`http`: standard HTTP authentication systems, including:

    bearer: a header Authorization with a value of Bearer plus a token. This is inherited from OAuth2.
    
    HTTP Basic authentication.
    
    HTTP Digest, etc.


`oauth2`: all the OAuth2 ways to handle security (called "flows").

    Several of these flows are appropriate for building an OAuth 2.0 authentication provider (like Google, Facebook, Twitter, GitHub, etc):
    其中一些流程适合构建OAuth 2.0身份验证提供程序（例如Google，Facebook，Twitter，GitHub等）

    implicit
    
    clientCredentials
    
    authorizationCode
        
    password: some next chapters will cover examples of this.
    

`openIdConnect`: has a way to define how to discover OAuth2 authentication data automatically.



### 验证身份信息


```python


# use OAuth2, 
# with the Password flow, 
# using a Bearer token

from fastapi.security import OAuth2PasswordBearer
from fastapi import Depends, FastAPI

app = FastAPI()

oauth2_sceme = OAuth2PasswordBearer("/token")

@app.get("/items/")
async def read_items(token: str = Depends(oauth2_sceme)):
    return {"token": token}

```



#### `password flow` password flow 的完整流程
 
 
to handle `security` and `authentication`


> `OAuth2PasswordBearer`
 
`OAuth2PasswordBearer` is a class that we create passing a `parameter of the URL` in where the client (the frontend running in the user's browser) can
use to send the username and password and `get a token`.


```python
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/token")
```


> scope 

In OAuth2 a `"scope"` is just a string that declares a `specific permission required`.



> demo 实例


```python

from hashlib import md5

from fastapi import Depends, FastAPI, HTTPException
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/token")


# 数据库数据

fake_users_db = {
    "jim": {
        "username": "jim",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_pwd": "fakehashed+pwd",
        "disabled": False,
    },
    "jimmy": {
        "username": "jimmy",
        "full_name": "Alice Wonderson",
        "email": "alice@example.com",
        "hashed_pwd": "fakehashed+pwd",
        "disabled": True,
    },
}



class User(BaseModel):
    username: str
    email: str = None
    full_name: str = None
    disabled: bool = None


class UserInDB(User):
    hashed_pwd: str



def gen_token(username: str):
    md = md5(username.encode('utf-8')).hexdigest()
    return md


def fake_hash_password(password: str):
    return "fakehashed+" + password


def get_user(token):
    for _, user_dict in fake_users_db.items():
        if user_dict.get("token") == token:
            return UserInDB(**user_dict)


async def get_current_user(token: str = Depends(oauth2_scheme)):
    user = get_user(token)
    if not user:
        raise HTTPException(
            status_code=401,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
    return user


async def get_active_user(user: User = Depends(get_current_user)):
    if user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return user

# 使用 login 生成的 token 获取数据
@app.get("/users/me")
async def read_users_me(current_user: User = Depends(get_active_user)):
    return current_user



# 生成token
@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    # print(form_data.__dict__)
    # {'grant_type': 'password', 'username': 'jim', 'password': 'pwd', 'scopes': [], 'client_id': None,
    #  'client_secret': None}

    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detail="Incorrect username or password")

    user = UserInDB(**user_dict)
    hashed_pwd = fake_hash_password(form_data.password)
    if not hashed_pwd == user.hashed_pwd:
        raise HTTPException(status_code=400, detail="Incorrect username or password")


    token = gen_token(user.username)
    user_dict["token"] = token

    return {"access_token": token, "token_type": "bearer"}
    # By the spec, you should return a JSON with 
    
    # an access_token  
    # a token_type

```



####  `使用 正式的JWT token 和 hashed pwd`



>  `JWT¶` JSON Web Tokens

这是将JSON对象编码为长而密集的字符串且没有空格的标准

    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

它没有加密，因此任何人都可以从内容中恢复信息。

但这是签名的， 因此，当您收到发出的令牌时，可以验证您确实发出了令牌。

一般需要设置过期时间


[see how JWT work](https://jwt.io/)




- 下载 `PyJWT`


    
    pip install pyjwt
    


```python

import jwt


# openssl rand -hex 32
SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"

access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
access_token = create_access_token(
    data={"username": user.username}, expires_delta=access_token_expires
    )



def create_access_token(*, data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
        
    to_encode.update({"exp": expire})
    #增加过期时间
    
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    # 生成 Token
    
    return encoded_jwt
    
    
# 解码
payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
username: str = payload.get("username")
...


```









- 下载 `passlib¶`  



用户 可以在`不同的框架`中使用相同的身份信息


支持许多安全的哈希算法 

推荐的算法是`“ Bcrypt”`



    pip install passlib[bcrypt]




```python

from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)


def get_password_hash(password):
    return pwd_context.hash(password)

```







> 完整的例子


```python

from datetime import datetime, timedelta

import jwt
from fastapi import Depends, FastAPI, HTTPException
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jwt import PyJWTError
from passlib.context import CryptContext
from pydantic import BaseModel
from starlette.status import HTTP_401_UNAUTHORIZED

# to get a string like this run:
# openssl rand -hex 32
SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30


fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW", # secret
        "disabled": False,
    }
}


class Token(BaseModel):
    access_token: str
    token_type: str


class TokenData(BaseModel):
    username: str = None


class User(BaseModel):
    username: str
    email: str = None
    full_name: str = None
    disabled: bool = None


class UserInDB(User):
    hashed_password: str


pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/token")

app = FastAPI()


def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)


def get_password_hash(password):
    return pwd_context.hash(password)


def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)


def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user


def create_access_token(*, data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt


async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except PyJWTError:
        raise credentials_exception
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user


async def get_current_active_user(current_user: User = Depends(get_current_user)):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user


@app.post("/token", response_model=Token)
async def login_for_access_token(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}


@app.get("/users/me/", response_model=User)
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    return current_user

# curl -X GET "http://127.0.0.1:8002/users/me/" -H "accept: application/json" -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJqb2huZG9lIiwiZXhwIjoxNTgyNjI2Njg4fQ.-NwF7iuS8mmu-7X0_y-mRnvcyxAz1Bgw-AN6Z81tLFA"

@app.get("/users/me/items/")
async def read_own_items(current_user: User = Depends(get_current_active_user)):
    return [{"item_id": "Foo", "owner": current_user.username}]

```
