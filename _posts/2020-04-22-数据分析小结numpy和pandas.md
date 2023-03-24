---
title: 数据分析小结numpy和pandas                              
description: 数据分析小结numpy和pandas
categories:
- python
tags:
- python   
---


### Pandas
 
`Pandas`是基于 `NumPy `的一个开源 Python 库, 它被广泛用于快速分析数据，以及数据清洗和准备等工作。它的名字来源是由`“ Panel data”`（面板数据，一个计量经济学名词）两个单词拼成的。简单地说，

    可以把 Pandas 看作是 Python 版的 Excel。


pandas库具有两个主要的数据容器：`DataFrame`和`Series`


#### Series  


Series 是一种`一维数组`，和 NumPy 里的数组很相似。

Series 基本上`基于 NumPy 的数组对象`
Series 能为数据自定义标签，也就是`索引（index）`，然后通过索引来访问数组中的数据。

```python
import numpy as np
import pandas as pd


countries_index = [
    "US",
    "CN",
    "NIG",
    "FR",
]

my_data = [
    100, 200, 300 ,400,
]

print(pd.Series(my_data, countries_index)) 

# countries_index -- index
# index 参数是可省略的
# 默认index 是 [0, …, len(data) - 1]

# US     100
# CN     200
# NIG    300
# FR     400
# dtype: int64


my_dict = {
    "a":50,
    "b":50,
    "c":50,
}
print(pd.Series(my_dict)) 
# a    50
# b    50
# c    50

```

> 从 Series 里获取数据

```python
import numpy as np
import pandas as pd

series1 = pd.Series(
    [1, 2, 3, 4],
    ["index1", "index2", "index3", "index4", ]
)

print(series1["index1"])

```

> 对 Series 进行算术运算操作

对 Series 的算术运算都是基于 index 进行的。我们可以用加减乘除（+ - * /）这样的运算符对两个 Series 进行运算，Pandas 将会根据索引 index，对响应的数据进行计算，结果将会以浮点数的形式存储，以避免丢失精度。

    如果 Pandas 在两个 Series 里找不到相同的 index，对应的位置就返回一个空值 NaN。



#### DataFrames



Pandas 的 DataFrame（数据表）是一种` 2 维数据结构`，数据以表格的形式存储，分成若干行和列

DataFrame由三个不同的组件组成：`索引`，`列`和`数据`。

    数据也称为值。
    
```python

import numpy as np
import pandas as pd

df = pd.DataFrame(
    np.random.randn(5, 4), # data
    ['a', 'b', 'c', 'd', 'e'],  # index
    ['w', 'x', 'y', 'z'] # column
)

print(df)


          w         x         y         z
a -0.014248  0.190399 -0.256364  1.292507
b  1.053277 -0.061699  1.141301  0.300222
c -2.036380  0.683192 -0.993787 -1.428764
d  1.528900 -1.651681 -0.436064 -0.900029
e -0.783422 -0.548151 -0.168499  0.248049


```

`每一列基本上就是一个 Series` ，它们都用了同一个 index。

`可以把 DataFrame 理解成一组采用同样索引的 Series 的集合`


```python

import numpy as np
import pandas as pd


series1 = pd.Series(
    [1, 2, 3, 4],
    ["index1", "index2", "index3", "index4" ]
)

series2 = pd.Series(
    [4, 3, 2],
    ["index1", "index2", "index3" ]
)

series3 = pd.Series(
    [1, -1, 3, 4],
    ["index1", "index2", "index3", "index4" ]
)

dt = {
    "a": series1,
    "b": series2,
    "c": series3
}

df = pd.DataFrame(dt)
print(df)

        a    b  c
index1  1  4.0  1
index2  2  3.0 -1
index3  3  2.0  3
index4  4  NaN  4


# 2 

import numpy as np
import pandas as pd

dt = {
    'name' : ["name1", "name2", "name3", "name4", "name5"],
    "age": [40, 24, 31, 21, 23],
    "year": [2012, 2015, 2019, 2019, 2019]
}

df = pd.DataFrame(dt, index=["logs", "dubai", "cd", "sda", "ds23"])
print(df)


        name  age  year
logs   name1   40  2012
dubai  name2   24  2015
cd     name3   31  2019
sda    name4   21  2019
ds23   name5   23  2019


print(df["name"]) # 获取列 (每一列基本上就是一个 Series)
print(type(df["name"])) # <class 'pandas.core.series.Series'>

logs     name1
dubai    name2
cd       name3
sda      name4
ds23     name5


print(df[["name", 'year']])
# 获取多个列，那返回的就是一个 DataFrame 类型
        name  year
logs   name1  2012
dubai  name2  2015
cd     name3  2019
sda    name4  2019
ds23   name5  2019
```

> 增加列
    
    # 增加列
    df["column_add"] = df["age"] + df["year"]
    
> 删除列或者行

    # 删除列和行
    df.drop("column_add", axis=1, inplace=True) # 确定要永久性删除某一行/列，需要加上 inplace=True 参数
    df.drop("ds23", axis=0, inplace=True)

> 获取一行或多行数据

    print(df.loc["column1"])
    print(df.iloc[0 ])
    
    print(df.loc["column1", "name"])
    print(df.iloc[0, 0])
    
    
    print(df.loc[["column1", "column2"], ["name", "age"]])


> 条件筛选
    
    df = df[df["age"]>30]["year"]  # 查看 age > 30的year 
    print(df)


> 重置索引

`.reset_index()` 

    不会永久改变你表格的索引，除非你调用的时候明确传入了 inplace 参数

`.set_index()`
    
    将会完全覆盖原来的索引值


> 多级索引

```python
import numpy as np
import pandas as pd

out_side = [
    '0 level',
    '0 level',
    '0 level',
    '1 level',
    '1 level',
    '2 level',
]

inside = [21, 22, 23, 24, 25, 26]

my_index = list(zip(out_side, inside))

my_index = pd.MultiIndex.from_tuples(my_index, names=["levels", "nums"])

df = pd.DataFrame(np.random.randn(6, 2), index=my_index, columns=["A", "B"])
print(df)


                     A         B
levels  nums                    
0 level 21   -0.160762 -1.504058
        22   -0.102505  0.819351
        23    0.763584 -0.660647
1 level 24   -1.438083 -0.357881
        25   -1.591041 -0.212282
2 level 26    0.684951 -1.617700




# 获取多索引的值

r = df.loc["0 level"].loc[21, "A"]
print(r)

-0.160762 


# 交叉选择行和列中的数据

r = df.xs("0 level", level="levels")
print(r)
```



> 分组统计

```python
import numpy as np
import pandas as pd

d = {
    "company": [
        "google", "google", "FB", "oracle", "twitter","twitter",
    ],
    "person": [
        "amy", "backe", "candy", "tony","antony", "yamy"
    ],
    "sales": [
        300, 500, 2000, 100, 100,4000
    ],
}

df = pd.DataFrame(d)

   company  person  sales
0   google     amy    300
1   google   backe    500
2       FB   candy   2000
3   oracle    tony    100
4  twitter  antony    100
5  twitter    yamy   4000



# 1 求平均值 mean

# 调用 .groupby() 方法，并继续用 .mean()
print(df.groupby("company").mean())
         sales
company       
FB        2000
google     400
oracle     100
twitter   2050



# 2 计数 count
print(df.groupby("company").count()["person"])

company
FB         1
google     2
oracle     1
twitter    2


# 3 数据描述 describe

print(df.groupby("company").describe())


        sales                                                             
        count    mean          std     min     25%     50%     75%     max
company                                                                   
FB        1.0  2000.0          NaN  2000.0  2000.0  2000.0  2000.0  2000.0
google    2.0   400.0   141.421356   300.0   350.0   400.0   450.0   500.0
oracle    1.0   100.0          NaN   100.0   100.0   100.0   100.0   100.0
twitter   2.0  2050.0  2757.716447   100.0  1075.0  2050.0  3025.0  4000.0


# 可以用 .transpose() 方法获得一个   竖排的格式
print(df.groupby("company").describe().transpose())

company          FB      google  oracle      twitter
sales count     1.0    2.000000     1.0     2.000000
      mean   2000.0  400.000000   100.0  2050.000000
      std       NaN  141.421356     NaN  2757.716447
      min    2000.0  300.000000   100.0   100.000000
      25%    2000.0  350.000000   100.0  1075.000000
      50%    2000.0  400.000000   100.0  2050.000000
      75%    2000.0  450.000000   100.0  3025.000000
      max    2000.0  500.000000   100.0  4000.000000


print(df.groupby("company").describe().transpose()["google"])
# print(df.groupby("company").describe().loc['google'])

sales  count      2.000000
       mean     400.000000
       std      141.421356
       min      300.000000
       25%      350.000000
       50%      400.000000
       75%      450.000000
       max      500.000000
Name: google, dtype: float64

```


> 合并 `DataFrame`


- 堆叠（Concat）

```python


import numpy as np
import pandas as pd

df1 = pd.DataFrame({'A': ['A0', 'A1', 'A2', 'A3'],
                    'B': ['B0', 'B1', 'B2', 'B3'],
                    'C': ['C0', 'C1', 'C2', 'C3'],
                    'D': ['D0', 'D1', 'D2', 'D3']},
                    index=[0, 1, 2, 3])
df2 = pd.DataFrame({'A': ['A4', 'A5', 'A6', 'A7'],
                    'B': ['B4', 'B5', 'B6', 'B7'],
                    'C': ['C4', 'C5', 'C6', 'C7'],
                    'D': ['D4', 'D5', 'D6', 'D7']},
                    index=[4, 5, 6, 7])
df3 = pd.DataFrame({'A': ['A8', 'A9', 'A10', 'A11'],
                    'B': ['B8', 'B9', 'B10', 'B11'],
                    'C': ['C8', 'C9', 'C10', 'C11'],
                    'D': ['D8', 'D9', 'D10', 'D11']},
                    index=[8, 9, 10, 11])

# 默认堆叠方向   按行的方向堆叠
df = pd.concat([df1, df2, df3])
print(pd.DataFrame(df))

      A    B    C    D
0    A0   B0   C0   D0
1    A1   B1   C1   D1
2    A2   B2   C2   D2
...
9    A9   B9   C9   D9
10  A10  B10  C10  D10
11  A11  B11  C11  D11



# 按列的方向堆叠，需要传入 axis=1 参数

df = pd.concat([df1, df2, df3], axis=1)
print(pd.DataFrame(df))

      A    B    C    D    A    B    C    D    A    B    C    D
0    A0   B0   C0   D0  NaN  NaN  NaN  NaN  NaN  NaN  NaN  NaN
1    A1   B1   C1   D1  NaN  NaN  NaN  NaN  NaN  NaN  NaN  NaN
2    A2   B2   C2   D2  NaN  NaN  NaN  NaN  NaN  NaN  NaN  NaN
3    A3   B3   C3   D3  NaN  NaN  NaN  NaN  NaN  NaN  NaN  NaN
4   NaN  NaN  NaN  NaN   A4   B4   C4   D4  NaN  NaN  NaN  NaN
5   NaN  NaN  NaN  NaN   A5   B5   C5   D5  NaN  NaN  NaN  NaN
6   NaN  NaN  NaN  NaN   A6   B6   C6   D6  NaN  NaN  NaN  NaN
7   NaN  NaN  NaN  NaN   A7   B7   C7   D7  NaN  NaN  NaN  NaN
8   NaN  NaN  NaN  NaN  NaN  NaN  NaN  NaN   A8   B8   C8   D8
9   NaN  NaN  NaN  NaN  NaN  NaN  NaN  NaN   A9   B9   C9   D9
10  NaN  NaN  NaN  NaN  NaN  NaN  NaN  NaN  A10  B10  C10  D10
11  NaN  NaN  NaN  NaN  NaN  NaN  NaN  NaN  A11  B11  C11  D11

```





   
    
- 归并（Merge）
  

```python
import numpy as np
import pandas as pd

left = pd.DataFrame(
    {
        "key1": ["k0", "k0", "k2", "k3"],
        "key2": ["k0", "k1", "k1", "k0"],
        "A": ["a0", "a1", "a2", "a3"],
        "B": ["b0", "b1", "b2", "b3"],
    }
)


right = pd.DataFrame(
    {
        "key1": ["k0", "k0", "k3", "k3"],
        "key2": ["k0", "k1", "k1", "k3"],
        "C": ["c0", "c1", "c2", "c3"],
        "D": ["d0", "d1", "d2", "d3"],
    }
)


print(pd.merge(left, right, how='inner', on=["key1", "key2"]))

# key1 = key1 and key2 = key2

  key1 key2   A   B   C   D
0   k0   k0  a0  b0  c0  d0
1   k0   k1  a1  b1  c1  d1

```

- 连接（Join）


连接采用`索引作为公共的键`，而不是某一列


```python

import numpy as np
import pandas as pd

left = pd.DataFrame(
    {
        "A": ["a0", "a1", "a2", "a3"],
        "B": ["b0", "b1", "b2", "b3"],
    },
    index=["i1", "i2", "i3", "i4"]
)




right = pd.DataFrame(
    {
        "C": ["c0", "c1", "c2", "c3"],
        "D": ["d0", "d1", "d2", "d3"],
    }, index=["i1", "i2", "i3", "i0"]
)


# inner 代表交集，Outer 代表并集

print(left.join(right, how='outer'))
      A    B    C    D
i0  NaN  NaN   c3   d3
i1   a0   b0   c0   d0
i2   a1   b1   c1   d1
i3   a2   b2   c2   d2
i4   a3   b3  NaN  NaN


print(left.join(right, how='inner'))
     A   B   C   D
i1  a0  b0  c0  d0
i2  a1  b1  c1  d1
i3  a2  b2  c2  d2



print(left.join(right))
     A   B    C    D
i1  a0  b0   c0   d0
i2  a1  b1   c1   d1
i3  a2  b2   c2   d2
i4  a3  b3  NaN  NaN
```


> 数值处理


- 去重 unique

```python
import numpy as np
import pandas as pd

df = pd.DataFrame(
    {
        "col1": [1, 2, 3, 4],
        "col2": [100, 200, 123, 100],
        "col3": [1, 2, 3, 4],
    }
)

   col1  col2  col3
0     1   100     1
1     2   200     2
2     3   123     3
3     4   100     4


print(df["col2"].unique()) # [100 200 123]

print(df["col2"].nunique()) # 3  不重复值的个数

print(df["col2"].value_counts())
# 获得所有值和对应值的计数
# 100    2
# 123    1
# 200    1
```

- `apply` 方法


对 DataFrame 中的数据`应用自定义函数`，进行数据处理



```python
import numpy as np
import pandas as pd

def square(x):
    return x**2

df = pd.DataFrame(
    {
        "col1": [1, 2, 3, 4],
        "col2": [100, 200, 123, 100],
        "col3": ["aasd", "sdas", "sadas", "dasdsa"],
    }
)


print(df["col1"].apply(square))

# lambda 匿名函数
print(df["col1"].apply(lambda x: x**2))

# 内置函数
print(df["col3"].apply(len))


```

- 排序 `sort_values(axis=0,ascending=True)`



#### 数据透视表 `pivot_table`


Pandas 的数据透视表能自动帮你对数据进行分组、切片、筛选、排序、计数、求和或取平均值，并将结果直观地显示出来

```python

import numpy as np
import pandas as pd

data = {
    "A" : ["DOG", "CAT", "FOX", "GOAT", "CAT", "DOG"],
    "B" : ["BLACK", "RED", "BLACK", "BROWN", "RED", "BROWN"],
    "C" : ["y", "n", "n", "y", "y", "n"],
    "D" : [12, 23, 123, 12, 123, 12],
}

df = pd.DataFrame(data)
print(df)

      A      B  C    D
0   DOG  BLACK  y   12
1   CAT    RED  n   23
2   FOX  BLACK  n  123
3  GOAT  BROWN  y   12
4   CAT    RED  y  123
5   DOG  BROWN  n   12



print(pd.pivot_table(df, values="D", index=["A", "B"], columns=["C"]))

# values 代表我们需要汇总统计的数据点所在的列，
# index 表示按该列进行分组索引
# columns 则表示最后结果将按该列的数据进行分列
C               n      y
A    B                  
CAT  RED     23.0  123.0
DOG  BLACK    NaN   12.0
     BROWN   12.0    NaN
FOX  BLACK  123.0    NaN
GOAT BROWN    NaN   12.0

```



#### pandas 实例


```
pivot_date = "2023-03-19"
base_column_fields = [
    "entity_name",
    "settlement_name",
    "account_id",
    "account_status",
    "sale_manage_id",
    "spend",
    "spend_last_7d",
    "spend_last_90d",
    "spend_date",
    "account_id",
    "company_id",
    "account_type",
    "_medium_type",
    "_first_time",
]
unique_fields = [
    "entity_name",
]

drop_fields = [
    'sale_manage_id',
    'spend',
    'account_type',
    '_medium_type',
    '_first_time',
]

basic_queryset = KpiAccountDailyStatistic.objects. \
    filter(spend_date=pivot_date). \
    values(*base_column_fields)

df = pd.DataFrame.from_records(basic_queryset)

# todo 只取 entity 不为空
df = df[df["entity_name"].apply(lambda x: x is not None)]

tmp_df = df.copy(deep=True)

# 只取 active的 indirect 账户 (第一次花费时间 > 90)
tmp_df = tmp_df[
    (tmp_df.account_status == AccountStatusEnum.ACTIVE)
    &
    (tmp_df._medium_type == MediumTypeEnum.InDirect)
    ]
tmp_df["_first_time"] = tmp_df["_first_time"].astype('str')
tmp_df = tmp_df[tmp_df["_first_time"].apply(lambda x: self._over_90days(pivot_date, x))]

# 合并主体 QTD > 25 K
tmp_df = tmp_df.groupby(list(unique_fields)).spend_last_90d.sum().reset_index()
tmp_df = tmp_df[tmp_df["spend_last_90d"] > 25000]
# tmp_df = tmp_df.to_dict("records")

# 获取 bc entity 已达标
_bc_entity_DICT = dict(tmp_df.values.tolist())
del tmp_df

tmp_df = df.copy(deep=True)
tmp_df["_first_time"] = tmp_df["_first_time"].astype('str')
tmp_df = tmp_df[tmp_df["_first_time"].apply(lambda x: self._over_90days(pivot_date, x, season_end=True))]
tmp_df = tmp_df["entity_name"]
# tmp_df = tmp_df.replace(to_replace="None", value=np.nan).dropna()

# 可达标 Merge Entity
_bc_entity_REACHABLE = tmp_df.values
_bc_entity_REACHABLE = list(set(tmp_df.values) - set(_bc_entity_DICT.keys()))
_bc_entity_REACHABLE_DICT = dict.fromkeys(_bc_entity_REACHABLE, 1)

# 按照 kpi_pivot_active_indirect_bonus_base 重新组织数据

# 全量 sale 关系
_SALE_RELATION_DICT = {}
for sale_obj in Sale.objects.all():
    _SALE_RELATION_DICT[sale_obj.id] = sale_obj.sale_id, sale_obj.default_leader_id

df[["sale_id", "sale_leader_id"]] = df["sale_manage_id"]. \
    apply(lambda x: self._get_sale_leader_info(x, _SALE_RELATION_DICT)).apply(pd.Series)
df.loc[:, "is_bc"] = df["entity_name"]. \
    apply(lambda x: 1 if _bc_entity_DICT.get(x) else 0)
df.loc[:, "bc_cost"] = df.apply(lambda x: x["spend_last_90d"] if x["is_bc"] else 0, axis=1)

df = df.drop(columns=drop_fields)

df = df.rename(
    columns={
        "spend_last_90d": "spend_quarter",
        "spend_date": "pivot_date",
    }
)

# 1 kpi_pivot_active_indirect_bonus_base
bulk_list_bc_base = []
trans_dic_list = df.to_dict("records")
for trans_dic in trans_dic_list:
    bulk_list_bc_base.append(
        KpiPivotActiveIndirectBonusBase(
            **trans_dic
        )
    )

#  kpi_pivot_bc_total Dataframe
tmp_df = df.copy(deep=True)
base_column_fields = [
    "pivot_date",
    "bc_cost",
    "company_id",
]
tmp_df = tmp_df[base_column_fields]

unique_fields = [
    'company_id',
    'pivot_date'
]
tmp_df = tmp_df[tmp_df["bc_cost"] > 0]
tmp_df = tmp_df.groupby(list(unique_fields)).sum().reset_index()

# 2  kpi_pivot_bc_total
trans_dic_list = tmp_df.to_dict("records")
bulk_list_bc_total = []
for trans_dic in trans_dic_list:
    bulk_list_bc_total.append(
        KpiPivotbcTotal(
            **trans_dic
        )
    )

#  kpi_pivot_active_indirect_bonus_trend Dataframe
tmp_df = df.copy(deep=True)
base_column_fields = [
    "pivot_date",
    "bc_cost",
    "spend_quarter",
    "spend_last_7d",
]
tmp_df = tmp_df[base_column_fields]

unique_fields = [
    'pivot_date'
]
tmp_df = tmp_df.groupby(list(unique_fields)).sum().reset_index()

# 3  kpi_pivot_active_indirect_bonus_trend
trans_dic_list = tmp_df.to_dict("records")
bulk_list_bc_trend = []
for trans_dic in trans_dic_list:
    bulk_list_bc_trend.append(
        KpiPivotActiveIndirectBonusTrend(
            **trans_dic
        )
    )

#  kpi_pivot_active_indirect_bonus_merge_entity Dataframe
tmp_df = df.copy(deep=True)
base_column_fields = [
    "entity_name",
    "sale_id",
    "sale_leader_id",
    "pivot_date",
    "bc_cost",
    "spend_quarter",
    "spend_last_7d",
]
tmp_df = tmp_df[base_column_fields]
unique_fields = [
    'pivot_date',
    'entity_name',
    'sale_id',
    'sale_leader_id',
]
tmp_df = tmp_df.groupby(list(unique_fields)).sum().reset_index()

# 4  kpi_pivot_active_indirect_bonus_merge_entity
trans_dic_list = tmp_df.to_dict("records")
bulk_list_bc_merge_entity = []
for trans_dic in trans_dic_list:
    bulk_list_bc_merge_entity.append(
        KpiPivotActiveIndirectBonusMergeEntity(
            **trans_dic
        )
    )

#  kpi_pivot_active_indirect_bonus_sale Dataframe
tmp_df = df.copy(deep=True)
base_column_fields = [
    "sale_id",
    "sale_leader_id",
    "pivot_date",
    "bc_cost",
    "spend_quarter",
    "spend_last_7d",
]

entity_group_df = tmp_df[[
    "sale_id",
    "entity_name",
]].groupby(["sale_id"])

# 按销售分组的 merged entity
_SALE_entity_DICT = {}
for entity_group in entity_group_df:
    sale_id, tmp_df_ = entity_group
    _SALE_entity_DICT[sale_id] = set(tmp_df_["entity_name"].values)

tmp_df = tmp_df[base_column_fields]
unique_fields = [
    'pivot_date',
    'sale_id',
    'sale_leader_id',
]
tmp_df = tmp_df.groupby(list(unique_fields)).sum().reset_index()

tmp_df[["reached", "reachable", "un_reachable"]] = tmp_df["sale_id"]. \
    apply(lambda x: self._get_sale_entity_info(x,
                                                      _bc_entity_DICT,
                                                      _bc_entity_REACHABLE_DICT,
                                                      _SALE_entity_DICT
                                                      )).apply(pd.Series)
# tmp_df = tmp_df[tmp_df["bc_cost"] > 0]

# 5  kpi_pivot_active_indirect_bonus_sale
trans_dic_list = tmp_df.to_dict("records")
bulk_list_bc_sale = []
for trans_dic in trans_dic_list:
    bulk_list_bc_sale.append(
        KpiPivotActiveIndirectBonusSale(
            **trans_dic
        )
    )
```
