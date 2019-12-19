---
title: 小结一下 python中的str 和 repr 函数 和 魔方方法_ _str_ _ , _ _repr_ _
categories:
- python基础
tags:
- python基础
---


<br>

## 1 repr 与 str 函数的实现

- **内置函数repr是通过__repr__这个特殊方法来得到一个对象的字符串表示形式**
- **内置函数str是通过__str__这个特殊方法来得到一个对象的字符串表示形式**


<br>

## 2 二者的区别

- **repr适用于交互式控制台和调试程序，将字符串转换为合法的表达式**

- **而str则是将字符串转换为更容易让用户理解的形式**

- **当然这些都是通过类中的 __str__ 和 __repr__ 来实现的**

```python
# 例1 

import datetime
now  = datetime.datetime.now()

print(str(now))
print(repr(now))
print(eval(repr(now)))

结果

2018-04-12 19:36:35.818188
datetime.datetime(2018, 4, 12, 19, 36, 35, 818188)
2018-04-12 19:36:35.818188

```

<br>

### 3 如果一个对象没有__str__,而Python又需要调用它的时候，解释器会使用__repr__来代替


```python
# 例2 


class MyString(object):
	def __str__(self):
		return 'str则是将字符串转换为更容易让用户理解的形式'

	def __repr__(self):
		return 'repr适用于交互式控制台和调试程序，将字符串转换为合法的表达式'


obj = MyString()
# print(obj)
print(repr(obj))
print(obj)
```

