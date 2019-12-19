---
title: 逗号的位置
description: 逗号的位置
categories:
- python
tags:
- python tricks
---

<br>

#### 逗号的位置 -- 每行一个逗号

很多代码资源管理工具(git) 是根据行来显示代码的变化的

```python

names = [
    'Alice',
     'Bob',
    'Dilbert'
    ]


要优于

names = ['Alice', 'Bob', 'Dilbert']

```

代码尽量不要写在一行


注意 `string literal concatenation`

```python
s = "hello" 'world' """It's OK!"""
>>> helloworldI's OK!
```

使用 `string literal concatenation` 可以减少 `"\"`的使用

```python

my_str = 'This is a super long string constant '\
'spread out across multiple lines. '\
'And look, no backslash characters needed!'


my_str = ('This is a super long string constant '
'spread out across multiple lines. '
'And look, no backslash characters needed!')

```
