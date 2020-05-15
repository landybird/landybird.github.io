---
title: 使用pandas的dataframe直接转excel或者csv                        
description: django不保存文件直接返回excel
categories:
- python
tags:
- python   
---


```python

import pandas as pd

indexs = {0: ['cost', '2020-01', 'testing'],
          1: ['cost', '2020-01', '360 Limited'],
          2: ['cost', '2020-02', 'Korea Co.,LTD'],
          3: ['cost', '2020-02', 'ADS4EACH HK TECH LIMITED']}
columns =[['贾米'], ['amy'], ['tom'], ['zara'], ['jay'], ['Lee'], ['Uzi'], ['Zome'], ['Qoe'], ['Aoi'], ['Yeezy'], ['Hazy'], ['Wash'], ['pany'], ['zoey'], ['Moe'], ['total']]
data = {
    0: [0.0, 0.0, 0.0, 0.0, 0.0, 7.85, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 7.85],
    1: [0.0, 0.0, 0.0, 0.0, 0.0, 7.85, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 7.85],
    2: [0.0, 0.0, 0.0, 0.0, 0.0, 7.85, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 7.85],
    3: [0.0, 0.0, 0.0, 0.0, 0.0, 7.85, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 7.85]
}


index = pd.MultiIndex.from_tuples(indices.values())
column = pd.MultiIndex.from_tuples(columns)
data = data.values()

df = pd.DataFrame(data, index=index, columns=column)

# 1 saved as excel

excel_writer = pd.ExcelWriter(path='download.xlsx', engine='openpyxl')
df.to_excel(excel_writer)
wb = excel_writer.book

response = HttpResponse(save_virtual_workbook(wb))
response["Content-Type"] = 'application/vnd.ms-excel'
response['Content-Disposition'] = 'attachment; filename={}.xlsx'.format("~".join(time_range))
return response


# 2 saved as csv

response = HttpResponse(content_type='text/csv')
response['Content-Disposition'] = 'attachment; filename="{}.csv"'.format("~".join(time_range))
response.write(codecs.BOM_UTF8)
# 防止中文乱码

df.to_csv(response, encoding="utf-8-sig")
return response
```
