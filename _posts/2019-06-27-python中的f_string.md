---
title: python 3.6 中的 f string 字面量格式化字符串
description: python 3.6 中的 f string 字面量格式化字符串
categories:
- python
tags:
- python
---



> `python3.6+` 的变化 


- `异步生成器、异步推导语法` (async def , async wait)

- `Path 模块`

- `f string` 格式化 `字面量格式化字符串`






> `python 3.6 +` 格式化的方式


    ％-formatting     ----   "%s" % (string)
    
        冗长的，会导致错误
        不能正确显示元组或字典
    
    

```python

name = 'Xiaoming'
print('Hello %s' % name) 


id = 123
print('User[%s]: %s' % (id, name))

# 字典参数
print('User[%(id)s]: %(name)s' % {'id': 123, 'name': 'Xiaoming'})

```    
  


    string.format（） ----   "{}".format(string)
    
    
```python

name = 'Xiaoming'
'Hello {}'.format(name)



# 通过位置访问：

'{0}, {1}, {2}'.format('a', 'b', 'c')
'a, b, c'

'{2}, {1}, {0}'.format('a', 'b', 'c')
'c, b, a'


# 通过关键字访问：

'Hello {name}'.format(name='Xiaoming')
'Hello Xiaoming'


# 通过对象属性访问：

from collections import namedtuple
p = Point(11, y=22)
'X: {0.x};  Y: {0.y}'.format(p)
'X: 11;  Y: 22'


# 通过下标访问：

coord = (3, 5)
'X: {0[0]};  Y: {0[1]}'.format(coord)
'X: 3;  Y: 5'

```
    
    
    f-Strings 
     # coding: future_fstrings
    
    
```python

# 格式

f ' <text> { <expression> <optional !s, !r, or !a> <optional : format specifier> } <text> ... '


# 基本用法

name = "Tom"
age = 3
f"His name is {name}, he's {age} years old."
"His name is Tom, he's 3 years old."


# 支持表达式

>>> f'He will be { age+1 } years old next year.'
>>> 'He will be 4 years old next year.'

# 对象操作
>>> spurs = {"Guard": "Parker", "Forward": "Duncan"}
>>> f"The {len(spurs)} players are: {spurs['Guard']} the guard, and {spurs['Forward']} the forward."
>>> 'The 2 players are: Parker the guard, and Duncan the forward.'

>>> f'Numbers from 1-10 are {[_ for _ in range(1, 11)]}'
>>> 'Numbers from 1-10 are [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]'


# 排版格式

>>> def show_players():
    print(f"{'Position':^10}{'Name':^10}")
    for player in spurs:
        print(f"{player:^10}{spurs[player]:^10}")
>>> show_players()


 Position    Name   
  Guard     Parker  
 Forward    Duncan
 
 
 
# 数字操作 


# 小数精度
>>> PI = 3.141592653
>>> f"Pi is {PI:.2f}"
>>> 'Pi is 3.14'

# 进制转换
>>> f'int: 31, hex: {31:x}, oct: {31:o}'
'int: 31, hex: 1f, oct: 37'



# 与原始字符串联合使用

 >>> fr'hello\nworld'
'hello\\nworld'

```



> 注意事项 

- `{}`内不能包含反斜杠` \ `


```python


SyntaxError: f-string expression part cannot include a backslash

# 而应该使用不同的引号，或使用三引号。
>>> f"His name is {'Tom'}"
'His name is Tom'

```


- 不能与`'u'`联合使用

'u'是为了与`Python2.7`兼容的，而Python2.7不会支持f-strings，因此与'u'联合使用不会有任何效果

- 插入大括号
    
```   
    >>> f"{{ {str(1)} }}"
    '{ 1 }'
    >>> f"{{ str(1) }}"
    '{ str(1) }
```

> 与str.format()的一点不同

使用str.format()，非数字索引将自动转化为字符串，而f-strings则不会。

```
    >>> "Guard is {spurs[Guard]}".format(spurs=spurs)
    'Guard is Parker'
    
    >>> f"Guard is {spurs[Guard]}"
    Traceback (most recent call last):
      File "<pyshell#34>", line 1, in <module>
        f"Guard is {spurs[Guard]}"
    NameError: name 'Guard' is not defined
    
    >>> f"Guard is {spurs['Guard']}"
    'Guard is Parker'
```


[Format String Syntax](https://docs.python.org/3/library/string.html#formatstrings)
