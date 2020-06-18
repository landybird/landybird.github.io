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
columns =[['米'], ['amy'], ['tom'], ['zara'], ['jay'], ['Lee'], ['Uzi'], ['Zome'], ['Qoe'], ['Aoi'], ['Yeezy'], ['Hazy'], ['Wash'], ['pany'], ['zoey'], ['Moe'], ['total']]
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




# 1 

stream_file = StringIO()
df.reset_index()
df.to_csv(stream_file)
stream_file.seek(0)
response = StreamingHttpResponse(stream_file)
response['Content-Type'] = 'application/octet-stream; charset=utf-8-sig'
response['Content-Disposition'] = 'attachment; filename="{}.csv"'.format(urlquote("数据报表"))

return response

```


```python
# zip 打包
def zip_file_response(response, path, *file_names):
    os.chdir(path)
    zip_file = zipfile.ZipFile(response, 'w')
    for file_name in file_names:
        if file_name:
            zip_file.write(file_name, arcname=file_name)
    for file_name in file_names:
        if file_name and os.path.exists(file_name):
            os.remove(file_name)
    zip_file.close()

response = HttpResponse(content_type='application/zip')
zip_file_response(response, path, pie_file_name, excel_name)
response['Content-Disposition'] = 'attachment; filename="{}.zip"'.format(urlquote("数据报表"))
return response
```