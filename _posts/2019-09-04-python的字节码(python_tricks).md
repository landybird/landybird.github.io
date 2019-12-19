---
title: python的字节码
description: python的字节码
categories:
- python
tags:
- python tricks
---

<br>


```python
def greet(name):
    return 'Hello, ' + name + '!'
```

#### 函数的 `__code__` 属性

获取函数运行时

    co_code         virtual machine instructions   虚拟机的指令
    
    co_consts      constants    常量
    
    co_varnames    variables    变量


```python

>>> greet.__code__.co_code
b'dx01|x00x17x00dx02x17x00Sx00'

>>> greet.__code__.co_consts
(None, 'Hello, ', '!')

>>> greet.__code__.co_varnames
('name',)

```

#### `反汇编` disassembler 和 `dis.dis()`

```python

dis.dis(greet)


  2           0 LOAD_CONST               1 ('Hello, ')
              2 LOAD_FAST                0 (name)
              4 BINARY_ADD
              6 LOAD_CONST               2 ('!')
              8 BINARY_ADD
             10 RETURN_VALUE




# >>  首先获取 Hello 常量        push -->> stack      push and pop    virtual machine
# >>  首先获取 name 变量         push -->> stack

        0: 'Guido' (contents of "name")    0 是最上面
        1: 'Hello, '

# >>  BINARY_ADD                pop   <<-- stack
# 把两个字符串pop后，组到一起   push -->> stack

        0: 'Hello, Guido'

      

# >>  获取 ! 常量               push -->> stack
        
        0: '!'
        1: 'Hello, Guido'

# >>  BINARY_ADD                pop   <<-- stack
# 把字符串pop后，组到一起       push -->> stack

        0: 'Hello, Guido!'

# >> RETURN_VALUE   可以通过 虚拟机把堆栈上的值返回
```

更多虚拟机相关的内容[Compiler Design: Virtual Machines by Wilhelm and Seidl]