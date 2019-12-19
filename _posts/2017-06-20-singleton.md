---
title: python的单例模式
description: 解决多线程的单例模式失效
categories:
 - python
tags:
 - 设计模式
---

#  单例模式

------

<br>

## 1 单例模式的概念（Singleton Pattern）

**是一种常用的软件设计模式**，**主要目的是确保某一个类只有一个实例存在**。

希望`某个类只能出现一个实例`时，单例对象就能派上用场

   比如，某个服务器程序的配置信息存放在一个文件中，客户端通过一个 AppConfig 的类来读取配置文件的信息。如果在程序运行期间，有很多地方都需要使用配置文件的内容，也就是说，很多地方都需要创建 AppConfig 对象的实例，这就导致系统中存在多个 AppConfig 的实例对象，而这样会严重浪费内存资源，尤其是在配置文件内容很多的情况下。事实上，类似 AppConfig 这样的类，我们希望在程序运行期间只存在一个实例对象。

使用需求: `程序运行起来 只需要一份`

	比如：	admin的register

		数据库连接，数据库连接池

		(全局变量)


	全局变量  --- 模块导入 (使用同一个数据)

	单例模式 


**单例模式的应用---数据库连接池**

```python
class SingleDBpool(object):
def __init__(self):
    self.pool = ...

def __new__(cls, *args, **kwargs):
    if not hasattr(cls,'_instance'):
	cls._instance = super(SingleDBpool,cls).__new__(*args, **kwargs)
    return cls._instance

def connect(self):
    return self.pool.connection()
```


<br>

## 2 Python 中，单例模式的实现：

<br>


###  (1)使用模块

Python 的模块就是天然的单例模式，因为模块在第一次导入时，会生成 .pyc 文件，当第二次导入时，就会直接加载 .pyc 文件，而不会再次执行模块代码。因此，我们只需把相关的函数和数据定义在一个模块中，就可以获得一个单例对象了。



```python	
# mysingleton.py
class My_Singleton(object):
    def foo(self):
	pass

my_singleton = My_Singleton()
```

将上面的代码保存在文件 mysingleton.py 中，然后这样使用：

```python	
from mysingleton import my_singleton

my_singleton.foo()
```

<br>

###  (2)使用 __new__

为了使类只能出现一个实例，我们可以使用 __new__ 来控制实例的创建过程，代码如下：

```python	
class Singleton(object):
    _instance = None
    def __new__(cls, *args, **kw):
	if not cls._instance:
	    cls._instance = super(Singleton, cls).__new__(cls, *args, **kw)  
	return cls._instance  

class MyClass(Singleton):  
    a = 1
```
	
<br>

### (3)使用类方法 

**但是每次调用会很繁琐 A.get_instance(params)**  `无法支持多线程`

```python
class A(object):

instance = None
def __init__(self,name):
    self.name = name

@classmethod
def get_instance(cls,*args,**kwargs):
    if not cls.instance:
	cls.instance = cls(*args,**kwargs)
    return cls.instance


a = A.get_instance('aaa')
b = A.get_instance('bbb')
print(a.name)
```

    
**多线程下，为什么会失效？**

```python
import threading

class Single(object):
instance = None

def __init__(self):
    import time
    time.sleep(0.5)     # 有延时的情况
    pass

@classmethod
def get_instance(cls,*args,**kwargs):
    if not cls.instance:
	cls.instance = cls(*args,**kwargs)
    return cls.instance


def task(arg):
obj = Single.get_instance()
print(obj)

for i in  range(5):
t = threading.Thread(target=task,args=[i,])
t.start()

结果：创建了不同的对象 -- 失效

# <__main__.Single object at 0x00000000029B8F60>
# <__main__.Single object at 0x00000000029B8E80>
# <__main__.Single object at 0x000000000299BD68>
# <__main__.Single object at 0x00000000029F2BE0>
# <__main__.Single object at 0x00000000029C2B38>
```

**如何解决多线程下单例的失效?**

```python
>>   加线程锁

import threading
import time

class Single(object):
instance = None
_threading_lock = threading.Lock()

def __init__(self):
    time.sleep(0.5)

@classmethod
def get_instance(cls,*args,**kwargs):
	if not cls.instance:           # 先判断是否存在(如果存在，说明不是多线程，直接获取)
	    with cls._threading_lock:  # 加锁，只有一个线程进入，然后判断 单例是否存在
		if not cls.instance:  # 先判断是否存在(如果存在，说明不是多线程，直接获取)
		    cls.instance = cls(*args,**kwargs)
		return cls.instance
	return cls.instance

def task(arg):
obj = Single.get_instance()
print(obj)

for i in  range(5):
t = threading.Thread(target=task,args=[i,])
t.start()

time.sleep(5)
obj = Single.get_instance()
print(obj)
```

<br>

### (4)使用装饰器

**可以使用装饰器来装饰某个类，使其只能生成一个实例**

```python	
def Singleton(cls):
instance = []
def inner(*args,**kwargs):
    if cls(*args,**kwargs) not in instance:
	instance.append(cls(*args,**kwargs))
    return instance[0]
return inner

@Singleton
class A(object):
pass

a = A()
b = A()
print(a == b)
```

<br>

### (5)使用 metaclass


   
**对象和类创建的 完整流程 ：**

```python
class F:
    pass

    1 执行type的 init 方法(类是type的对象)

obj = F()
    2 执行type的 call 方法
	2.1 调用 F类的 new 方法 (创建对象)
	2.2 调用 F类的 init 方法 (对象初始化)
obj()
    3 执行 F的 call 方法


#  继承 type 类(模拟重写 type -- 用于创建类)

class Single(type):

def __init__(self,*args,**kwargs):
    super(Single,self).__init__(*args,**kwargs)

def __call__(cls, *args, **kwargs):
    obj = cls.__new__(cls,*args, **kwargs)
    cls.__init__(obj,*args, **kwargs)
    return obj

# 用伪type 创建Foo类
class Foo(metaclass=Single): # 通过 Single 创建

def __init__(self,name):
    self.name= name

def __new__(cls, *args, **kwargs):
    return object.__new__(cls,*args, **kwargs)
```




**元类（metaclass）可以控制类的创建过程，它主要做三件事：**

        拦截类的创建
        修改类的定义
        返回修改后的类

**使用元类实现单例模式：**
	
```python	
class Single(type):            # 

    def __call__(cls, *args, **kwargs):
	if not hasattr(cls,'_instance'):
	    cls._instance  = cls.__new__(cls,*args, **kwargs)
	return cls._instance

class Foo(metaclass=Single): # 通过 Single 创建
    pass

这样 通过创建Single类，以后需要单例模式的类，可以指定 用 它来创建就可以了
```

**`Python 的模块`是天然的`单例模式`，这在大部分情况下应该是够用的，也可以使用装饰器、元类等方法**


**metaclass补充**
    
                class MyType(type):
            def __init__(self, *args, **kwargs):
                super(MyType, self).__init__(*args, **kwargs)
        
            def __call__(cls, *args, **kwargs):
                return super(MyType, cls).__call__(*args, **kwargs)
        
        
        def with_metaclass(base):
            return MyType('XX', (base,), {})
        
        
        class Foo(with_metaclass(object)):
            pass
