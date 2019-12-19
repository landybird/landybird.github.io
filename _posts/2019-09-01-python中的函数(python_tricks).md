---
title: python中的函数
description: python中的函数
categories:
- python
tags:
- python tricks
---

<br>

- 函数是对象

```python

def yell(text):
    return text.upper() + '!'
    
yell('hello')
'HELLO!'


bark = yell
bark('woof')
'WOOF!'

del yell

yell('hello?')
NameError: "name 'yell' is not defined"


bark('hey')
'HEY!'

```


- 函数可以储存在容器里

```python

funcs = [bark, str.lower, str.capitalize]

funcs

[<function yell at 0x10ff96510>,
<method 'lower' of 'str' objects>,
<method 'capitalize' of 'str' objects>]

funcs[0]('heyho')
'HEYHO!

```


- 函数可以作为参数传递给另一个函数

```python
list(map(bark, ['hello', 'hey', 'hi']))

['HELLO!', 'HEY!', 'HI!']
```

- 函数嵌套, 闭包, 装饰器


```python

def get_speak_func(volume):
    def whisper(text):
        return text.lower() + '...'
    def yell(text):
        return text.upper() + '!'
    if volume > 0.5:
        return yell
    else:
        return whisper


def make_adder(n):
    def add(x):
        return x + n
    return add
    
plus_3 = make_adder(3)
plus_5 = make_adder(5)

>>> plus_3(4)
7
>>> plus_5(4)
9


```


- 对象像函数一样使用 `__call__`


```python

class Adder:
    def __init__(self, n):
        self.n = n
    def __call__(self, x):
        return self.n + x
        
>>> plus_3 = Adder(3)
>>> plus_3(4)
7


```
