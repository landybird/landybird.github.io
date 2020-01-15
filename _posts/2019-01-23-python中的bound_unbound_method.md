---
title: python中的bound和unbound方法
description: bound method 和 unbound method
categories:
- python
tags:
- python基础
---


> tips: There is `no unbounded` method in `Python 3`

[https://bugs.python.org/issue23702](https://bugs.python.org/issue23702)



类的属性都是放在 `__dict__` 中的


在Python中使用`描述器` 来表示`具有“绑定”行为`的`对象属性`，
使用`描述器协议方法`来控制对具有`绑定行为属性`的访问，
这些描述器协议方法包括：`__get__()`、`__set__()`和`__delete__()`。



#### `py2+` 中存在 `unbound method`

> 定义在类中的普通函数, 是`unbound method`， 没有 绑定`instance`

```python

class F(object):

    def foo_normal_function():
        print "foo_normal_function"


print(F.foo_normal_function)
print(F.__dict__["foo_normal_function"].__get__(None, F) )
print(F().foo_normal_function)
print(F.__dict__["foo_normal_function"].__get__(F(), F) )

# <unbound method F.foo_normal_function>
# <unbound method F.foo_normal_function>
# <bound method F.foo_normal_function of <__main__.F object at 0x03AE1910>>
# <bound method F.foo_normal_function of <__main__.F object at 0x03AE14D0>>

```


> `普通方法`, `实例方法`, `类方法`, `静态方法`的调用

```python

# coding: utf-8


class F():

    def foo_normal_function():
        print "foo_normal_function"

    def foo_instance(self):
        print "call foo_instance"

    @classmethod
    def foo_classmethod(cls):
        print "call foo_classmethod"

    @staticmethod
    def foo_staticmethod():
        print "call foo_staticmethod"


# F.foo_staticmethod()                              # call foo_staticmethod
# F.__dict__["foo_staticmethod"].__get__(None, F)() #  call foo_staticmethod

# F.foo_classmethod()
# F.__dict__["foo_classmethod"].__get__(None, F)()


# F.foo_instance(F())
# F().foo_instance()
# F.__dict__["foo_instance"].__get__(None, F)(F())
# F.__dict__["foo_instance"].__get__(F(), F)()


# F().foo_normal_function(F())                           # TypeError: foo_normal_function() takes no arguments (2 given)
# F.__dict__["foo_normal_function"].__get__(F(), F)(F()) # TypeError: foo_normal_function() takes no arguments (2 given)

```


有装饰器 `@staticmethod`， `@classmethod`装饰, (这里的装饰器是`描述器`), 调用描述器的`__get__`方法

```python

class staticmethod(object):
    """
    staticmethod(function) -> method
    
    Convert a function to be a static method.
    
    A static method does not receive an implicit first argument.
    To declare a static method, use this idiom:
    
         class C:
             @staticmethod
             def f(arg1, arg2, ...):
                 ...
    
    It can be called either on the class (e.g. C.f()) or on an instance
    (e.g. C().f()).  The instance is ignored except for its class.
    
    Static methods in Python are similar to those found in Java or C++.
    For a more advanced concept, see the classmethod builtin.
    """
    def __get__(self, *args, **kwargs): # real signature unknown
        """ Return an attribute of instance, which is of type owner. """
        pass

    def __init__(self, function): # real signature unknown; restored from __doc__
        pass

    @staticmethod # known case of __new__
    def __new__(*args, **kwargs): # real signature unknown
        """ Create and return a new object.  See help(type) for accurate signature. """
        pass

    __func__ = property(lambda self: object(), lambda self, v: None, lambda self: None)  # default

    __isabstractmethod__ = property(lambda self: object(), lambda self, v: None, lambda self: None)  # default


    __dict__ = None # (!) real value is ''

```




#### `py3+`中不存在 `unbound method`


```python


class F(object):
    
    # 没有self ,cls参数的 普通函数
    def foo_normal_function():
        print( "foo_normal_function")


# py2
print(F.foo_normal_function)   # <unbound method F.foo_normal_function>

# py3 
print(F.foo_normal_function)   # <function F.foo_normal_function at 0x0000018797783598>



```

[why-staticmethod-decorator-not-needed](https://stackoverflow.com/questions/59729444/why-staticmethod-decorator-not-needed#59730075)

