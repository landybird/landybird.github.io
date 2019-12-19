---
title: Django使用https协议
description: Django使用https协议
categories:
 - python
tags:
- python基础
---


# Django 使用 https协议

<br>

默认runserver启动是使用 http协议, 在做 google第三方应用 授权的时候, callback url必须使用 https协议, 所以需要
加几个插件
```python
pip install django-extensions
pip install django-werkzeug-debugger-runserver
pip install pyOpenSSL
```

然后在 settings中的 INSTALLED_APPS中 添加 应用

```python
‘werkzeug_debugger_runserver’,
‘django_extensions’,
```

启动命令使用：

```python
python manage.py runserver_plus --cert server.crt  xxx-206.xxx-inc.cn:9431
```

