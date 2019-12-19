---
title: ThreadPoolExecutor的map传递参数
description: ThreadPoolExecutor的map传递参数
categories:
- python
tags:
- python基础
---

<br>

# ThreadPoolExecutor的map传递参数

<br>


通常使用ThreadPoolExecutor的情况是

```python
from concurrent.futures import ThreadPoolExecutor


def foo(item):
    # do something
    
   
with ThreadPoolExecutor(max_workers=20) as executor:
    executor.map(foo, item_list)

```

如果在每次执行`foo`函数的时候需要传递固定的参数

```python

from concurrent.futures import ThreadPoolExecutor


def foo(param1,param2,item):
    # do something

# 需要使用 partial 函数

from functools import partial

with ThreadPoolExecutor(max_workers=20) as executor:
    executor.map(partial(param1,param2), item_list)

```