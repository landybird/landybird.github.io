---
title: python中装饰器与wraps
description: python中装饰器与wraps
categories:
- python
tags:
- python基础
---

<br>


# python中装饰器与wraps


先来看一个装饰器（丢失信息）

```python


    def log(func):
        def inner(self,*args):
            try:
                logging.info(" server: %s" % (func.__name__,))  // 或丢失被装饰的函数名信息
                return func(self, *args)
            except Exception as e:
                error_info = traceback.format_exc()
                logging.error(error_info)
                if isinstance(e,ServerException):
                    raise e
                else:
                    raise ServerException(500, str(e))
            finally:
                logging.info("server: %s" % ( func.__name__,))
        return inner

```

而 `functools.wraps `则可以将原函数对象的指定属性复制给包装函数对象,
 
 默认有 __module__、__name__、__doc__,或者通过参数选择
 
 
 ```python
 from functools import wraps
 def logged(func):
     @wraps(func)
     def with_logging(*args, **kwargs):
         print func.__name__ + " was called"
         return func(*args, **kwargs)
     return with_logging
  
 @logged
 def f(x):
    """does some math"""
    return x + x * x
  
 print f.__name__  # prints 'f'
 print f.__doc__   # prints 'does some math'
 
 ```
