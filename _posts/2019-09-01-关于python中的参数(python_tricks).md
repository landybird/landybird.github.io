---
title: 关于python中的参数 `args`, `**kwargs`
description: 关于python中的参数
categories:
- python
tags:
- python tricks
---

<br>
 
- 位置参数 `*args` 与 关键字参数 `**kwargs` 可以增加灵活性

```python
def foo(required, *args, **kwargs):
    print(required)
    if args:
        print(args)
    if kwargs:
        print(kwargs)
```

上面这个函数中 有`required`必须传入的参数， 还可以接收 多个`位置参数`和 `关键字参数`

其中 `args`因为 `*`前缀 的存在， 会把位置参数收集到一个`tuple`元組中
其中 `kwargs`因为 `**`前缀 的存在， 会把位置参数收集到一个`dict`字典中

当然 `args` 和 `kwargs`可以为空

```python

def foo(required, *args, **kwargs):
    print(required)
    if args:
        print(args)
    if kwargs:
        print(kwargs)


foo()
TypeError:
"foo() missing 1 required positional arg: 'required'"

foo('hello')
hello

foo('hello', 1, 2, 3)
hello
(1, 2, 3)


foo('hello', 1, 2, 3, key1='value', key2=999)
hello
(1, 2, 3)
{'key1': 'value', 'key2': 999}
```

- 将接收的 `位置参数` 和 `关键字参数` 传递给另一个函数


```python

def bar(x, *args, **kw):
    pass


def foo(x, *args, **kwargs):
    kwargs['name'] = 'Alice'
    new_args = args + ('extra', )
    bar(x, *new_args, **kwargs)
    

class Car:
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage
        
class AlwaysBlueCar(Car):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.color = 'blue'
        
AlwaysBlueCar('green', 48392).color
'blue'

```

- `unpacking` 使用 `*`, `**` unpack 一個字典

```python


def print_vector(x, y, z):
    print(x, y, z)
    print('<%s, %s, %s>' % (x, y, z))

dict_vec = {'y': 0, 'z': 1, 'x': 1}
print_vector(*dict_vec)
y z x
<y, z, x>

print_vector(**dict_vec)
1 0 1
<1, 0, 1>
``` 

- 沒有返回值

```python

def foo1(value):
    if value:
        return value
    else:
        return None
        
def foo2(value):
    """Bare return statement implies `return None`"""
    if value:
        return value
    else:
        return
        
def foo3(value):
    """Missing return statement implies `return None`"""
    if value:
        return value

# 在一個函數沒有返回值的时候，不使用return 语句 

```



