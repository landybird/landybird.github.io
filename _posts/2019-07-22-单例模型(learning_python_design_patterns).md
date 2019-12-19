---
title: 单例模型(learning_python_design_patterns)
description: 单例模式(learning_python_design_patterns)
categories:
- python
tags:
- learning python design patterns
---

<br>

`Singleton`

很多情况下 在程序运行周期中， 至始至终只需要一个数据实例

    类对象， 一个list或者dict
    
    
> 什么时候使用

    控制 并发请求共享资源
    
    系统资源的全局访问
    
    只用一个实例对象
    

> 几个典型的使用场景


  
    • The logging class and its subclasses (global point of access for the logging
      class to send messages to log)
    
    • Printer spooler (your application should only have a single instance of the
      spooler in order to avoid having a conflicting request for the same resource)
      
    • Managing a connection to a database
    
    • File manager
    
    • Retrieving and storing information on external configuration files
    
    • Read-only singletons storing some global states (user language, time, time
       zone, application path, and so on)   
    
      

#### A module-level singleton  python模块是单例的

下面是python一个模块的导入过程
    
    1. Check whether a module is already imported.
    2. If yes, return it.
    3. If not, find a module, initialize it, and return it.
    4. Initializing a module means executing code, including all module-level
    assignments. When you import the module for the first time,
    all initializations are done; however, if you try to import the module for
    the second time, Python will return the initialized module.
    Thus, the initialization will not be done, and you get a previously imported
    module with all of its data
    

所以在python中 可以把数据作为模块的属性， 来实现单例模式

```python


# singletone.py:
 only_one_var = "I'm only one var"
 
# module1.py:
import singletone
print singleton.only_one_var
singletone.only_one_var += " after modification"
import module2

# module2.py:
import singletone
print singleton.only_one_var

```


缺点

    容易出错
    
    占用 module 本身的 namespace
    
    不支持懒加载， 每次导入都会全部加载
    
    • It's ugly, especially if you have a lot of objects that should remain as singletons.
    • It pollutes the module namespace with unnecessary variables.
    • They don't permit lazy allocation and initialization; all global variables will
      be loaded during the module import process.
    • It's not possible to reuse the code because you cannot use the inheritance.
    • It has no special methods and no object-oriented programming benefits at all
    


####  A classic singleton  类单例模式

在创建实例的时候， 检查是否存在实例 (作为类的一个属性), 没有便创建返回



```python

class Singleton(object):
    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, 'instance'):
            cls.instance = super(Singleton, cls).__new__(cls, *args, **kwargs)
        return cls.instance


single1 = Singleton()
single2 = Singleton()

print(single1 is single2)

single1.only_one_var = 'only one'

print(single2.only_one_var)

```

> 当然， 继承之后的类也是单例模式

```python

class Sub(Singleton):
    pass
```


但是不是单态模式

```python

s = Sub()

print(s is single1)
# False

```

#### The borg singleton  单态模式`monostate`

所有的实例都不同， 但共享相同的属性

```python

class Borg(object):
    _shared_state = {}

    def __new__(cls, *args, **kwargs):
        obj = super(Borg, cls).__new__(cls, *args, **kwargs)
        obj.__dict__ = cls._shared_state
        return obj



class Child(Borg):
    pass

c = Child()

b = Borg()

print(b is c)  # False
b.only_one_var = "only one"
print(vars(b))    
print(c.__dict__)

```

不需要共享， 在繼承的時候重新定義`_shared_state = {}`

```python
class Borg(object):
    _shared_state = {}

    def __new__(cls, *args, **kwargs):
        obj = super(Borg, cls).__new__(cls, *args, **kwargs)
        obj.__dict__ = cls._shared_state
        return obj



class Child(Borg):
    _shared_state = {}

c = Child()

b = Borg()

print(b is c)
b.only_one_var = "only one"
print(vars(b))
print(c.__dict__)

```

實例

```python

import httplib2
import os
import re
import threading
import urllib
from urlparse import urlparse, urljoin

from BeautifulSoup import BeautifulSoup

#  Singleton
class Singleton(object):
    def __new__(cls):
    if not hasattr(cls, 'instance'):
        cls.instance = super(Singleton, cls).__new__(cls)
    return cls.instance


#  class for creating a thread.
class ImageDownloaderThread(threading.Thread):
    def __init__(self, thread_id, name, counter):
        threading.Thread.__init__(self)
        self.name = name
    def run(self):
        print ('Starting thread ', self.name)
        download_images(self.name)
        print ('Finished thread ', self.name)


# BFS algorithm, finds links 深度優先

def traverse_site(max_links=10):
    link_parser_singleton = Singleton()
    # While we have pages to parse in queue
    while link_parser_singleton.queue_to_parse:
    # If collected enough links to download images, return
        if len(link_parser_singleton.to_visit) == max_links:
            return
        url = link_parser_singleton.queue_to_parse.pop()
        http = httplib2.Http()
        try:
            status, response = http.request(url)
        except Exception:
            continue

        # Skip if not a web page
        if status.get('content-type') != 'text/html':
            continue
        # Add the link to queue for downloading images
        link_parser_singleton.to_visit.add(url)
        print('Added', url, 'to queue')
        bs = BeautifulSoup(response)
        for link in BeautifulSoup.findAll(bs, 'a'):
            link_url = link.get('href')
        # <img> tag may not contain href attribute
            if not link_url:
                continue
            parsed = urlparse(link_url)
        # If link follows to external webpage, skip it
            if parsed.netloc and parsed.netloc != parsed_root.netloc:
                continue
        # Construct a full url from a link which can be relative
            link_url = (parsed.scheme or parsed_root.scheme) + '://' + (parsed.netloc or parsed_root.netloc) + parsed.path or ''
        # If link was added previously, skip it
            if link_url in link_parser_singleton.to_visit:
                continue
        # Add a link for further parsing
            link_parser_singleton.queue_to_parse = [link_url] + link_parser_singleton.queue_to_parse



# 
def download_images(thread_name):
    singleton = Singleton()
    # While we have pages where we have not download images
    while singleton.to_visit:
        url = singleton.to_visit.pop()
        http = httplib2.Http()
        print(hread_name, 'Starting downloading images from', url)
        try:
            status, response = http.request(url)
        except Exception:
            continue
        bs = BeautifulSoup(response)
        # Find all <img> tags
        images = BeautifulSoup.findAll(bs, 'img')
        for image in images:
        # Get image source url which can be absolute or relative
            src = image.get('src')
        # Construct a full url. If the image url is relative,
        # it will be prepended with webpage domain.
        # If the image url is absolute, it will remain as is
            src = urljoin(url, src)
        # Get a base name, for example 'image.png' to name file locally
            basename = os.path.basename(src)
            if src not in singleton.downloaded:
                singleton.downloaded.add(src)
                print('Downloading', src)
        # Download image to local filesystem
                urllib.urlretrieve(src, os.path.join('images', basename))
        print(thread_name, 'finished downloading images from', url)


# main.py


if __name__ == '__main__':
    root = 'http://python.org'
    parsed_root = urlparse(root)
    singleton = Singleton()
    singleton.queue_to_parse = [root]
    # A set of urls to download images from
    singleton.to_visit = set()
    # Downloaded images
    singleton.downloaded = set()
    traverse_site()
    # Create images directory if not exists
    if not os.path.exists('images'):
        os.makedirs('images')
    # Create new threads
    thread1 = ImageDownloaderThread(1, "Thread-1", 1)
    thread2 = ImageDownloaderThread(2, "Thread-2", 2)
    # Start new Threads
    thread1.start()
    thread2.start()
```