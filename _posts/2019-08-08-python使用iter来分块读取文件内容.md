---
title: python使用iter来分块读取文件内容
description: python使用iter来分块读取文件内容
categories:
- python
tags:
- python基础
---

<br>

#### python使用iter来分块读取文件内容

当读取的文件一行的内容过多的时候， 使用生成器可以减少内存消耗

```python

def chunked_file_reader(fp, block_size=1024 * 8):
    """生成器函数：分块读取文件内容
    """
    while True:
        chunk = fp.read(block_size)
        # 当文件没有更多内容时，read 调用将会返回空字符串 ''
        if not chunk:
            break
        yield chunk


def count_nine_v3(fname):
    count = 0
    with open(fname, encoding='utf-8'') as fp:
        for chunk in chunked_file_reader:
            count += chunk.count('一')
    return count
```

也可以使用iter函数 


iter(iterable) 是一个用来构造迭代器的内建函数 , 使用 iter(callable, sentinel) 的方式调用它时，会返回一个特殊的对象，迭代它将不断产生可调用对象 callable 的调用结果，直到结果为 setinel 时，迭代终止

```python

from functools import partial

def count_nine_v3(fname):
    count = 0
    with open(fname, encoding='utf-8') as fp:
        for chunk in iter(partial(fp.read, 1024*8), ''):
           count += chunk.count('a')
    return count

```