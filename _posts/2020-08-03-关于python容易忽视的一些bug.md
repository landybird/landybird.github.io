---
title: 一些容易忽视的bug      
description: 不成熟的小建议   
categories:
- python
tags:
- python   
---
    
#### 对于常规的异常, 不要单独使用`except`关键字，`使用 except Exception`

- 1 `except` 会捕捉 `memory errors`, `interrupts`, `system exit` ..


    尽量未知的异常暴露出来, 然后对应处理
    
    

- 2 `except Exception` 可捕捉所有常规的异常 (`base type for all "Regular" exceptions`)
    
    
    处理常规的异常


- 3 使用`finally` 或者`上下文管理 context managers` 来处理 `资源连接的关闭`


```python
import time

try:
    time.sleep(10)
    1/0

except Exception as e:
    print(e)
    # division by zero

except:
    import traceback as tb
    print(tb.format_exc()[:])
    # KeyboardInter

```


#### 使用`hasattr（x, "__call__"）`判断是否可以被调用不可靠, 应该使用 `callable()`

```python
class X:
    def __getattr__(self, name):
        return name

i = X()

print(callable(i)) # False
print(hasattr(i, "__call__")) # True
```


####  不要使用 `assert False`, 应该`raise AssertionError()`
 
`python -O`  removes these calls.


#### [to be continued ...]()
