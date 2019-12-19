---
title: python的dataclasses模块
description: python的dataclasses模块
categories:
- python
tags:
- python基础
---

<br>

#### python的dataclasses模块

`py3.7+`

有如下的数据

```python

# 去过普吉岛的人员数据
users_visited_phuket = [
    {"first_name": "Sirena", "last_name": "Gross", "phone_number": "650-568-0388", "date_visited": "2018-03-14"},
    {"first_name": "James", "last_name": "Ashcraft", "phone_number": "412-334-4380", "date_visited": "2014-09-16"},
    ... ...
]

# 去过新西兰的人员数据
users_visited_nz = [
    {"first_name": "Justin", "last_name": "Malcom", "phone_number": "267-282-1964", "date_visited": "2011-03-13"},
    {"first_name": "Albert", "last_name": "Potter", "phone_number": "702-249-3714", "date_visited": "2013-09-11"},
    ... ...
]

```

找到去过普吉岛但是没去过新西兰的人


- 0  使用`两层for循环`, 复杂度为 O(n*m)

- 1  利用`集合`的特性， 复杂度为 O(n+m)

```python

def find_potential_customers():
    nz_records_idx = {
        (rec['first_name'], rec['last_name'], rec['phone_number'])
        for rec in users_visited_nz
    }

    for rec in users_visited_phuket:
        key = (rec['first_name'], rec['last_name'], rec['phone_number'])
        if key not in nz_records_idx:
            yield rec

```

- 3 新建一个类， 包装成可以hash的对象, 放在set里面求差集

```python

class VisitRecord:
    """旅游记录
    """
    def __init__(self, first_name, last_name, phone_number, date_visited):
        self.first_name = first_name
        self.last_name = last_name
        self.phone_number = phone_number
        self.date_visited = date_visited
        
    # 定义 hash 使对象可hash
    def __hash__(self):
        return hash(
            (self.first_name, self.last_name, self.phone_number)
        )
        
    # 定义 eq 使对象可 做差
    def __eq__(self, other):
        # 当两条访问记录的名字与电话号相等时，判定二者相等。
        if isinstance(other, VisitRecord) and hash(other) == hash(self):
            return True
        return False


# 做差集
def find_potential_customers():
    return set(VisitRecord(**r) for r in users_visited_phuket) - \
        set(VisitRecord(**r) for r in users_visited_nz)

```

- 4 为了方便定义数据类, 在 3.7 版本中，标准库中新增了 `dataclasses` 模块

[dataclasses--python--doc](https://docs.python.org/3/library/dataclasses.html)
    
    
    相比普通class，dataclass通常不包含私有属性，数据可以直接访问
    dataclass的repr方法通常有固定格式，会打印出类型名以及属性名和它的值
    dataclass拥有__eq__和__hash__魔法方法
    dataclass有着模式单一固定的构造方式，或是需要重载运算符，而普通class通常无需这些工作

```python
@dataclass(unsafe_hash=True)
class VisitRecordDC:
    first_name: str
    last_name: str
    phone_number: str
    # 跳过“访问时间”字段，不作为任何对比条件
    date_visited: str = field(hash=False, compare=False)


def find_potential_customers():
    return set(VisitRecordDC(**r) for r in users_visited_phuket) - \
        set(VisitRecordDC(**r) for r in users_visited_nz)

```

<table>
<thead>
<tr class="header">
<th>key</th>
<th>含义</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>init</td>
<td>指定是否自动生成<code>__init__</code>，如果已经有定义同名方法则忽略这个值，也就是指定为True也不会自动生成</td>
</tr>
<tr class="even">
<td>repr</td>
<td>同init，指定是否自动生成<code>__repr__</code>；自动生成的打印格式为<code>class_name(arrt1:value1, attr2:value2, ...)</code></td>
</tr>
<tr class="odd">
<td>eq</td>
<td>同init，指定是否生成<code>__eq__</code>；自动生成的方法将按属性在类内定义时的顺序逐个比较，全部的值相同才会返回True</td>
</tr>
<tr class="even">
<td>order</td>
<td>自动生成<code>__lt__</code>，<code>__le__</code>，<code>__gt__</code>，<code>__ge__</code>，比较方式与eq相同；如果order指定为True而eq指定为False，将引发<code>ValueError</code>；如果已经定义同名函数，将引发<code>TypeError</code></td>
</tr>
<tr class="odd">
<td>unsafehash</td>
<td>如果是False，将根据eq和frozen参数来生成<code>__hash__</code>:<br> 1. eq和frozen都为True，<code>__hash__</code>将会生成<br>2. eq为True而frozen为False，<code>__hash__</code>被设为<code>None</code><br>3. eq为False，frozen为True，<code>__hash__</code>将使用超类（object）的同名属性（通常就是基于对象id的hash）<br>当设置为True时将会根据类属性自动生成<code>__hash__</code>，然而这是不安全的，因为这些属性是默认可变的，这会导致hash的不一致，所以除非能保证对象属性不可随意改变，否则应该谨慎地设置该参数为True</td>
</tr>
<tr class="even">
<td>frozen</td>
<td>设为True时对field赋值将会引发错误，对象将是不可变的，如果已经定义了<code>__setattr__</code>和<code>__delattr__</code>将会引发<code>TypeError</code></td>
</tr>
</tbody>
</table>