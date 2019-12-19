---
title: python中的装饰器
description: python中的装饰器
categories:
- python
tags:
- python tricks
---

<br>

核心作用是 扩展 修改 一个可调用的对象(callable `function`, `method`, `class`), 而不是永久性的改变他

    
    • logging        日志
    • enforcing access control and authentication  权限
    • instrumentation and timing functions  
    • rate-limiting  频率限制
    • caching, and more  缓存等


- 装饰器基础  `syntactic sugar` `@ 语法糖`

- 装饰器改变函数行为

```python
def uppercase(func):
    def wrapper():
        original_result = func()
        modified_result = original_result.upper()
        return modified_result
    return wrapper


@uppercase
def greet():
    return 'Hello!'
    
greet()
'HELLO!'


```

- 多个装饰器

```python

def strong(func):
    def wrapper():
        return '<strong>' + func() + '</strong>'
    return wrapper
    
def emphasis(func):
    def wrapper():
        return '<em>' + func() + '</em>'
    return wrapper


@strong
@emphasis
def greet():
    return 'Hello!'
    
greet()
'<strong><em>Hello!</em></strong>'  


decorated_greet = strong(emphasis(greet))


```

- 装饰带有参数的 函数, `*args, **kw`

```python

def trace(func):
    def wrapper(*args, **kwargs):
        print(f'TRACE: calling {func.__name__}() '
            f'with {args}, {kwargs}')
        original_result = func(*args, **kwargs)
        print(f'TRACE: {func.__name__}() '
            f'returned {original_result!r}')
        return original_result
    return wrapper

@trace
def say(name, line):
    return f'{name}: {line}'


say('Jane', 'Hello, World')
'TRACE: calling say() with ("Jane", "Hello, World"), {}'
'TRACE: say() returned "Jane: Hello, World"'
'Jane: Hello, World'

```

- 让装饰器有迹可循 `debuggable`

```python
def greet():
"""Return a friendly greeting."""
    return 'Hello!'
    
decorated_greet = uppercase(greet)

greet.__name__
    'greet'
greet.__doc__
    'Return a friendly greeting.'

decorated_greet.__name__
'wrapper'

decorated_greet.__doc__
None


# ====================


import functools

def uppercase(func):
    @functools.wraps(func)
    def wrapper():
        return func().upper()
    return wrapper

@uppercase
def greet():
"""Return a friendly greeting."""
    return 'Hello!'
    
>>> greet.__name__
'greet'

>>> greet.__doc__
'Return a friendly greeting.'



```

推荐在装饰器中使用 `@functools.wraps` 保证函数的原始信息