---
title: python中的下划线
description: python中的各种下划线变量的使用
categories:
- python
tags:
- python tricks
---

<br>

- 先来看python中的五种下划线
        
        • Single Leading Underscore: _var
        • Single Trailing Underscore: var_
        • Double Leading Underscore: __var
        • Double Leading and Trailing Underscore: __var__
        • Single Underscore: _


- 单前缀下划线 `_var`

约定俗成的使用 `_var` 来标识内部使用的 变量或者是方法

当然只是规范， 并不会被python解释器强制执行, 也不会影响正常的程序运行

```python
class Test:
    def __init__(self):
        self.foo = 11
        self._bar = 23

t = Test()
t._bar
>> 23 
#不影响程序的运行
```

不过 如果使用 `from module_test import *` ，module中定义`_var`, 这时的 `_var`不会被引用

```python

# module_test.py

def external_func():
    return 23

def _internal_func():
    return 42


# main.py

from my_module import *

external_func()
>>23

_internal_func()
NameError: "name '_internal_func' is not defined"

```

当然可以通过在 `module_text.py` 中定义 `__all__`列表 来指定引用时候的变量

```python


# module_test.py

__all__ = ['external_func', '_internal_func']

def external_func():
    return 23

def _internal_func():
    return 42


# main.py

from my_module import *

external_func()
>>23

_internal_func()
>> 42

```

- 单下划线后缀 `var_`

在定义变量的时候，合适的变量名已经使用(python的既定变量, 或者其他定义的变量)

为了避免命名冲突 -- python关键字


```python

def make_object(name, class):
SyntaxError: "invalid syntax"

def make_object(name, class_):
... pass

```

- 双下划线前缀 `__var` 使用在类中

       1 类中的私有变量
       2 _Test__var1  将var1 全局的传给 Test类
 
<1>

为了避免子类中的`naming conflicts`

Python中的成员函数和成员变量都是公开的(public),在python中没有类似public,private等关键词来修饰成员函数和成员变量。
在python中定义私有变量只需要在变量名或函数名前加上 __两个下划线，那么这个函数或变量就是私有的了。

在内部，python使用一种 `name mangling`，将 __membername替换成 _classname__membername，也就是说，类的内部定义中,
所有以双下划线开始的名字都被"翻译"成前面加上单下划线和类名的形式。

```python
class Test:
    def __init__(self):
        self.foo = 11
        self._bar = 23
        self.__baz = 23

t = Test()

dir(t)

['_Test__baz', '__class__', '__delattr__', '__dict__',
'__dir__', '__doc__', '__eq__', '__format__', '__ge__',
'__getattribute__', '__gt__', '__hash__', '__init__',
'__le__', '__lt__', '__module__', '__ne__', '__new__',
'__reduce__', '__reduce_ex__', '__repr__',
'__setattr__', '__sizeof__', '__str__',
'__subclasshook__', '__weakref__', '_bar', 'foo']


# 在子类中复写 __baz属性

class ExtendedTest(Test):
    def __init__(self):
        super().__init__()
        self.foo = 'overridden'
        self._bar = 'overridden'
        self.__baz = 'overridden'



t2 = ExtendedTest()

dir(t2)

['_ExtendedTest__baz', '_Test__baz', '__class__',
'__delattr__', '__dict__', '__dir__', '__doc__',
'__eq__', '__format__', '__ge__', '__getattribute__',
'__gt__', '__hash__', '__init__', '__le__', '__lt__',
'__module__', '__ne__', '__new__', '__reduce__',
'__reduce_ex__', '__repr__', '__setattr__',
'__sizeof__', '__str__', '__subclasshook__',
'__weakref__', '_bar', 'foo', 'get_vars']

t2._ExtendedTest__baz 
>> 'overridden'

t2._Test__baz
>> 42

```

<2> 
还有一个特殊的例子

```python

_MangledGlobal__mangled = 23

# 将__mangled 传给 MangledGlobal

class MangledGlobal:
    def test(self):
        return __mangled

m = MangledGlobal()
r = m.test()
print(r)

```



- 魔法方法 `__init__之类`

有趣的是 `__var__` 并不适用于 `name mangling` 规则

```python
class PrefixPostfixTest:
    def __init__(self):
        self.__bam__ = 42
        
PrefixPostfixTest().__bam__
>> 42

```


- 单下划线 `_`


习惯上 `_` 用来表示 临时, 不使用的变量

```python
for _ in range(32):
    print('Hello, World.')
```


用不到的变量
```python
car = ('red', 'auto', 12, 3812.4)
color, _, _, mileage = car
```

临时变量(在解释器的session中)
```python
#会用 _ 来标识上次结果

>>> 20 + 3
23
>>> _
23
>>> print(_)
23


>>> list()
[]
>>> _.append(1)
>>> _.append(2)
>>> _.append(3)
>>> _
[1, 2, 3]

```
