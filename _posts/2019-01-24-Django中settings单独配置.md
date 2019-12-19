---
title: django中settings配置文件设置(脚本使用django中的部分组件)
description: django中settings配置文件设置(脚本使用django中的部分组件)
categories:
 - django
tags:
- django
---

> 如何运行一些`独立`于整个Django项目的`脚本`, `只使用Django的部分组件`


#### 1 Set `DJANGO_SETTINGS_MODULE` before you run， 设置`DJANGO_SETTINGS_MODULE`


> 设置环境变量 (`Unix-based systems` vs `windows`)

The simplest method is to simply assign a value to the DJANGO_SETTINGS_MODULE environment variable before you run your script, 
and that’s not terribly hard to do if you understand a little bit about how `environment variables` work. On most Unix-based systems (including Linux and Mac OS X),
you can typically do this with the export command of the standard shell:


    export DJANGO_SETTINGS_MODULE=yoursite.settings


> 应用在`crontab` 定时任务脚本


    # Cron jobs for foo.com run at 3AM
    
    DJANGO_SETTINGS_MODULE=foo.settings
    
    0 3 * * * python /path/to/maintenance/script.py
    30 3 * * * python /path/to/other/script.py
    
    # Cron jobs for bar.com run at 4AM
    
    DJANGO_SETTINGS_MODULE=bar.settings
    
    0 4 * * * python /path/to/maintenance/script.py
    30 4 * * * python /path/to/other/script.py
    



#### 2 使用 `setup_environ()`


```python

from django.core.management import setup_environ
from mysite import settings

setup_environ(settings)
```



#### 3 Use `settings.configure()`

```python
from django.conf import settings

settings.configure(TEMPLATE_DIRS=('/path/to/template_dir',), DEBUG=False,
                   TEMPLATE_DEBUG=False)

# django.conf.settings is always an instance of LazySettings, which is used to ensure that settings aren’t accessed until they’re actually needed
```



#### 4 命令行指定


```python
import os
from optparse import OptionParser

usage = "usage: %prog -s SETTINGS | --settings=SETTINGS"
parser = OptionParser(usage)
parser.add_option('-s', '--settings', dest='settings', metavar='SETTINGS',
                  help="The Django settings module to use")
(options, args) = parser.parse_args()
if not options.settings:
    parser.error("You must specify a settings module")

os.environ['DJANGO_SETTINGS_MODULE'] = options.settings

# python myscript.py --settings=yoursite.settings


```

[more--standalone script that uses Django ](https://www.b-list.org/weblog/2007/sep/22/standalone-django-scripts/)
