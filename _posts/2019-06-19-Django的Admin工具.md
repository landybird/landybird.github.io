---
title: Django的Admin组件 
description: Django的Admin组件 
categories:
- python
tags:
- Django
---

> Django框架的优势之一就是自带的Admin组件



#### Django的Admin不是面向终端用户的 `Not for End Users`


    是提供给 site administrators, 用来增删改查数据， 管理后台任务的



#### 显示管理对象

- 使用 `__str__`

```python
from django.db import models
from django.utils.encoding import python_2_unicode_compatible

@python_2_unicode_compatible # For Python 3.5+ and 2.7
class IceCreamBar(models.Model):
    name = models.CharField(max_length=100)
    shell = models.CharField(max_length=100)
    filling = models.CharField(max_length=100)
    has_stick = models.BooleanField(default=True)
    
    def __str__(self):
        return self.name
```

- 使用 `list_display` 增加字段

```python
from django.contrib import admin
from .models import IceCreamBar

@admin.register(IceCreamBar)
class IceCreamBarModelAdmin(admin.ModelAdmin):
    list_display = ('name', 'shell', 'filling')
```

- 增加`URL`链接

```python

# icecreambars/admin.py
from django.contrib import admin
from django.urls import reverse, NoReverseMatch
from django.utils.html import format_html

from .models import IceCreamBar

@admin.register(IceCreamBar)
class IceCreamBarModelAdmin(admin.ModelAdmin):
    list_display = ('name', 'shell', 'filling')
    readonly_fields = ('show_url',)
    
    def show_url(self, instance):
        url = reverse('ice_cream_bar_detail', kwargs={'pk': instance.pk})
        response = format_html("""<a href="{0}">{0}</a>""", url)
        return response
        
    show_url.short_description = 'Ice Cream Bar URL'
    # Displays HTML tags
    # Never set allow_tags to True against user submitted data!!!
    show_url.allow_tags = True
    
    # When allow_tags is set to True, HTML tags are allowed to be displayed in the admin.


```

#### 注意Django的admin user操作没有锁
    
    多个user同时操作的时候， 可能会覆盖前者的操作
    


#### Django Admin文档生成工具 `django.contrib.admindocs` 
    
    
    1 pip install docutils into your project’s virtualenv.
    
    2 Add django.contrib.admindocs to your INSTALLED_APPS.
    
    3 Add (r'^admin/doc/', include('django.contrib.admindocs.urls')) to your
    root URLConf. Make sure it’s included before the r'^admin/' entry, so that requests to
    /admin/doc/ don’t get handled by the latter entry.
    
    4 Optional: Using the admindocs bookmarklets requires the XViewMiddleware to be installed.



#### 增加安全性 `Secure the Django Admin`


- 修改默认的admin url `yoursite.com/admin/`

- 只允许`HTTPS`请求访问, 增加 `TLS`
    
    
    Without TLS, if you log into your Django admin on an open WiFi network, it’s trivial for someone
    to sniff your admin username/password.

- 限制Admin的 IP， `White list`

1 配置在web server中， 但是有时会缺少 配置文件的权限 (只能在项目中处理)

2 加入判断逻辑到 `middleware`中， 当然会包裹每个views处理

[Tightening Django Admin Logins](https://tech.marksblogg.com/django-admin-logins.html)
