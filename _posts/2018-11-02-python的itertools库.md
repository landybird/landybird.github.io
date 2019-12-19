---
title: python的itertools库
description: python的itertools库 
categories:
- python
tags:
- python基础
---

<br>

# python的itertools库

<br>

Python的内建模块`itertools` 提供了非常有用的`用于操作迭代对象的函数`。


> 无限循环累加数字 `itertools.count(start=1, step=1)`

```python

import itertools

count_iter = itertools.count(1) # 步长是1 

for n in count_iter:
    print(n)

    1
    2
    ...
    150206
    150207
    150208
    150209
    150210
    150211
    150212
    150213
    150214
    150215
    ...
    
```

> 无限循环可迭代对象 `itertools.cycle(iterable)`


```python

import itertools

cycle_iter = itertools.cycle("ABCD") 

for n in cycle_iter:
    print(n)
    
    ..
    A
    B
    C
    D
    ...
   
   
```

> 制定重复的次数 `itertools.repeat(item, times)`


```python


In [47]: repeat_iter = itertools.repeat('A', 10)

In [48]: repeat_iter
Out[48]: repeat('A', 10)

In [49]: for i in repeat_iter:
    ...:     print(i)
    ...:
A
A
A
A
A
A
A
A
A
A

```

> 合并可迭代对象 `itertools.chain(iter1, iter2,...)`


```python

for c in itertools.chain('ABC', 'XYZ'):
    print c
# 迭代效果：'A' 'B' 'C' 'X' 'Y' 'Z'


```

> 分组重复的元素, 只是相邻的数值 `itertools.groupby("AABBBCCDA")`

```python

In [68]: ret = itertools.groupby('AABBBCCDA')

In [69]: for key, item in ret:
    ...:     print(key, item)
    ...:
A <itertools._grouper object at 0xfe9f9850>
B <itertools._grouper object at 0xfe9f9d50>
C <itertools._grouper object at 0xfe9f9850>
D <itertools._grouper object at 0xfe9f9d50>
A <itertools._grouper object at 0xfe9f9850>

In [70]: ret = itertools.groupby('AABBBCCDA')

In [71]: for key, item in ret:
    ...:     print(key, list(item)
    ...:     )
    ...:
    ...:
A ['A', 'A']
B ['B', 'B', 'B']
C ['C', 'C']
D ['D']
A ['A']

```

> 与map相似 （python2) `itertools.imap(lambda x: x*x, itertools.count(1))`

python3 中的map功能一样

`imap()` 和 `map()` 的区别在于，`imap()可以作用于无穷序列`，并且，如果两个序列的长度不一致，以短的那个为准。

同理 ifilter 与 filter