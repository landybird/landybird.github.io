---
title: 类以及OOP面向对象
description: 类以及OOP面向对象
categories:
- python
tags:
- python tricks
---

<br>
 
#### 对象的比较 `is` 和 `==`

```python

a = [1, 2, 3]
b = a


a == b
True
a is b
True


c = list(a)

a == c
True

a is c
False


```

上面的 `a`, `c` 指向不同的list对象, 但是他们的内容是相同的


`is`  如果两个变量指向相同的对象

`==`  两个变量内容相等
    
    • An is expression evaluates to True if two variables point to the
    same (identical) object.
    
    • An == expression evaluates to True if the objects referred to by
    the variables are equal (have the same contents).



#### 每个类需要一个 `__repr__`

 - 使用`__repr__`, `__str__` 方法 转换 对象成字符串

```python

class Car:
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage
    def __str__(self):
        return f'a {self.color} car'

my_car = Car('red', 37281)

print(my_car)
'a red car'

```

- 什么时候会转换(调用)

```python
# 1
print(my_car)
a red car

#2
str(my_car)
'a red car'


#3
'{}'.format(my_car)
'a red car'

```

- `__str__` 和 `__repr__`的区别

如果没有 `__str__` python 会自动使用 `__repr__`的结果


```python

class Car:
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage
    def __repr__(self):
        return '__repr__ for Car'
    def __str__(self):
        return '__str__ for Car'

```

`__repr__` 是展现给python解释器的
`__str__` 是展现给用户的

我们可以看一下 `datetime`模块中的使用方式
```python

import datetime
today = datetime.date.today()

str(today)
'2017-02-02'


repr(today)
'datetime.date(2017, 2, 2)'
# 会带上 模块名

```

- 怎么样写一个 `__repr__`方法

```python

class Test:
    def __init__(self, name, color):
        self.name = name
        self.color = color

    def __repr__(self):
        # Don’t Repeat Yourself (DRY) 规则
        return (
                  f'{self.__class__.__name__}'
                  f'({self.name!r}, {self.color!r})'
                )
                
    def __str__(self):
        return f'a {self.color} car'

```

- python2中的 `__unicode__`

在 `py2`中 默认 `__str__`返回 bytes,  而`__unicode__`返回字符串


以下是py2 字符串方法定义的一个完整方法

```python

class Car(object):
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage
    def __repr__(self):
        return '{}({!r}, {!r})'.format(
        self.__class__.__name__,
        self.color, self.mileage)
    def __unicode__(self):
        return u'a {self.color} car'.format(
        self=self)
    def __str__(self):
        return unicode(self).encode('utf-8')

```


#### 定义一个自己的异常类

更好地定位，以及处理异常

```python

def validate(name):
    if len(name) < 10:
        raise ValueError


class NameTooShortError(ValueError):
    pass
    
def validate(name):
    if len(name) < 10:
    raise NameTooShortError(name)
```

如果是一个会被调用的模块, 异常应该细化

```python

class BaseValidationError(ValueError):
    pass


class NameTooShortError(BaseValidationError):
    pass
class NameTooLongError(BaseValidationError):
    pass
class NameTooCuteError(BaseValidationError):
    pass
    

# 方便使用 细化处理
try:
    validate(name)
except BaseValidationError as err:
    handle_validation_error(err)
```


#### 对象的复制, 拷贝

对于不可变对象， python中的赋值语句不会创建新的对象, 只是把变量指向这个对象


一个浅拷贝的例子 `shallow copy`

```python

xs = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
ys = list(xs) # shallow copy

xs.append(['new sublist'])
# ys 浅拷贝， 最外层的列表 list 到另一个内存地址, 
xs
[[1, 2, 3], [4, 5, 6], [7, 8, 9], ['new sublist']]
ys
[[1, 2, 3], [4, 5, 6], [7, 8, 9]]



xs[1][0] = 'X'
# ys 浅拷贝  但是列表内部的元素只是拷贝的引用， 还是指向的原地址

 xs
[[1, 2, 3], ['X', 5, 6], [7, 8, 9], ['new sublist']]
ys
[[1, 2, 3], ['X', 5, 6], [7, 8, 9]]

```

深拷贝 `deep copy`


```python
import copy

xs = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
zs = copy.deepcopy(xs)

xs
[[1, 2, 3], [4, 5, 6], [7, 8, 9]]
zs
[[1, 2, 3], [4, 5, 6], [7, 8, 9]]


xs[1][0] = 'X'

xs
[[1, 2, 3], ['X', 5, 6], [7, 8, 9]]
zs
[[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```


#### 抽象类 `Abstract Base Classes (ABCs)` 继承指定的方法


`Abstract Base Classes` 可以防止忘记实现子类的特定接口, 及时抛出错误


- 没有抽象类， 实现方式

```python

class Base:
    def foo(self):
        raise NotImplementedError()
        
    def bar(self):
        raise NotImplementedError()

class Concrete(Base):
    def foo(self):
        return 'foo() called'
    # Oh no, we forgot to override bar()...
    # def bar(self):
    #   return "bar() called"
    

>>> c = Concrete()
>>> c.foo()
'foo() called'
>>> c.bar()
NotImplementedError
```

- 使用 `abc`模块， `py2.6+`

```python

from abc import ABCMeta, abstractmethod

class Base(metaclass=ABCMeta):
    @abstractmethod
    def foo(self):
        pass
    @abstractmethod
    def bar(self):
        pass
    
class Concrete(Base):
    def foo(self):
        pass
    # We forget to declare bar() again...


> c = Concrete()
TypeError:
"Can't instantiate abstract class Concrete
with abstract methods bar"
```

没有 `abc` 模块 , 没有实现指定的父类接口， 我们只是获得一个 `a NotImplementedError`




#### 有名分组 `Namedtuples` 的用处 `py2.6+`


    是一种手动`定义 类` 的方式
    
    是 python 内置的 tuple的扩展
    
    是不可变的
    
Namedtuples are a memory-efficient shortcut to defining an immutable class in Python manually


```python
from collections import namedtuple
Car = namedtuple('Car' , 'color mileage')
# Car = namedtuple('Car', ['color', 'mileage']) 
```

- Namedtuple是类， 可以继承

```python
Car = namedtuple('Car', 'color mileage')

class MyCarWithMethods(Car):
    def hexcolor(self):
        if self.color == 'red':
            return '#ff0000'
        else:
            return '#000000'
```

- namedtuple 的一些属性，方法


```python
from collections import namedtuple

Car = namedtuple('Car', 'color mileage')
car = Car('red', 2456)
```

`_fields`

```python

Car._fields
('color', 'mileage')


ElectricCar = namedtuple(
   'ElectricCar', Car._fields + ('charge',))

```

`_asdict()`

```python
my_car._asdict()
OrderedDict([('color', 'red'), ('mileage', 3812.4)])

json.dumps(my_car._asdict())
'{"color": "red", "mileage": 3812.4}'
```

`_replace()`

创建一个新的对象

```python
new_car = my_car._replace(color='blue')
Car(color='blue', mileage=3812.4)
```

`_make()`
创建一个新的对象

```python

Car._make(['red', 999])
Car(color='red', mileage=999)

```

#### 类与对象的 变量问题


类变量 和 对象变量

```python

class Dog:
    num_legs = 4 # <- Class variable
    
    def __init__(self, name):
        self.name = name # <- Instance variable

```

类变量的变化会影响全部的实例对象

```python

class CountedObject:
    num_instances = 0
    def __init__(self):
        self.num_instances += 1

print(CountedObject.num_instances)
print(CountedObject().num_instances)
print(CountedObject().num_instances)
print(CountedObject.num_instances)

# 0
# 1
# 1
# 0



class CountedObject:
    num_instances = 0
    def __init__(self):
        self.__class__.num_instances += 1

print(CountedObject.num_instances)
print(CountedObject().num_instances)
print(CountedObject().num_instances)
print(CountedObject.num_instances)

# 0
# 1
# 2
# 2

```




