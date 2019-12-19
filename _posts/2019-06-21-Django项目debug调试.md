---
title: Django项目的调试
description: Django项目的调试
categories:
- python
tags:
- Two_Scoops_of_Django_Best_Practices_for_Django_2017
---


<br>

### 开发环境下的调试 `Debugging in Development`

本地调试有很多工具可以使用

#### 1 `django-debug-toolbar`

它能显示有关当前`请求/应答周期中`的各种调试信息，如模板呈现时间、查询操作、哪些变量等


[django-debug-toolbar](https://pypi.org/project/django-debug-toolbar/)


#### 2 掌握 Python 调试器 `PDB` Master the `Python Debugger`

[pdb — The Python Debugger](https://docs.python.org/3/library/pdb.html)

PDB 用于
    
    测试中
    调试管理命令中
    
在生产环境中不要有 PDB断点，它会中止代码执行。

ipdb 扩展后，功能会更加强大
[ipdb](https://pypi.org/project/ipdb/)


#### 3 文件上传时容易出现的问题

- 确保  `<form> tag` 包含 `encoding type`

```html
<form action="{ url 'stores:file_upload' store.pk }"
method="post"
enctype="multipart/form-data">
```

- 确保 视图函数处理 `request.FILES`

```python


# FBV

# stores/views.py

from django.shortcuts import render, redirect, get_object_or_404
from django.views.generic import View

from stores.forms import UploadFileForm
from stores.models import Store

def upload_file(request, pk):
    """Simple FBV example"""
    store = get_object_or_404(Store, pk=pk)
    if request.method == 'POST':
        # Don't forget to add request.FILES!
        form = UploadFileForm(request.POST, request.FILES)
        if form.is_valid():
            store.handle_uploaded_file(request.FILES['file'])
            return redirect(store)
    else:
        form = UploadFileForm()
    return render(request, 'upload.html', {'form': form, 'store': store})



# CBV
# stores/views.py

from django.shortcuts import render, redirect, get_object_or_404
from django.views.generic import View

from stores.forms import UploadFileForm
from stores.models import Store

class UploadFile(View):
    """Simple CBV example"""
    def get_object(self):
        return get_object_or_404(Store, pk=self.kwargs['pk'])

    def post(self, request, *args, **kwargs):
        store = self.get_object()
        form = UploadFileForm(request.POST, request.FILES)
        if form.is_valid():
            store.handle_uploaded_file(request.FILES['file'])
            return redirect(store)
        return redirect('stores:file_upload', pk=pk)

    def get(self, request, *args, **kwargs):
        store = self.get_object()
        form = UploadFileForm()
        return render(request, 'upload.html', {'form': form, 'store': store})

```


### 生产环境中的调试

某些问题只会在生成环境中出现， 如`负载情况`、`第三方 API` 及`数据量`等。


- 用更便捷的方式查看日志


    日志量可能很多，因此最好通过类似 Sentry 等的日志聚集工具进行查看。


- 制作生产环境的镜像

步骤：
    
    在防火墙或其它保护措施下，配置一个与生产环境一样的远程主机
    复制数据，注意要去除与个人相关的数据。

设置好后，尝试重现 BUG，因为该服务器在防火墙后，故可以将 settings.DEBUG 设置为 True



- 使用基于用户的异常中间件 `UserBasedExceptionMiddleware`

在生产环境中将 settings.DEBUG 设置为 True 会有安全问题。但是基于特定用户来显示调试信息将不会有安全问题：

    provide superusers with access to the settings.DEBUG=True 500 error page in production

```python
# core/middleware.py
import sys

from django.views.debug import technical_500_response

class UserBasedExceptionMiddleware(object):
    def process_exception(self, request, exception):
        if request.user.is_superuser:
            return technical_500_response(request, *sys.exc_info())

```

- `settings.ALLOWED_HOSTS` 在本地与生产环境的设置

`settings.ALLOWED_HOSTS` 列出了 `Django 站点`能提供`服务的主机/域名列表`。

当 settings.DEBUG 设置为 False 时，必须将该值设置起来

没有设置好该配置项的网站会一直产生 500 错误，而且日志中会有 SuspiciousOperation 错误。

```python
# settings.py
ALLOWED_HOSTS = [
    '.djangopackages.com',
    'localhost', # Ensures we can run DEBUG = False locally
    '127.0.0.1' # Ensures we can run DEBUG = False locally
]


```








