---
title: python中pathlib模块处理文件路径
description: python中pathlib模块处理文件路径
categories:
- python
tags:
- python基础
---

<br>


> `python3.4-` 之前路径相关的操作都放在 `os`中， 基本集中在 `o.path`


#### `os(os.path)` 与 `pathlib` 的对比




<table>
<thead><tr>
<th>os and os.path</th>
<th>pathlib</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>os.path.abspath</code></td>
<td><code>Path.resolve</code></td>
</tr>
<tr>
<td><code>os.chmod</code></td>
<td><code>Path.chmod</code></td>
</tr>
<tr>
<td><code>os.mkdir</code></td>
<td><code>Path.mkdir</code></td>
</tr>
<tr>
<td><code>os.rename</code></td>
<td><code>Path.rename</code></td>
</tr>
<tr>
<td><code>os.replace</code></td>
<td><code>Path.replace</code></td>
</tr>
<tr>
<td><code>os.rmdir</code></td>
<td><code>Path.rmdir</code></td>
</tr>
<tr>
<td><code>os.remove</code>, <code>os.unlink</code></td>
<td><code>Path.unlink</code></td>
</tr>
<tr>
<td><code>os.getcwd</code></td>
<td><code>Path.cwd</code></td>
</tr>
<tr>
<td><code>os.path.exists</code></td>
<td><code>Path.exists</code></td>
</tr>
<tr>
<td><code>os.path.expanduser</code></td>
<td><code>Path.expanduser</code> and <code>Path.home</code></td>
</tr>
<tr>
<td><code>os.path.isdir</code></td>
<td><code>Path.is_dir</code></td>
</tr>
<tr>
<td><code>os.path.isfile</code></td>
<td><code>Path.is_file</code></td>
</tr>
<tr>
<td><code>os.path.islink</code></td>
<td><code>Path.is_symlink</code></td>
</tr>
<tr>
<td><code>os.stat</code></td>
<td><code>Path.stat</code>, <code>Path.owner</code>, <code>Path.group</code></td>
</tr>
<tr>
<td><code>os.path.isabs</code></td>
<td><code>PurePath.is_absolute</code></td>
</tr>
<tr>
<td><code>os.path.join</code></td>
<td><code>PurePath.joinpath</code></td>
</tr>
<tr>
<td><code>os.path.basename</code></td>
<td><code>PurePath.name</code></td>
</tr>
<tr>
<td><code>os.path.dirname</code></td>
<td><code>PurePath.parent</code></td>
</tr>
<tr>
<td><code>os.path.samefile</code></td>
<td><code>Path.samefile</code></td>
</tr>
<tr>
<td><code>os.path.splitext</code></td>
<td><code>PurePath.suffix</code></td>
</tr>
</tbody>
</table>



```python
import os
from pathlib import Path

print(os.path.abspath(__file__))
print(Path(__file__).resolve())


print(os.path.isdir(os.getcwd()))
print(Path.cwd().is_dir())

print(os.path.basename(os.path.abspath(__file__)))
print(Path(__file__).resolve().name)



```

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

#### `/` 用来拼接 路径 vs `os.path.join`



```python

import os
from pathlib import Path

p = os.path.join("home", "your/code")

p2 = Path("/").joinpath("home", "your/code") # /home/your/code

p3 = "/" / Path("home") / Path("your/code")
p1 = Path("home") / Path("your/code")
p3 = Path("/") / Path("home") / Path("your/code")


print(p3)

```



#### 文件的前后缀  `suffix/stem`


    Path().suffix
    Path().stem
    

```python

import os
from pathlib import Path

base = os.path.basename(os.path.abspath(__file__))
stem, suffix = os.path.splitext(base)
print(stem, suffix  )



base = Path(__file__).resolve()
print(base.suffix, base.stem)


# 当文件有多个后缀，可以用suffixes返回文件所有后缀列表:


Path('my.tar.bz2').suffixes
['.tar', '.bz2']

```



#### 其他的使用方法


> `touch` 方法 

    Path().touch()

Python 语言没有内置创建文件的方法 (linux 下的  touch 命令)


```python
# 过去需要使用文件句柄操作
with open('new.txt', 'a') as f:
    ...
    
# 可以直接使用 Path
Path('new.txt').touch()

```

touch 接受`mode参数`，能够在创建时确认`文件权限`，
还能通过`exist_ok`参数方式确认是否可以`重复 touch` (默认可以重复创建，会更新文件的 mtime)


> `home`  获取用户的 home 目录

    Path.home()


```python

os.path.expanduser('~')

Path.home()

```

> `owner`  获取当前文件的用户 `*nux系统`

```python

import pwd
pwd.getpwuid(os.stat('/usr/local/etc/my.cnf').st_uid).pw_name


p.owner()

```

> 创建多级目录 `Path().mkdir(parents=True)`

```python

Path('1/2/3').mkdir(parents=True)

```






[https://www.dongwm.com/post/use-pathlib/](https://www.dongwm.com/post/use-pathlib/)