---
title: django_command与python_fire_command
description: django_command与python_fire_command
categories:
- python
tags:
- django
---

<br>

# django_command与python_fire_command

<br>

### django 自带的 command 命令

```python

    
    # your_app/management/commands/fire_command.py
    
    from datetime import datetime
    from your_app.models import ModelOne
    from django.core.management.base import BaseCommand
    
    class Command(BaseCommand):
        help = 'Change Date'
    
        def add_arguments(self, parser):
            parser.add_argument('change_date', type=str, help='Change Date')
    
        def handle(self, *args, **kwargs):
            change_date = kwargs['change_date']
            model_one_obj = ModelOne.objects.all().first()
            model_one_obj.dt = datetime.strptime(change_date, '%Y%m%d')  # convert it to date
            model_one_obj.save()
    
    # Usage
    python manage.py fire_command 20190101


```


### python-fire 模块


```python

import datetime, os, sys

import fire

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.append(BASE_DIR)
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

from management.models import ModelOne


class ManagementCommand(object):

    def change_data(self, date):

        model_one_obj = ModelOne.objects.all().first()
        model_one_obj.dt = date
        model_one_obj.save()

if __name__ == '__main__':
    fire.Fire()



```
