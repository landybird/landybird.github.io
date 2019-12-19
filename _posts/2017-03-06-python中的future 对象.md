---
title: python的future对象
description: python的future对象
categories:
 - python
tags:
- python基础
---


# python的future对象

<br>

**`future 对象`是 `concurrent.futures 模块` 和 `asyncio 包`的重要组件**

>concurrent.futures.Future 和 asyncio.Future

这两个类的作用相同：两个 Future 类的实例都`表示可能已经完成或者尚未完成的延迟计算`。

这与 Twisted 引擎中的 Deferred 类、Tornado 框架中的Future 类，以及多个 JavaScript 库中的 Promise 对象类似。
