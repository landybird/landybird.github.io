---
title: python中的循环遍历
description: python中的循环遍历
categories:
- python
tags:
- python tricks
---

<br>

#### 实现一个`pythonic的循环`

先来看一个 `C, Java 风格`的 循环实现

```c++

my_items = ['a', 'b', 'c']

i = 0
while i < len(my_items):
    print(my_items[i])
    i += 1
   
```

- `range() 函数` 占用很小的内存, 可以自动的生成遍历元素的索引
    
`py2` with `xrange()`
    
    Using the range() built-in, I can generate the indexes automatically


```python
for i in range(len(my_items)):
    print(my_items[i])
    
# 低效率的, 要避免使用 range(len(..))
```

在range()中可以指定 `step步长` , `range(1, 10, 2)`





- `enumerate()函数`  高效的获取索引和元素

```python
for i, item in enumerate(my_items):
    print(f'{i}: {item}')
    
0: a
1: b
2: c

```

这里的 range(), enumerate() 的返回值都是一个 `Iteratble` 可迭代对象

```python
from collections import Iterable, Iterator

e = enumerate(range(0,10))
r = range(0, 1)

print(isinstance(e, Iterable))  # True
print(isinstance(e, Iterator))  # True

print(isinstance(r, Iterable))  # True

```


#### 理解python中的 推导式 `Comprehensions`

首先要明确 推导式的使用， 是为了更好的 `可读性` 和 `维护性`

    “more readable” and “easier to maintain”

- 推导式的实现

`values = [expression for item in collection]`

```python

values = []
for item in collection:
    values.append(expression)


values = [expression for item in collection]
```

- 带条件过滤的推导式

`values = [expression for item in collection if condition]`


```python
even_squares = [x * x for x in range(10) if x % 2 == 0]

even_squares = []
for x in range(10):
    if x % 2 == 0:
        even_squares.append(x * x)
```

- 集合推导式   `无序的`

```python

{ x * x for x in range(-9, 10) }
set([64, 1, 36, 0, 49, 9, 16, 81, 25, 4])

```

-  字典推导式

```python

{ x: x * x for x in range(5) }
{0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

```


#### 列表切片和 `sushi operator :`

列表有一个优雅的 切片操作 特性 `[start:stop:step]`

- 一般的切片操作
```python

>>> lst = [1, 2, 3, 4, 5]
>>> lst
[1, 2, 3, 4, 5]

# lst[start:end:step]
>>> lst[1:3:1]
[2, 3]
```

- 使用 `::` 列表的边界

```python
numbers = [1, 2, 3, 4, 5]
numbers[::-1] # 得到一个反序的拷贝

[5, 4, 3, 2, 1]
```

- 使用 `c = l[:]` 来获得一个浅拷贝

```python
l = [1,2,3]
c = l[:]

l.clear()

print(l)  # []
print(c)  # [1,2,3]

```



- 使用 `del l[:]` 来清空 列表 `l`
`py3+` 可以使用 `l.clear()` 
`py2+` 中没有 `clear()`方法


```python

l = [1,2,3,4,5]
c = l[:]

del l[:]

print(l)  # []
print(c)  # [1, 2, 3, 4, 5]

```

#### 怎样实现可迭代 `for in loop`


python的迭代器原则

     Python’s iterator protocol: 
     
         Objects that support the __iter__ and __next__ dunder methods
         automatically work with for-in loops.


- `__iter__` and `__next__` 是使一个Python对象iterable的关键.

```python

class Repeater:
    def __init__(self, value):
        self.value = value
        
    def __iter__(self):
        return RepeaterIterator(self)



class RepeaterIterator:
    def __init__(self, source):
        self.source = source

    def __next__(self):
        return self.source.value

for item in Repeater("hello"):
    print(item)


Hello
Hello
Hello
Hello
Hello
...

repeater = Repeater('Hello')
iterator = repeater.__iter__()  # iter(repeater)
while True:
    item = iterator.__next__()  # next(iterator)
    print(item)

```

- `for-in loops` 是怎么工作的

```python

class BoundedRepeater:
    def __init__(self, value, max_repeats):
        self.value = value
        self.max_repeats = max_repeats
        self.count = 0
        
    def __iter__(self):
        return self
        
    def __next__(self):
        if self.count >= self.max_repeats:
            raise StopIteration
        self.count += 1
        return self.value
                                                  
```
    
    首先会调用 __iter__() 生成一个 iterator 对象
    • It first prepared the repeater object for iteration by calling its
    __iter__ method. This returned the actual iterator object.
    
    然后调用iterator的 __next__() 方法产生 value
    • After that, the loop repeatedly called the iterator object’s
    __next__ method to retrieve values from it
    

这里注意 使用`next()` 或者 `__next__()` 方法, 或出现`StopIteration`

    迭代器耗尽后不会 重置

```python
>>> my_list = [1, 2, 3]
>>> iterator = iter(my_list)   # 生成一个iterator

>>> next(iterator)             # 调用迭代器的 __next__() next()方法  
1
>>> next(iterator)
2
>>> next(iterator)
3
 
>>> next(iterator)
StopIteration
 ```
for-in loop 中的异常处理方式

```python
repeater = BoundedRepeater("hello", 5)

# 1 调用 for-in loop, 遍历第一个可迭代对象
for item in repeater:
   print(item)

# 2 实际的操作
iterator = iter(repeater)
while True:
    try:
        item = next(iterator)
    except StopIteration:
        break
    print(item)
```

- `py2+`的兼容行


`py3+`中 使用 从迭代器中获取 value的方法 是  `__next__()`

`py2+` 中是 `next()`

```python

# 继承 object, 创建一个新式类

class InfiniteRepeater(object):
    def __init__(self, value):
        self.value = value

    def __iter__(self):
        return self
        
    def __next__(self):
        return self.value
        
    # Python 2 compatibility:
    def next(self):
        return self.__next__()
    
    # 增加 next(), 指向 __next__() 同时兼容 py2 和 py3

```


#### `生成器`  一种简化的迭代器 和 `yield`关键字  `memory efficient`

通过`generators` 和 `yield`, 更快，更有效的写一个 iterator 

```python

# 定义一个迭代器类 
class Repeater:
    def __init__(self, value):
        self.value = value
    def __iter__(self):
        return self
    def __next__(self):
        return self.value

# 定义一个生成器
def repeater(value):
    while True:
        yield value
```


- 通过 `yield` 来控制迭代次数

```python
def repeat_three_times(value):
    yield value
    yield value
    yield value

>>> for x in repeat_three_times('Hey there'):
... print(x)
'Hey there'
'Hey there'
'Hey there'

>>> iterator = repeat_three_times('Hey there')
>>> next(iterator)
'Hey there'
>>> next(iterator)
'Hey there'
>>> next(iterator)
'Hey there'
>>> next(iterator)
StopIteration
>>> next(iterator)
StopIteration




class BoundedRepeater:
    def __init__(self, value, max_repeats):
        self.value = value
        self.max_repeats = max_repeats
        self.count = 0
        
    def __iter__(self):
        return self
        
    def __next__(self):
        if self.count >= self.max_repeats:
            raise StopIteration
        self.count += 1
        return self.value


def bounded_repeater(value, max_repeats):
    count = 0
    while True:
        if count >= max_repeats:
            return
        count += 1
        yield value


def bounded_repeater(value, max_repeats):
    for _ in range(max_repeats):
        yield value

b = bounded_repeater("hello", 6)
for item in b:
    print(item)
    
hello
hello
hello
hello
hello
hello


```


#### 生成器表达式 `generator expression (...)` 


还是用上面的例子

```python

def bounded_repeater(value, max_repeats):
    for _ in range(max_repeats):
        yield value
        
        
iterator = bounded_repeater('Hello', 3)

# 生成器表达式
iterator_ = ("Hello" for _ in range(3))


>>> for x in iterator:
... print(x)
'Hello'
'Hello'
'Hello'

```

- `生成器表达式 (...)` vs `列表表达式 [...]`

```python
>>> listcomp = ['Hello' for i in range(3)]
>>> genexpr = ('Hello' for i in range(3))
```

- 带过滤的生成器表达式 

```python

genexpr = (expression for item in collection 
            if condition)


def generator():
    for item in collection:
        if condition:
            yield expression



# 多条件的推导式 ， 不推荐

for x in xs:
    if cond1:
        for y in ys:
            if cond2:
            ...
                for z in zs:
                    if condN:
                        yield expr

(expr for x in xs if cond1
      for y in ys if cond2
      ...
      for z in zs if condN)
```

#### `Iterator Chains` highly efficient data processing “pipelines”

来看一个例子

```python

def integers():
    for i in range(1, 9):
        yield i


def squared(seq):
    for i in seq:
        yield i * i 


def negated(seq):
    for i in seq:
        yield -i
     

>>> chain = integers()
>>> list(chain)
[1, 2, 3, 4, 5, 6, 7, 8]   
        
>>> chain = squared(integers())
>>> list(chain)
[1, 4, 9, 16, 25, 36, 49, 64]


>>> chain = negated(squared(integers()))
>>> list(chain)
[-1, -4, -9, -16, -25, -36, -49, -64]
```
    
    
    嵌套的 generator 最后生成值都是独立的
    Chained generators process each element going through the chain individually.
    
    



