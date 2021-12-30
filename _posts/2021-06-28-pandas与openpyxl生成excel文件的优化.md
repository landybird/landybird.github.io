---
title: pandas与openpyxl生成excel文件的优化
description: pandas与openpyxl生成excel文件的优化
categories: 
- python    
tags:
- python   
---


### 优化 `pandas dataframe` 数据 (`float`, `int`, `object`)

[>> Tutorial: Using Pandas with Large Data Sets in Python](https://www.dataquest.io/blog/pandas-big-data/)

```python 
pivot_df = ... # pandas dataframe

pivot_df_int = pivot_df.select_dtypes(include=['int'])
converted_pivot_df_int = pivot_df_int.apply(pd.to_numeric, downcast='unsigned')
pivot_df_float = pivot_df.select_dtypes(include=['float'])
converted_pivot_df_float = pivot_df_float.apply(pd.to_numeric, downcast='float')

optimized_pivot_df = pivot_df.copy()
optimized_pivot_df[converted_pivot_df_int.columns] = converted_pivot_df_int
optimized_pivot_df[converted_pivot_df_float.columns] = converted_pivot_df_float

optimized_pivot_df_obj = optimized_pivot_df.select_dtypes(include=['object']).copy()
converted_obj = pd.DataFrame()
for col in optimized_pivot_df_obj.columns:
    num_unique_values = len(optimized_pivot_df_obj[col].unique())
    num_total_values = len(optimized_pivot_df_obj[col])
    if num_unique_values / num_total_values < 0.5:
        converted_obj.loc[:, col] = optimized_pivot_df_obj[col].astype('category')
    else:
        converted_obj.loc[:, col] = optimized_pivot_df_obj[col]
optimized_pivot_df[converted_obj.columns] = converted_obj
dtypes = optimized_pivot_df.dtypes
dtypes_type = [i.name for i in dtypes.values]
print(optimized_pivot_df.info(memory_usage='deep'))
for dtype in dtypes_type:
    selected_dtype = optimized_pivot_df.select_dtypes(include=[dtype])
    mean_usage_b = selected_dtype.memory_usage(deep=True).mean()
    mean_usage_mb = mean_usage_b / 1024 ** 2
    print("Average memory usage for {} columns: {:03.2f} MB".format(dtype, mean_usage_mb))

```


### 优化 `pandas to_excel`


#### `engine="xlsxwriter"` 效率更高

```python 
# 1 xlsxwriter
q = QiNiu()
stream_file = BytesIO()
# writer = pd.ExcelWriter(tmp_file, engine="openpyxl")    # 1.6g Memory
# openpyxl 占用内存过大
# writer = pd.ExcelWriter(tmp_file, engine="xlsxwriter")  # 830M Memory
writer = pd.ExcelWriter(stream_file, engine="xlsxwriter") # 819m
pivot_df.to_excel(writer)
writer.save()
stream_file.seek(0)
msg = q.upload_data(f"{title}-{hash_str}.xlsx", stream_file)

```

#### 使用 `openpyxl` `write_only`模式读取 `dataframe`

> 注意： `write_only` 需要 `lxml` 安装

    lxml 可以将 XML 流式传输到缓冲区
    
    
```python 
# 2 openpyxl write only 读取 dataframe  500 M
q = QiNiu()
output = BytesIO()
# output = tmp_file
wb = Workbook(write_only=True)
ws = wb.create_sheet()
# write_only 只写工作簿不包含任何工作表；
# 必须使用 create_sheet() 方法专门创建工作表

for r in dataframe_to_rows(pivot_df, index=True, header=True):
    ws.append(r)

wb.save(output)

output.seek(0)
msg = q.upload_data(f"{title}-{hash_str}.xlsx", output)
print(msg)
```


#### 使用 `pyexcelerate` 模式读取`dataframe`

```python 

# 3 pyexcelerate
# from pyexcelerate import Workbook as Workbook_
# pyexcelerate    700 M 左右， 内存消耗更多
q = QiNiu()
output = BytesIO()
# output = tmp_file
wb = Workbook_()
values = [pivot_df.columns] + list(pivot_df.values)
wb.new_sheet('sheet', data=values)
wb.save(output)

# data = {"key": hash_str, "path": tmp_file}
# msg = q.upload(data)
output.seek(0)
msg = q.upload_data(f"{title}-{hash_str}.xlsx", output)
print(msg)
exit()

```


#### 数据源使用 `生成器`