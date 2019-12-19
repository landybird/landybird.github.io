---
title: python中字典的操作
description: python中字典的操作
categories:
- python
tags:
- python tricks
---

<br>

- `get` 字典的默认值

```python

d = {
    "a": 1,
    "b": 2,
    "c": 3
}

d.get("a")

d.get("d", 4)

```

- 字典排序 `operator.itemgetter(1)`

```python

xs = {'a': 4, 'c': 2, 'b': 3, 'd': 1}

s = sorted(xs.items(), reverse=True, key=lambda x:x[0])

d = dict(s)
print(d)  # {'d': 1, 'c': 2, 'b': 3, 'a': 4}


import operator

xs = {'a': 4, 'c': 2, 'b': 3, 'd': 1}

s = sorted(xs.items(), reverse=True, key=operator.itemgetter(1))

d = dict(s)
print(d)  # {'a': 4, 'b': 3, 'c': 2, 'd': 1}
```

#### 使用字典来实现 `switch .. case ..`

python中没有 `switch-case`， 所以需要用 `if-else`语句代替


```python

def dispatch_if(operator, x, y):
... if operator == 'add':
...     return x + y
... elif operator == 'sub':
...     return x - y
... elif operator == 'mul':
...     return x * y
... elif operator == 'div':
...     return x / y


def add(x, y):
    return x + y

def sub(x, y):
    return x - y

def mul(x, y):
    return x * y

def div(x, y):
    return x / y


def dispatch(operator, x, y):
    return {
        'add': add,
        'sub': sub,
        'mul': mul,
        'div': div,
    }.get(
        operator, None
    )(x, y)




# 利用lambda函数

def dispatch_dict(operator, x, y):
... return {
... 'add': lambda: x + y,
... 'sub': lambda: x - y,
... 'mul': lambda: x * y,
... 'div': lambda: x / y,
... }.get(operator, lambda: None)()

```

- 来看一个字典的结果 `{True: 'yes', 1: 'no', 1.0: 'maybe'}`

```python
{True: 'yes', 1: 'no', 1.0: 'maybe'} 

#{True: 'maybe'} # bool值是int的子集
```

相当于
```python
>>> xs = dict()
>>> xs[True] = 'yes'
>>> xs[1] = 'no'
>>> xs[1.0] = 'maybe'

```

因为三个键的值相等 `__eq__`， 且hash `__hash__`也相同， 所以第一个键`True`不再更新，而是更新了最后的值`maybe`



保证值相等 `__eq__`

```python

class AlwaysEquals:
    def __eq__(self, other):
        return True

    def __hash__(self):
        return id(self)


a = AlwaysEquals()
b = AlwaysEquals()
print(a==b)   # True

d = {a: 'yes', b: 'no'}
print(d)      # {<__main__.AlwaysEquals object at 0x0000021955B97240>: 'yes', <__main__.AlwaysEquals object at 0x0000021955B97278>: 'no'}
              # 并没有更新
```

    
    In CPython, id() returns the address of the object in memory, which
    is guaranteed to be unique.


保证hash相等 `__hash__`

```python

class SameHash:
  
    def __hash__(self):
        return 1


a = SameHash()
b = SameHash()
print(a==b)   # False

d = {a: 'yes', b: 'no'}
print(d)      # {<__main__.SameHash object at 0x0000014FC2817240>: 'yes', <__main__.SameHash object at 0x0000014FC282D6D8>: 'no'}
              # 并没有更新
```


同时保证 值, hash相等


```python
class SameHashValue:

    def __hash__(self):
        return 1
    
    def __eq__(self, other):
        return True


a = SameHashValue()
b = SameHashValue()
print(a == b)  # True

d = {a: 'yes', b: 'no'}
print(d)       # {<__main__.SameHash object at 0x000002A3D3697240>: 'no'}

```


- 有好几种方法可以 合并一个字典

```python

# 1 

>>> xs = {'a': 1, 'b': 2}
>>> ys = {'b': 3, 'c': 4}

>>> zs = {}
>>> zs.update(xs)
>>> zs.update(ys)


# 2

def update(dict1, dict2):
    for key, value in dict2.items():
        dict1[key] = value


# 3

xs = {'a': 1, 'b': 2}
ys = {'b': 3, 'c': 4}
zs = dict(xs, **ys)


# 4 

xs = {'a': 1, 'b': 2}
ys = {'b': 3, 'c': 4}
zs = dict(**xs, **ys)

```

-  用更好的格式 输出一个字典`（转换成字符串）`


<1> 使用 `str` 

```python
mapping = {'a': 23, 'b': 42, 'c': 0xc0ffee}
print(str(mapping))
```

<2> 使用 json

`函数, 集合之类的不可序列化`  

    键必须是string,  primitive data type



```python
import json
json_r = json.dumps(mapping, indent=4, sort_keys=True)
print(json_r)


{
    "a": 23,
    "b": 42,
    "c": 12648430
}
```

<3> 使用 `pprint`

```python
data = (
    "this is a string", [1, 2, 3, 4], ("more tuples",
    1.0, 2.3, 4.5), "this is yet another string"
    )
import pprint

pprint.pprint(data, indent=5)
(    'this is a string',
     [1, 2, 3, 4],
     ('more tuples', 1.0, 2.3, 4.5),
     'this is yet another string')

```