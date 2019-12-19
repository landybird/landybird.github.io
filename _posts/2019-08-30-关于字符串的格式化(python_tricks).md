---
title: 关于字符串的格式化
description: 关于字符串的格式化
categories:
- python
tags:
- python tricks
---

<br>

旧版本的格式化


```python
errno = 50159747054
name = 'Bob'


# 1 
'Hey %s, there is a 0x%x error!' % (name, errno)


#2 
'Hey %(name)s, there is a 0x%(errno)x error!' % 
{ "name": name,
 "errno": errno }

```

新版本的格式化


```python

# 1
'Hey {name}, there is a 0x{errno:x} error!'.format(name=name,
 errno=errno)


#2  py3.6+  更快

uses the BUILD_STRING opcode
f'Hey {name}, there is a 0x{errno:x} error!'


```

来看一下时间的快慢，和代码的执行过程


```python

errno = 50159747054
name = 'Bob'


def foo1():
    return f'Hey {name}, there is a 0x{errno:x} error!'

def foo2():
    return  'Hey {name}, there is a 0x{errno:x} error!'.format(name=name,
 errno=errno)


import timeit

t1 = timeit.Timer('foo1()', "from __main__ import foo1")
t2 = timeit.Timer('foo2()', "from __main__ import foo2")
print(t1.timeit())  # 0.8687919000000001

print(t2.timeit())  # 1.2195221999999997





import dis
dis.dis(foo1)

  5           0 LOAD_CONST               1 ('Hey ')
              2 LOAD_GLOBAL              0 (name)
              4 FORMAT_VALUE             0
              6 LOAD_CONST               2 (', there is a 0x')
              8 LOAD_GLOBAL              1 (errno)
             10 LOAD_CONST               3 ('x')
             12 FORMAT_VALUE             4 (with format)
             14 LOAD_CONST               4 (' error!')
             16 BUILD_STRING             5
             18 RETURN_VALUE


dis.dis(foo2)

  8           0 LOAD_CONST               1 ('Hey {name}, there is a 0x{errno:x} error!')
              2 LOAD_ATTR                0 (format)
              4 LOAD_GLOBAL              1 (name)

  9           6 LOAD_GLOBAL              2 (errno)
              8 LOAD_CONST               2 (('name', 'errno'))
             10 CALL_FUNCTION_KW         2
             12 RETURN_VALUE


```

