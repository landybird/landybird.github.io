---
title: 代理模型_观察者模式(learning_python_design_patterns)
description: 代理模型_观察者模式(learning_python_design_patterns)
categories:
- python
tags:
- learning python design patterns
---

<br>


#### Proxy  代理模式 -- `lazy initialization`

![](https://landybird.github.io/assets/images/lpdp4.png)

```python
from abc import ABCMeta, abstractmethod
import random

class AbstractSubject(object):
    """A common interface for the real and proxy objects."""
    __metaclass__ = ABCMeta

    @abstractmethod
    def sort(self, reverse=False):
        pass


class RealSubject(AbstractSubject):
    """A class for a heavy object which takes a lot of memory
    space and takes some time to instantiate."""
    def __init__(self):
        self.digits = []
        for i in range(10000000):
            self.digits.append(random.random())

    def sort(self, reverse=False):
        self.digits.sort()
        if reverse:
            self.digits.reverse()


class Proxy(AbstractSubject):
    """A proxy which has the same interface as RealSubject."""
    reference_count = 0

    def __init__(self):
        """A constructor which creates an object if it is not exist and
        caches it otherwise."""
        if not getattr(self.__class__, 'cached_object', None):
            self.__class__.cached_object = RealSubject()
            print('Created new object')
        else:
            print('Using cached object')
        self.__class__.reference_count += 1
        print('Count of references = ', self.__class__.reference_count)

    def sort(self, reverse=False):
        """The args are logged by the Proxy."""
        print('Called sort method with args:')
        print(locals().items())
        self.__class__.cached_object.sort(reverse=reverse)

    def __del__(self):
        """Decreases a reference to an object, if the number of
         references is 0, delete the object."""
        self.__class__.reference_count -= 1
        if self.__class__.reference_count == 0:
            print('Number of reference_count is 0. Deleting cached object...')
            del self.__class__.cached_object
        print('Deleted object. Count of objects = ', self.__class__.reference_count)



if __name__ == '__main__':
    proxy1 = Proxy()
    print()
    proxy2 = Proxy()
    print()
    proxy3 = Proxy()
    print()
    proxy1.sort(reverse=True)
    print()
    print('Deleting proxy2')
    del proxy2
    print()
    print('The other objects are deleted upon program termination')



Created new object
Count of references =  1

Using cached object
Count of references =  2

Using cached object
Count of references =  3

Called sort method with args:
dict_items([('reverse', True), ('self', <__main__.Proxy object at 0x000001FF447D0F60>)])

Deleting proxy2
Deleted object. Count of objects =  2

The other objects are deleted upon program termination
Deleted object. Count of objects =  1
Number of reference_count is 0. Deleting cached object...
Deleted object. Count of objects =  0


```


#### Observer design pattern 观察者模式 (消息订阅模式)


 `publishing-subscriber`


    多个博客的订阅
    
    鼠标事件的监听处理
    
    手机 app 接受信息
    
![](https://landybird.github.io/assets/images/lpdp5.png)

    
```python

import time

class Subject(object):
    def __init__(self):
        self.observers = []
        self.cur_time = None


    def register_observer(self, observer):
        if observer in self.observers:
            print(observer, 'already in subscribed observers')
        else:
            self.observers.append(observer)

    def unregister_observer(self, observer):
        try:
            self.observers.remove(observer)
        except ValueError:
            print('No such observer in subject')


    def notify_observers(self):
        self.cur_time = time.time()
        for observer in self.observers:
            observer.notify(self.cur_time)




from abc import ABCMeta, abstractmethod
import datetime

class Observer(object):
    """Abstract class for observers, provides notify method as
    interface for subjects."""
    __metaclass__ = ABCMeta
    @abstractmethod
    def notify(self, unix_timestamp):
        pass
    # And a couple of concrete observer derived from abstract observer.
    # They need to implement notify method. This method will take UNIX
    # timestamp converts it to 12H or 24H format and print it to
    # standard out.
class USATimeObserver(Observer):

    def __init__(self, name):
        self.name = name

    def notify(self, unix_timestamp):
        time = datetime.datetime.fromtimestamp(int(unix_timestamp)).strftime('%Y-%m-%d %I:%M:%S %p')
        print('Observer', self.name, 'says:', time)


class EUTimeObserver(Observer):
    def __init__(self, name):
        self.name = name

    def notify(self, unix_timestamp):
        time = datetime.datetime.fromtimestamp(int(unix_timestamp)).strftime('%Y-%m-%d %H:%M:%S')
        print('Observer', self.name, 'says:', time)



if __name__ == '__main__':
     subject = Subject()
     print ('Adding usa_time_observer')
     observer1 = USATimeObserver('usa_time_observer')
     subject.register_observer(observer1)
     subject.notify_observers()
     time.sleep(2)
     print('Adding eu_time_observer')
     observer2 = EUTimeObserver('eu_time_observer')
     subject.register_observer(observer2)
     subject.notify_observers()
     time.sleep(2)
     print('Removing usa_time_observer')
     subject.unregister_observer(observer1)
     subject.notify_observers()



Adding usa_time_observer
Observer usa_time_observer says: 2019-09-17 04:41:32 PM
Adding eu_time_observer
Observer usa_time_observer says: 2019-09-17 04:41:34 PM
Observer eu_time_observer says: 2019-09-17 16:41:34
Removing usa_time_observer
Observer eu_time_observer says: 2019-09-17 16:41:36


```
    



