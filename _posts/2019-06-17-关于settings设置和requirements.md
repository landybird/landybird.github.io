---
title: 关于settings设置和requirements 
description: 关于settings设置和requirements 
categories:
- python
tags:
- Two_Scoops_of_Django_Best_Practices_for_Django_2017
---

`settings`文件的注意：
    
    
    ➤ All settings files need to be version-controlled.
    版本控制用在设置文件中
    
    ➤ Don’t Repeat Yourself
    继承一个基础的settings设置
    
    ➤ Keep secret keys safe
    保证密钥的安全性 (设置为环境变量)
  

####  使用多个settings文件 

当然, 每个settings对应的模块要有对应的 `requirements 文件`


    settings/
    ├── __init__.py
    ├── base.py
    ├── local.py
    ├── staging.py
    ├── test.py
    ├── production.py


对应的说明

| Settings file | Purpose | 
| --- | --- | 
| base.py | 基本设置 Settings common to all instances of the project | 
| local.py | 本地开发使用, `DEBUG` 模式 包括一些测试的工具 `django-debug-toolbar` | 
| staging.py | Staging version for running a semi-private version of the site on a production server. This is where managers and clients should be looking before your work is moved to production | 
| test.py | Settings for running tests including test runners, in-memory database definitions, and log settings. | 
| production.py | This is the settings file used by your live production server(s). That is, the server(s) that host the real live website. This file contains production-level settings only. It is sometimes called prod.py. | 

> 指定settings文件

```python
python manage.py shell --settings=twoscoops.settings.local

python manage.py runserver --settings=twoscoops.settings.local
```

当然可以通过当前环境的全局变量 `DJANGO_SETTINGS_MODULE` 和 `PYTHONPATH` 来设置


#### 把 `secret key` 密钥之类的配置， 写在环境变量中


> 把 `secrets` 独立出`settings` 的好处:
    
    可以将所有的代码进行 version control， 包括 settings 配置 (不用担心密钥的泄漏)
    



> Environment Variables Do Not Work With Apache
    
    If your target production environment uses Apache (outside of Elastic Beanstalk), then you
    will discover that setting operating system environment variables as described below doesn’t
    work. 


- 本地设置

linux设置环境变量

在 `.bashrc`, `.bash_profile`, or `.profile`加入
    
    
    ```python
    export SOME_SECRET_KEY=1c3-cr3am-15-yummy
    export AUDREY_FREEZER_KEY=y34h-r1ght-d0nt-t0uch-my-1c3-cr34m
    
    unset SOME_SECRET_KEY
    unset AUDREY_FREEZER_KEY
    ```

当然在特定的虚拟环境中 可以使用虚拟环境的`hook钩子`， `venv/bin/postactivate` or `venv/bin/preactivate`



windows设置环境变量

在命令行中 

    
    ```python
     setx SOME_SECRET_KEY 1c3-cr3am-15-yummy
    ```

但是需要重启命令行


- 生产环境 (PaaS)


    ```python
    eb setenv SOME_SECRET_KEY=1c3-cr3am-15-yummy # Elastic Beanstalk
    heroku config:set SOME_SECRET_KEY=1c3-cr3am-15-yummy # Heroku
    ```


最后的settings文件中

```python
# Top of settings/production.py
import os
SOME_SECRET_KEY = os.environ['SOME_SECRET_KEY']
```


#### 使用多个`requirements`

    
    requirements/
    ├── base.txt
    ├── local.txt
    ├── staging.txt
    ├── production.txt


`requirements/base.txt`
```python
Django==1.11.0
psycopg2==2.6.2
djangorestframework==3.4.0
```

继承`base.txt`
`requirements/local.txt`
```python
-r base.txt # includes the base.txt requirements file
coverage==4.2
django-debug-toolbar==1.5
```


#### 处理Django中的文件目录,使用项目中的 `BASE_DIR` (使用`pathlib`代替`os.path`)



```python

from pathlib import Path
BASE_DIR = Path(__file__).resolve().parent.parent

# resolve()
#  Make the path absolute, resolving all symlinks on the way and also
#  normalizing it (for example turning slashes into backslashes under
#  Windows).

# Path()对象支持 / 语法

MEDIA_ROOT = BASE_DIR / 'media'
STATIC_ROOT = BASE_DIR / 'static_root'
STATICFILES_DIRS = [BASE_DIR / 'static']
TEMPLATES = [
{
'BACKEND': 'django.template.backends.django.DjangoTemplates',
'DIRS': [BASE_DIR / 'templates']
},
]

```

