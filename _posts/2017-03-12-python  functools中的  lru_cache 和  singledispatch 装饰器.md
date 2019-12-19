---
title: python  functools中的  lru_cache 和  singledispatch 装饰器
description: python  functools中的  lru_cache 和  singledispatch 装饰器
categories:
 - python
tags:
- python基础
---


# python  functools中的  lru_cache 和  singledispatch 装饰器

<br>


## 1 LRU_CACHE


**除了`优化递归算法`之外，`lru_cache` 在从 `Web 中获取信息的应用`中也能发挥巨大作用。**

用来`做缓存`，能`把相对耗时的函数结果进行保存，避免传入相同的参数重复计算`。同时，缓存并不会无限增长，不用的缓存会被释放。


特别要注意，lru_cache 可以使用两个可选的参数来配置。它的签名是：

    functools.lru_cache(maxsize=128, typed=False)

`maxsize`   参数指定存储多少个调用的结果。缓存满了之后，旧的结果会
被扔掉，腾出空间。为了得到最佳性能，maxsize 应该设为 2 的
幂。

`typed`    参数如果设为 True，把不同参数类型得到的结果分开保


<br>


## 2 singledispatch

`@singledispatch` 装饰的普通函数会变成`泛函数（generic function）`

作用和c++中函数的重载类似

```python
    
    from functools import singledispatch
    
    
    @singledispatch
    def show(obj):
        print (obj, type(obj), "obj")
    
    #字符串
    @show.register(str)
    def _(text):
        print (text, type(text), "str")
    
    #int
    @show.register(int)
    def _(n):
        print (n, type(n), "int")
    
    
    #元祖或者字典均可
    @show.register(tuple)
    @show.register(dict)
    def _(tup_dic):
        print (tup_dic, type(tup_dic), "tup_dic")

```
