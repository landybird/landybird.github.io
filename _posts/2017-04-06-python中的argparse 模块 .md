---
title: python中的argparse 模块
description: python中的argparse 模块
categories:
 - python
tags:
- python基础
---


# python中的argparse 模块

<br>


地址 [argparse文档]( https://docs.python.org/3/library/argparse.html)


**2.7 版本之后 已经是`python的默认包`**

<br>

## 1 使用步骤

    创建 parser 对象
    添加参数
    解析参数
    
    ap = argparse.ArgumentParser(description = 'projectA service')
    ap.add_argument('-d', '--execute_dir', type = str, help = 'execute directory', default = basepath)    
    args = ap.parse_args()

<br>

## 2 添加参数  add_argument()常用的参数：

    
    dest：如果提供dest，例如dest="a"，那么可以通过args.a访问该参数
    default：设置参数的默认值
    action：参数出发的动作
    store：保存参数，默认
    store_const：保存一个被定义为参数规格一部分的值（常量），而不是一个来自参数解析而来的值。
    store_ture/store_false：保存相应的布尔值
    append：将值保存在一个列表中。
    append_const：将一个定义在参数规格中的值（常量）保存在一个列表中。
    count：参数出现的次数
    parser.add_argument("-v", "--verbosity", action="count", default=0, help="increase output verbosity")
    version：打印程序版本信息
    type：把从命令行输入的结果转成设置的类型
    choice：允许的参数值
    parser.add_argument("-v", "--verbosity", type=int, choices=[0, 1, 2], help="increase output verbosity")
    help：参数命令的介绍
    
<br>

## 3 实例：


```python

    import argparse
    import os
    
    pwd = os.path.dirname(os.path.realpath(__file__))
    
    parser = argparse.ArgumentParser(description=u"初始化任务")
    
    parser.add_argument("--d", type=str, default=pwd, help = u'制定执行的目录')
    parser.add_argument("--init_field", type=str, default='', help=u'初始化需要计算的字段数据')
    parser.add_argument("--ensure_id", type=str, default='', help=u'制定初始化的ensure_id')
    parser.add_argument("--s", type=str, default='', help=u'stop停止任务')
   
    args = parser.parse_args()
    
    execute_dir = args.d
    init_field = args.init_field
    ensure_id = args.ensure_id
    stop = args.s

```