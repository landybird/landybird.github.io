---
title: 上下文管理与contextlib
description: 上下文管理与contextlib
categories:
- python
tags:
- python tricks
---

<br>

- 常见的上下文管理 open 文件操作

```python

with open('hello.txt', 'w') as f:
    f.write('hello, world!')


f = open('hello.txt', 'w')
try:
    f.write('hello, world')
finally:
    f.close()
```

- 常用的上下文管理  threading.Lock 锁操作

```python

some_lock = threading.Lock()

# Harmful:
some_lock.acquire()
try:
    # Do something...
finally:
    some_lock.release()
    
# Better:
with some_lock:
    # Do something...

```

- context manager 上下文管理
 
如果想实现上下文管理机制， 需要做的只是 `__enter__` and `__exit__` 方法
或者是使用 `contextmanager 装饰器`


```python

class ManagedFile:
    def __init__(self, name):
        self.name = name
    def __enter__(self):
        self.file = open(self.name, 'w')
        return self.file
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()


# 使用装饰器

from contextlib import contextmanager

@contextmanager
def managed_file(name):
    try:
        f = open(name, 'w')
        yield f
    finally:
        f.close()

```
 
