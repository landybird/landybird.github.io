---
title: openpyxl的write_only模式   
description: openpyxl的write_only模式
categories: 
- python    
tags:
- python   
---

[openpyxl的读写优化](https://www.osgeo.cn/openpyxl/optimized.html)

#### 使用`openpyxl`后端导出数据, 占用内存过高

```python
from django.http import HttpResponse
from openpyxl import Workbook, load_workbook
from openpyxl.writer.excel import save_virtual_workbook

class ExcelTableObj(object):
    def __init__(self, file_name=None, write_only=False):
        if file_name:
            self.file_name = file_name
            self.wb = load_workbook(file_name)
        else:
            self.wb = Workbook(write_only=write_only)
        self.write_only = write_only

    def fix_save_excel(self, header, sheet_name, datas):
        ws = self.wb.create_sheet(title=sheet_name)
        ws.append(header)
        for data in datas:
            ws.append(data)


excel = ExcelTableObj(write_only=True)


try:
    excel.fix_save_excel(title, sheetname, data_lists)
except Exception as e:
    import traceback as t; t.print_exc()
    logger.error(repr(e))

finally:
    response = HttpResponse(save_virtual_workbook(excel.wb),
                            content_type='application/vnd.openxmlformats-officedocument.spreadsheetml.sheet')
    response['Content-Disposition'] = 'attachment; filename={}'.format(filename)

```

> 当您要转储大量数据时，请确保 `lxml` 安装


    与普通工作簿不同，新创建的只写工作簿不包含任何工作表；必须使用 create_sheet() 方法。
    
    在只写工作簿中，只能使用 append() . 不能在任意位置用 cell() 或 iter_rows() .
    
    它能够导出无限量的数据（甚至超过了Excel的实际处理能力），同时将内存使用量保持在10MB以下。
    
    只写工作簿只能保存一次。之后，每次试图将工作簿或append（）保存到现有工作表时，都会引发 openpyxl.utils.exceptions.WorkbookAlreadySaved 例外。
    
    在添加单元格之前，必须创建实际单元格数据之前出现在文件中的所有内容，因为在此之前必须将其写入文件。例如， freeze_panes 应在添加单元格之前设置。
    

但是`lxml`安装之后会影响 `save_virtual_workbook(excel.wb)`生成的excel文件完整性

[lxml + django + uwsgi failed to generate a right format excel file？](https://stackoverflow.com/questions/61935831/lxml-django-uwsgi-failed-to-generate-a-right-format-excel-file)