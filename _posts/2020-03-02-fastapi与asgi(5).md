---
title: fastapi与asgi(5)                                              
description: fastapi 中 的数据库相关操作
categories:
- python
tags:
- python   
---


以 `SQLAlchemy`为例， 如何进行`关系型数据库`操作

[>> sqlalchemy](https://www.sqlalchemy.org/)


    PostgreSQL
    MySQL
    SQLite
    Oracle
    Microsoft SQL Server, etc.


项目的总体结构:
      .
    └── sql_app
        ├── __init__.py
        ├── crud.py
        ├── database.py
        ├── main.py
        ├── models.py
        └── schemas.py  


#### `sql_app/database.py` `SQLAlchemy` 数据库连接部分



```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
# SQLALCHEMY_DATABASE_URL = "postgresql://user:password@postgresserver/db"
# 创建数据库连接URI

engine = create_engine(
    SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False}
)
#  create a SQLAlchemy "engine"
# connect_args={"check_same_thread": False} is needed only for SQLite. It's not needed for other databases.


SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
# 数据库 session once we create an instance of the SessionLocal class, this instance will be the actual database session


Base = declarative_base()
# 数据model的基础类 database models or classes (the ORM models)

```

#### `sql_app/models.py` 数据库models

```python
from sqlalchemy import Boolean, Column, ForeignKey, Integer, String
from sqlalchemy.orm import relationship

from .database import Base

# Create SQLAlchemy models from the Base class¶


class User(Base):
    __tablename__ = "users"
    # __tablename__ attribute tells SQLAlchemy the name of the table to use in the database for each of these models.

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)

    items = relationship("Item", back_populates="owner")


class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    description = Column(String, index=True)
    owner_id = Column(Integer, ForeignKey("users.id"))

    owner = relationship("User", back_populates="items")

```

#### `sql_app/schemas.py`  `Fastapi`中的 `Pydantic` models 数据类 

    for reading / returning
    
    读取和返回使用
    
    

```python
from typing import List

from pydantic import BaseModel


class ItemBase(BaseModel):
    title: str
    description: str = None


class ItemCreate(ItemBase):
    pass


class Item(ItemBase):
    id: int
    owner_id: int

    class Config:
        orm_mode = True
        #  Pydantic的orm_mode将告诉Pydantic模型读取数据，即使它不是字典，而是ORM模型（或具有属性的任何其他任意对象）。
class UserBase(BaseModel):
    email: str


class UserCreate(UserBase):
    password: str


class User(UserBase):
    id: int
    is_active: bool
    items: List[Item] = []

    class Config:
        orm_mode = True
```

`SQLAlchemy style`  vs `Pydantic style`

    # SQLAlchemy style
    name = Column(String)
    
    # Pydantic style
    name: str




#### `sql_app/crud.py`  数据的增删改查操作


>注意这里 `SQLAlchemy` 不支持 `async`的方式， 所以函数定义为 `def`


```python
from sqlalchemy.orm import Session

from . import models, schemas


def get_user(db: Session, user_id: int):
    return db.query(models.User).filter(models.User.id == user_id).first()


def get_user_by_email(db: Session, email: str):
    return db.query(models.User).filter(models.User.email == email).first()


def get_users(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.User).offset(skip).limit(limit).all()


def create_user(db: Session, user: schemas.UserCreate):
    fake_hashed_password = user.password + "notreallyhashed"
    db_user = models.User(email=user.email, hashed_password=fake_hashed_password)
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user


def get_items(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.Item).offset(skip).limit(limit).all()


def create_user_item(db: Session, item: schemas.ItemCreate, user_id: int):
    db_item = models.Item(**item.dict(), owner_id=user_id)
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item

```

#### `sql_app/main.py` main主程序

> 1 需要先进行数据表的初始化
    
通常，您可能会使用[`Alembic`](https://alembic.sqlalchemy.org/en/latest/)初始化数据库（创建表等）。

> 2 每个请求有一个独立的数据库会话 `db session`

我们需要每个请求有一个独立的数据库会话/连接（SessionLocal），在所有请求中使用相同的会话，然后在请求完成后将其关闭。 然后将为下一个请求创建一个新会话

    可以通过 中间件 或者 dependency 实现
    
    中间件：        会使得每个请求都建立可能不需要的db连接
    dependency：   更方便
    
   
```python
@app.middleware("http")
async def db_session_middleware(request: Request, call_next):
    response = Response("Internal server error", status_code=500)
    try:
        request.state.db = SessionLocal()
        response = await call_next(request)
    finally:
        request.state.db.close()
    return response


# Dependency
def get_db(request: Request):
    return request.state.db

```  

    
> 3 视图函数路径的参数 `db`的`type hint` 
    
    The parameter `db` is actually of type `SessionLocal`, 
    but this class (created with sessionmaker()) is a "proxy" of a SQLAlchemy Session,
    so, the editor doesn't really know what methods are provided.


    by declaring the type as Session, the editor now can know the available methods (.add(), .query(), .commit(), etc) 
    and can provide better support (like completion). 
    The type declaration doesn't affect the actual object.


> 4 由于`SQLAlchemy`不支持`async`， 视图函数使用 `def`


```python
from typing import List

from fastapi import Depends, FastAPI, HTTPException
from sqlalchemy.orm import Session

from . import crud, models, schemas
from .database import SessionLocal, engine

models.Base.metadata.create_all(bind=engine)
# Create the database tables

app = FastAPI()


# Dependency
def get_db():
    try:
        db = SessionLocal()
        yield db
    finally:
        db.close()


@app.post("/users/", response_model=schemas.User)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    # 这里的db 实际是 SessionLocal 实例， 但是也是 Session， 这里直接type hint Session 方便代码补全
    db_user = crud.get_user_by_email(db, email=user.email)
    if db_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    return crud.create_user(db=db, user=user)


@app.get("/users/", response_model=List[schemas.User])
def read_users(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    users = crud.get_users(db, skip=skip, limit=limit)
    return users


@app.get("/users/{user_id}", response_model=schemas.User)
def read_user(user_id: int, db: Session = Depends(get_db)):
    db_user = crud.get_user(db, user_id=user_id)
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return db_user


@app.post("/users/{user_id}/items/", response_model=schemas.Item)
def create_item_for_user(
    user_id: int, item: schemas.ItemCreate, db: Session = Depends(get_db)
):
    return crud.create_user_item(db=db, item=item, user_id=user_id)


@app.get("/items/", response_model=List[schemas.Item])
def read_items(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    items = crud.get_items(db, skip=skip, limit=limit)
    return items
```


#### 关于`request.state`


`request.state` 是每个`Request对象`的属性。 它可以存储附加到请求本身的任意对象，例如 `数据库会话 db session`

[> more request.state ](https://www.starlette.io/requests/#other-state)