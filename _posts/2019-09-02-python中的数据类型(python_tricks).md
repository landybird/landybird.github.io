---
title: python中的数据类型
description: python中的数据类型
categories:
- python
tags:
- python tricks
---

<br>
 
#### 字典 `Dictionaries, Maps, and Hashtables`


- 定义一个字典的姿势

```python

phonebook = {
    'bob': 7387,
    'alice': 3719,
    'jack': 7052,
    }


squares = {x: x * x for x in range(6)}

```

python的键必须是一个 `hashable`类型 

    __hash__ 有一个周期内不会改变的值
    
    __eq__  还可以进行比较
    


- 各种字典


`collections.OrderedDict`  有序字典


`py3.6+`已经是有序字典

```python

>>> import collections
>>> d = collections.OrderedDict(one=1, two=2, three=3)
OrderedDict([('one', 1), ('two', 2), ('three', 3)])

>>> d['four'] = 4
>>> d
OrderedDict([('one', 1), ('two', 2),
('three', 3), ('four', 4)])

>>> d.keys()
odict_keys(['one', 'two', 'three', 'four'])


```

`collections.defaultdict`   给字典设置默认的值

```python

>>> from collections import defaultdict
>>> dd = defaultdict(list)

# Accessing a missing key creates it and
# initializes it using the default factory,
# i.e. list() in this example:

>>> dd['dogs'].append('Rufus')
>>> dd['dogs'].append('Kathrin')
>>> dd['dogs'].append('Mr Sniffles')
>>> dd['dogs']
['Rufus', 'Kathrin', 'Mr Sniffles']

```

`collections.ChainMap`   Search Multiple Dictionaries as a Single Mapping

```python
from collections import ChainMap, OrderedDict

d1 = {"a": 1, "b": 2}
d2 = {"c": 1, "d": 2}


chain = ChainMap(d1, d2)

chain['c'] = 1.1

print(chain)
print(d1)

# 只会影响第一个mapping

ChainMap({'a': 1, 'b': 2, 'c': 1.1}, {'c': 1, 'd': 2})
{'a': 1, 'b': 2, 'c': 1.1}
```

`types.MappingProxyType` `py3.3+`  A Wrapper for Making Read-Only Dictionaries

```python

from types import MappingProxyType

writable = {'one': 1, 'two': 2}
read_only = MappingProxyType(writable)

# The proxy is read-only:
read_only['one']
1
read_only['one'] = 23
TypeError:
"'mappingproxy' object does not support item assignment"

# Updates to the original are reflected in the proxy:
writable['one'] = 42
read_only
mappingproxy({'one': 42, 'two': 2})


```


#### 列表， 元祖 `Array`


- 列表 `动态的， 可变的array`

```python

arr = ['one', 'two', 'three']
arr[0]
'one'

# Lists have a nice repr:
arr
['one', 'two', 'three']


# Lists are mutable:
arr[1] = 'hello'

arr
['one', 'hello', 'three']

del arr[1]
arr
['one', 'three']

# Lists can hold arbitrary data types:
arr.append(23)

arr
['one', 'three', 23]

```

- 元祖 `不可变容器`

```python

arr = 'one', 'two', 'three'

arr[0]
'one'

# Tuples have a nice repr:
arr
('one', 'two', 'three')


# Tuples are immutable:
arr[1] = 'hello'

TypeError:
"'tuple' object does not support item assignment"

del arr[1]
TypeError:
"'tuple' object doesn't support item deletion"

# Tuples can hold arbitrary data types:
# (Adding elements creates a copy of the tuple)
arr + (23,)
('one', 'two', 'three', 23)

```

元祖的初始化 要比 列表 更快捷

```python

r = compile("(1,2,3)", '', 'eval')
r1 = compile("[1,2,3]", '', 'eval')

import dis

dis.dis(r)
1           0 LOAD_CONST               3 ((1, 2, 3))
            2 RETURN_VALUE


dis.dis(r1)
            
1         0 LOAD_CONST               0 (1)
          2 LOAD_CONST               1 (2)
          4 LOAD_CONST               2 (3)
          6 BUILD_LIST               3
          8 RETURN_VALUE

```


- array.array `Basic Typed Arrays`

array模块是python中实现的一种高效的数组存储类型。它和list相似，但是所有的数组成员必须是同一种类型，在创建数组的时候，就确定了数组的类型

```python

import array
arr = array.array('f', (1.0, 1.5, 2.0, 2.5))

```

- str 不可变的 `unicode characters`array

```python

>>> arr = 'abcd'
>>> arr[1]
'b'

>>> arr
'abcd'

# Strings are immutable:
>>> arr[1] = 'e'
TypeError:
"'str' object does not support item assignment"

>>> del arr[1]
TypeError:
"'str' object doesn't support item deletion"

# Strings can be unpacked into a list to
# get a mutable representation:
>>> list('abcd')
['a', 'b', 'c', 'd']
>>> ''.join(list('abcd'))
'abcd'

# Strings are recursive data structures:
>>> type('abc')
"<class 'str'>"
>>> type('abc'[0])
"<class 'str'>"

```

- bytes 不可变的 `single bytes` array

- bytearray 可变的 `single bytes` array


- `typing.NamedTuple` – Improved Namedtuples
    
```python

from typing import NamedTuple

class Car(NamedTuple):
    color: str
    mileage: float
    automatic: bool

car1 = Car('red', 3812.4, True)

# Instances have a nice repr:
>>> car1
Car(color='red', mileage=3812.4, automatic=True)



```

使用 `Namedtuple` 来初始化数据结构



#### 集合 `Set and Multisets` `hashable`



注意： 初始化一个空集合 `set()` 而不是 `{}`, 后者只是一个字典

```python

>>> vowels = {'a', 'e', 'i', 'o', 'u'}
>>> 'e' in vowels
True

>>> letters = set('alice')
>>> letters.intersection(vowels)
{'a', 'e', 'i'}

>>> vowels.add('x')
>>> vowels
{'i', 'a', 'u', 'o', 'x', 'e'}
>>> len(vowels)
6
```

- `frozenset` 不可变 集合

```python

>>> vowels = frozenset({'a', 'e', 'i', 'o', 'u'})
>>> vowels.add('p')
AttributeError:
"'frozenset' object has no attribute 'add'"
# Frozensets are hashable and can
# be used as dictionary keys:

>>> d = { frozenset({1, 2, 3}): 'hello' }
>>> d[frozenset({1, 2, 3})]
'hello'

```

- `collections.Counter `– `Multisets or bag`

可以装进去很多 集合 ， 字典的元素

```python

rom collections import Counter

c = Counter()

l = {1, 2, 'a'}
l1 = {1: 1, 2: 1, 'a': 1}

c.update(l)
c.update(l1)

print(c)
Counter({1: 2, 2: 2, 'a': 2})


print(len(c))          # 3
print(sum(c.values())) # 6 

```


#### stack 堆栈 （LIFOs）


- list `simple, built-in-stack`

只有 append 和 pop 操作是 O(1), 其他操作性能很低


```python
>>> s = []
>>> s.append('eat')
>>> s.append('sleep')
>>> s.append('code')

>>> s
['eat', 'sleep', 'code']
>>> s.pop()
'code'
>>> s.pop()
'sleep'
>>> s.pop()
'eat'
>>> s.pop()
IndexError: "pop from empty list"
```


- collections.deque `fast & robust stack` (基于双向队列)

```python

>>> from collections import deque
>>> s = deque()
>>> s.append('eat')
>>> s.append('sleep')

>>> s.append('code')
>>> s
deque(['eat', 'sleep', 'code'])
>>> s.pop()
'code'
>>> s.pop()
'sleep'
>>> s.pop()
'eat'
>>> s.pop()
IndexError: "pop from an empty deque"

```

-  queue.LifoQueue –` Locking Semantics for Parallel Computing`

`locking` + `multi-computing` + `synchronized`

```python
>>> from queue import LifoQueue
>>> s = LifoQueue()

>>> s.put('eat')
>>> s.put('sleep')
>>> s.put('code')

>>> s
<queue.LifoQueue object at 0x108298dd8>

>>> s.get()
'code'
>>> s.get()
'sleep'
>>> s.get()
'eat'

>>> s.get_nowait()
queue.Empty

>>> s.get()
# Blocks / waits forever...
```


    list 的左右操作， append() 和 pop() 是 O(1), 其他操作是 O(n)

    collections.deque 基于 双向队列, 插入和删除方法 O(1)


> `collections.deque` 是 python中优秀的堆栈实现

    

####  队列 `Queues` (FIFOs)
    


- list `Terribly Sloooow Queues`  

```python

>>> q = []
>>> q.append('eat')
>>> q.append('sleep')
>>> q.append('code')

>>> q
['eat', 'sleep', 'code']

# Careful: This is slow!
>>> q.pop(0)
'eat'

```

 
- collections.deque – `Fast & Robust Queues`

     
    Python’s deque objects are implemented as doubly-linked lists.36 This
    gives them excellent and consistent performance for inserting and
    deleting elements, but poor O(n) performance for randomly accessing
    elements in the middle of the stack.


```python

>>> from collections import deque

>>> q = deque()
>>> q.append('eat')
>>> q.append('sleep')

>>> q.append('code')
>>> q
deque(['eat', 'sleep', 'code'])

>>> q.popleft()
'eat'
>>> q.popleft()
'sleep'
>>> q.popleft()
'code'

>>> q.popleft()
IndexError: "pop from an empty deque"

```

- queue.Queue – `Locking Semantics for Parallel Computing`

`locking` + `multi-computing` + `synchronized`

```python

>>> from queue import Queue

>>> q = Queue()
>>> q.put('eat')
>>> q.put('sleep')
>>> q.put('code')

>>> q
<queue.Queue object at 0x1070f5b38>

>>> q.get()
'eat'
>>> q.get()
'sleep'
>>> q.get()
'code'

>>> q.get_nowait()
queue.Empty

>>> q.get()
# Blocks / waits forever...

```

- multiprocessing.Queue – `Shared Job Queues`

    
    Processbased parallelization is popular in CPython due to the 
    global interpreter lock (GIL) that prevents some forms of 
    parallel execution on a single interpreter process.
    
    multiprocessing.Queue makes it easy to distribute work across multiple 
    processes in order to work around the GIL limitations.
    
这种类型的`queue`可以在进程中 储存和传递 任何可以序列化的对象(`pickle-able`)    
    
    
```python

>>> from multiprocessing import Queue

>>> q = Queue()
>>> q.put('eat')
>>> q.put('sleep')

>>> q.put('code')
>>> q
<multiprocessing.queues.Queue object at 0x1081c12b0>

>>> q.get()
'eat'
>>> q.get()
'sleep'
>>> q.get()
'code'

>>> q.get()
# Blocks / waits forever...

```

#### 优先级队列 ` Priority Queues`

不是根据元素进入的 时间先后, 而是根据 优先级(权重) 来操作队列

- list – `Maintaining a Manually Sorted Queue`

```python

q = []
q.append((2, 'code'))
q.append((1, 'eat'))
q.append((3, 'sleep'))

# NOTE: Remember to re-sort every time
# a new element is inserted, or use
# bisect.insort().

q.sort(reverse=True)

while q:
    next_item = q.pop()
    print(next_item)
    
# Result:
# (1, 'eat')
# (2, 'code')
# (3, 'sleep')

```

- heapq – `List-Based Binary Heaps`

```python

import heapq

q = []
heapq.heappush(q, (2, 'code'))
heapq.heappush(q, (1, 'eat'))
heapq.heappush(q, (3, 'sleep'))

while q:
    next_item = heapq.heappop(q)
    print(next_item)
    
# Result:
# (1, 'eat')
# (2, 'code')
# (3, 'sleep')

```



- queue.PriorityQueue – `Beautiful Priority Queues`

`locking` + `multi-computing` + `synchronized`

```python
from queue import PriorityQueue

q = PriorityQueue()
q.put((2, 'code'))
q.put((1, 'eat'))
q.put((3, 'sleep'))

while not q.empty():
    next_item = q.get()
    print(next_item)
    
# Result:
# (1, 'eat')
# (2, 'code')
# (3, 'sleep')


```

