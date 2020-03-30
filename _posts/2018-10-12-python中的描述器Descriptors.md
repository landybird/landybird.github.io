---
title: python中的描述器Descriptors 
description: python中的描述符，描述器Descriptors
categories:
- python
tags:
- python基础
---

<br>


[https://docs.python.org/3.8/howto/descriptor.html](https://docs.python.org/3.8/howto/descriptor.html)

[https://pyzh.readthedocs.io/en/latest/Descriptor-HOW-TO-Guide.html](https://pyzh.readthedocs.io/en/latest/Descriptor-HOW-TO-Guide.html)


> tips: `描述器`只对于`新式对象和新式类`才`起作用`。继承于 object 的类叫做新式类




### 定义`描述器` Descriptors

`描述器 `是一种特殊 的对象，这种对象实现了 `__get__` ，`__set__` ，`__delete__` 这三个特殊方法。


    They are the mechanism behind properties, methods, static methods, class methods, and super().
    They are used throughout Python itself to implement the new style classes introduced in version 2.2
    
    在新式类中，属性，方法调用，静态方法，类方法等都是基于 描述符的特定使用



> 描述符可以说是`新式类调用链中的根基`

> 可以`利用描述符的特性`来将我们的`调用过程变得更为可控`



####  `资料描述器 data descriptors ` vs `非资料描述器 non data descriptors`

在描述符中同时实现了 `__get__` 与 `__set__ `协议的描述符是 `data descriptors `

只实现了 `__get__` 协议的则是 `non data descriptors`


#### 描述器协议


    descr.__get__(self, obj, type=None) --> value
    
    descr.__set__(self, obj, value) --> None
    
    descr.__delete__(self, obj) --> None


`一个对象`具有其中`任一个方法`就会成为`描述器`，从而在被`当作对象属性时`， `重写默认的查找行为`



#### 如果`实例字典`中有与`描述器同名的属性`


如果描述器是`资料描述器`，优先使用资料描述器，

如果是`非资料描述器`，优先使用字典中的属性



> tips: `descriptors` should only be defined as `class attributes`, not `instance attributes`

[https://stackoverflow.com/questions/23309698/why-is-the-descriptor-not-getting-called-when-defined-as-instance-attribute](https://stackoverflow.com/questions/23309698/why-is-the-descriptor-not-getting-called-when-defined-as-instance-attribute)

```python
def __getattribute__(self, attr):
        obj = object.__getattribute__(self, attr)
        if hasattr(obj, '__get__'):
            return obj.__get__(self, type(self))
        return obj

def __setattr__(self, attr, val):
    try:
        obj = object.__getattribute__(self, attr)
    except AttributeError:
        # This will be raised if we are setting the attribute for the first time
        # i.e inside `__init__` in your case.
        object.__setattr__(self, attr, val)
    else:
        if hasattr(obj, '__set__'):
            obj.__set__(self, val)
        else:
            object.__setattr__(self, attr, val)
```




### 描述器的调用



1 直接调用： `d.__get__(obj)`


2 更常见的情况是描述器在`属性访问时被自动调用`


    obj.d 会在 obj 的字典中找 d 
    
    如果 d 定义了 __get__ 方法，那么 d.__get__(obj)



> 调用的细节`取决于 obj` 是一个`类`还是一个`实例`

#### 调用顺序




举例来说， `a.x` 的查找顺序是, `a.__dict__['x'] `, 然后 `type(a).__dict__['x'] `, 然后找 `type(a) 的父类(不包括元类(metaclass))`.

如果`查找到的值是一个描述器`, Python就会`调用描述器的方法来重写默认的控制行为`。这个重写发生在这个查找环节的哪里取决于定义了哪个描述器方法


    注意, 只有在新式类中时描述器才会起作用。
    (新式类是继承自 type 或者 object 的类)





- 对于`对象`来说, `object.__getattribute__()`



      b.x 变成 type(b).__dict__['x'].__get__(b, type(b))


- 对于`类`来讲,  `type.__getattribute__()`



      B.x 变成 B.__dict__['x'].__get__(None, B)
      
  

注意
  
    描述器的调用是因为 __getattribute__()
    
    重写 __getattribute__() 方法会阻止正常的描述器调用
    
    __getattribute__() 只对新式类的实例可用
    
    object.__getattribute__() 和 type.__getattribute__() 对 __get__() 的调用不一样
    
    资料描述器总是比实例字典优先。
    
    非资料描述器可能被实例字典重写。(非资料描述器不如实例字典优先)
    


>`资料描述器` >> `实例变量` >> `非资料描述器` >> `__getattr__()`方法(具有最低的优先级)
    
```python

# >`资料描述器` >> `实例变量` >> `非资料描述器` >> `__getattr__()`方法(具有最低的优先级)


#data descriptor
class A(object):
   def __get__(self, obj, type):
       print( "hello from get A")
       return "A __get__"

   def __set__(self, obj, val):
       print ("hello from set A")

#non data descriptor
class B(object):
   def __get__(self, obj, type):
       print ("hello from get B")

class C(object):
   #our data descriptor
   a = A()
   #our non data descriptor
   b = B()

c = C()

print(c.a)
# hello from get A
# A __get__
print(c.b)
# hello from get B
# None
c.a = 0
print(c.a)
# hello from set A
# hello from get A
# A __get__
c.b = 0
print(c.b)
# 0




class RevealAccess(object):
    """A data descriptor that sets and returns values
       normally and prints a message logging their access.
    """

    def __init__(self, initval=None, name='var'):
        self.val = initval
        self.name = name

    def __get__(self, obj, objtype):
        print('Retrieving', self.name)
        return self.val

    def __set__(self, obj, val):
        print('Updating' , self.name)
        self.val = val


class MyClass(object):
    # x = RevealAccess(10, 'x')
    y = 5

    # def __init__(self):
    #     self.x = "__init__  obj_dict"


    def __getattr__(self, item):
        if item == "x":
            return 1.5

    # def __getattribute__(self, item):
    #     if item == "x":
    #         return "__getattribute__"
    #     return super(MyClass, self).__getattribute__(item)






m = MyClass()

print(m.x)







```


    


### `属性(properties)` 调用


调用 property() 是建立资料描述器的一种简洁方式，从而可以`在访问属性时触发相应的方法调用`


```python


#  类似 property装饰器的使用


import math
class lazyproperty:
    def __init__(self, func):
        self.func = func

    def __get__(self, instance, owner):
        print("__get__")
        if instance is None:
            return self
        else:
            value = self.func(instance)
            setattr(instance, self.func.__name__, value)
            return value


class Circle:
    def __init__(self, radius):
        self.radius = radius
        pass

    @lazyproperty
    def area(self):
        print("Com")
        return math.pi * self.radius * 2

    # area = lazyproperty(area)

    def test(self):
        pass


if __name__=='__main__':
    c=Circle(4)
    print(c.__dict__)      # {'radius': 4}
    print(Circle.__dict__) # {'__module__': '__main__', '__init__': <function Circle.__init__ at 0x00000181C38436A8>, 'area': <__main__.lazyproperty object at 0x00000181C385DA20>, 'test': <function Circle.test at 0x00000181C38437B8>, '__dict__': <attribute '__dict__' of 'Circle' objects>, '__weakref__': <attribute '__weakref__' of 'Circle' objects>, '__doc__': None}

    print(c.area)
    #  第一次调用 c.area 的时候，我们首先查询实例 c 的 __dict__ 中是否存在着 area 描述符
    #  c 中既不存在描述符，也不存在这样一个属性，接着我们向上查询 Circle 中的 __dict__
    #  然后查找到名为 area 的属性，同时这是一个 non data descriptors ，由于我们的实例字典内并不存在 area 属性
    #  实例字典内 不存在 area 属性，调用类字典中的 area 的 __get__ 方法
    #  这里的 area = lazyproperty(area) === @lazyproperty
    #  调用类字典中的 area 的 __get__ 方法，并在 __get__ 方法中通过调用 setattr 方法为实例字典注册属性 area
    print(c.__dict__)      # {'radius': 4, 'area': 25.132741228718345}
    print(Circle.__dict__) # {'__module__': '__main__', '__init__': <function Circle.__init__ at 0x00000181C38436A8>, 'area': <__main__.lazyproperty object at 0x00000181C385DA20>, 'test': <function Circle.test at 0x00000181C38437B8>, '__dict__': <attribute '__dict__' of 'Circle' objects>, '__weakref__': <attribute '__weakref__' of 'Circle' objects>, '__doc__': None}


    print(c.__dict__)      # {'radius': 4, 'area': 25.132741228718345}
```



#### 描述器应用的实例:



```python


from weakref import WeakKeyDictionary


class NonNegative(object):
    """A descriptor that forbids negative values"""

    def __init__(self, default):
        self.default = default
        self.data = WeakKeyDictionary()  # dict()
        # print(self.data) # <WeakKeyDictionary at 0x26447ad5790>

    def __get__(self, instance, owner):
        # we get here when someone calls x.d, and d is a NonNegative instance
        # instance = x
        # owner = type(x)
        # print(dict(self.data))
        return self.data.get(instance, self.default)

    def __set__(self, instance, value):
        # we get here when someone calls x.d = val, and d is a NonNegative instance
        # instance = x
        # value = val
        if value < 0:
            raise ValueError("Negative value not allowed: %s" % value)
        self.data[instance] = value


class Movie(object):
    # always put descriptors at the class-level
    # 类级别的描述器

    rating = NonNegative(0)
    # runtime = NonNegative(0)
    # budget = NonNegative(0)
    # gross = NonNegative(0)

    def __init__(self, title, rating, runtime, budget, gross):
        self.title = title

        # 调用描述器NonNegative的__set__方法
        self.rating = rating
        self.runtime = runtime
        self.budget = budget
        self.gross = gross

    def profit(self):
        return self.gross - self.budget


m = Movie('Casablanca', 97, 102, 964000, 1300000)

print(m.budget)  # calls Movie.budget.__get__(m, Movie))

m.budget = 111
print(m.budget)

m.budget = -11
try:
    m.rating = -1  # calls Movie.budget.__set__(m, -100)
except ValueError:
    print("Woops, negative value")




```
