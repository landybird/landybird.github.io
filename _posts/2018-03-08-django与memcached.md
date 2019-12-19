---
title: django&memcached
description: django&memcached
categories:
- python
tags:
- django
---

<br>

# django & memcached：

<br>

## 下载安装

`win` [地址](https://www.newasp.net/soft/63735.html)

`linux` 

先用 `rpm` 查看是否安装 libevent

`rpm -qa|grep libevent`
`rpm -ql libevent` 查看路径

安装libevent
    
    下载
        `wget http://www.monkey.org/~provos/libevent-1.4.12-stable.tar.gz`    
    
    解压
    
        `tar zxvf libevent-1.4.12-stable.tar.gz -C /usr/local/ `
    
    进入解压后的目录
    
        `cd libevent-1.4.12-stable/  `
    
    配置编译、安装
    
        ./configure -prefix=/usr/libevent  
        make  
        make install  


下载memcached


## 启动 memcached


    命令行
    
        -d是启动一个守护进程；
        -m是分配给Memcache使用的内存数量，单位是MB；
        -u是运行Memcache的用户；
        -l是监听的服务器IP地址，可以有多个地址；
        -p是设置Memcache监听的端口，，最好是1024以上的端口；
        -c是最大运行的并发连接数，默认是1024；
        -P是设置保存Memcache的pid文件



`# /usr/bin/memcached  -d -m 1024 -u root -l 127.0.0.1 -p 11211 -c 1024 -P /tmp/memcached.pid`



## django中配置 缓存


```python

setting.py 
    
      
    CACHES = {
        'default': {
            'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
            'LOCATION': [
                '127.0.0.1:11211',
            ]
        }
    }
    

在urls.py 中指定缓存(或者在视图views.py函数中加上装饰器)

    from django.views.decorators.cache import cache_page
    
  # 获取广告文案模版列表
    re_path(
        r'getCopyTemplate/',
        # CopyTemplateView.as_view(),
        cache_page(60 * 15)(CopyTemplateView.as_view()),
        name='CopyTemplate'),

   
   效果 --->> 不加缓存平均 200ms ，加上之后变成 20ms
   



```

只缓存某个字段

```python

from django.core.cache import cache

def test(request):
    key = 'cache_key'
    time = 60
    result = cache.get(key)
    if not result:
        result = ""
        cache.set(key, value, time)
    return result
        

```

