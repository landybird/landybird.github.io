---
title: python中pathlib模块处理文件路径
description: python中pathlib模块处理文件路径
categories:
- python
tags:
- python基础
---

<br>

#### python中pathlib模块处理文件路径

`py3.6 +`


之前的文件处理都是使用 `os` + `os.path`

```python  

#  修改文件后缀


base_dir = os.path.dirname(os.path.abspath(__file__))


1 ）
import os, sys

for filename in  os.listdir(base_dir):
     base_name, ext = os.path.splitext(filename)
     if ext == '.txt':
        os.rename(filename, os.path.join(path, f'{basename}.csv'))
         
     
2） 

from pathlib import Path

for fpath in Path(base_dir).glob('*.txt'):
    fpath.rename(fpath.with_suffix('.csv'))




#  组合文件路径


1) 
path_ = os.path.join(base_dir, 'foo.txt')

2) 
path__ = Path(base_dir) / 'foo.txt'



#  读取文件内容

1） with open('foo.txt', encoding='utf-8') as file:
       print(file.read())

2） print(Path('foo.txt').read_text(encoding='utf-8'))


```
