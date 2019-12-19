---
title: python annotations注释
description: python annotations注释
categories:
 - python
tags:
- python基础
---


# python annotations注释


```python

文档中的例子：
    
    >>> def f(ham: str, eggs: str = 'eggs') -> str:
    ...     print("Annotations:", f.__annotations__)
    ...     print("Arguments:", ham, eggs)
    ...     return ham + ' and ' + eggs
    ...
    >>> f('spam')
    Annotations: {'ham': <class 'str'>, 'return': <class 'str'>, 'eggs': <class 'str'>}
    Arguments: spam eggs
    'spam and eggs
    
```

不是语法级别的硬性要求,但是顾名思义,它可做为函数额外的注释，

只是注释 参数，返回值， 并没有强制约束 


用到的场景：
    
    > 类型检查
      
    > RPC 参数转换


<br>

## 1 使用 enforce 模块来强制约束


注意：

    Python versions 3.5.2 and earlier (3.5.0-3.5.2) are now deprecated. 
    Only Python versions 3.5.3+ would be supported. 
 
 
 [enforce 文档地址](https://github.com/RussBaz/enforce)   
    
```python

    pip install  enforce
    (测试使用)
  
  
  1 装饰类
  
      from enforce import runtime_validation
      
      @runtime_validation
      class DoTheThing(object):
          def __init__(self):
              self.do_the_stuff(5, 1) //  报错，不是参数规定额float
      
          def do_the_stuff(self, a: int, b: float) -> str:
              return str(a * b)
      
      DoTheThing()

  
  2  函数使用
   
    @enforce.runtime_validation
   ... def foo(text: str) -> None:
   ...     print(text)


```


