---
title: python中的dataclass(`py3.7+`)                  
description: python中的dataclass(`py3.7+`)   
categories:
- python
tags:
- python   
---


    
    减少代码量, a data class requires a minimal amount of code
    
    可以进行比较， 因为实现了 __eq__ , you can compare data classes because __eq__ is implemented for you 
        基于 对象属性  self.attr == other.attr
    
    debugging 的时候可以打印详细的信息, 因为实现了__repr__, you can easily print a data class for debugging because __repr__ is implemented as well
    
    进行了类型hint，减少数据类型bug， data classes require type hints, reduced the chances of bugs


[dataclass介绍](https://landybird.github.io/python/2020/01/09/python30%E4%B8%AAtips/)
[实例(dataclass 类对象去重--有hash方法)](https://landybird.github.io/python/2019/08/20/python%E7%9A%84dataclasses%E6%A8%A1%E5%9D%97/)


#### 适合存储`数据对象`

        1 存储数据并表示某种数据类型
        2 对象进行比较


```python
from dataclasses import dataclass

class Number:
    def __init__(self, val):
        self.val = val

one = Number(1)
print(one.val)


@dataclass()
class NewNumber:
    val:int=0

new_one = Number(1)
print(new_one.val)

```
     

####  创建不可变对象 `frozen=True`

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class NewNumber:
    val:int=0

a = NewNumber()
print(a.val)
a.val = 0

```



#### 使用`__post_init__`进行初始化后处理

```python
import math

class Float:
    def __init__(self, val=0):
        self.val = val
        self._post_init()

    def _post_init(self):
        self.decimal, self.integer = math.modf(self.val)


f = Float(2.5)

print(f.decimal)
print(f.integer)



from dataclasses import dataclass
import math

@dataclass
class NewFloat:
    val: float = 0.0
    def __post_init__(self):
        self.decimal,\
        self.integer = math.modf(self.val)


nf = NewFloat(2.5)
print(nf.val)
print(nf.decimal)
print(nf.integer)

```



        
        