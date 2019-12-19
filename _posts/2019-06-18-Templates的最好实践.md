---
title: Templates的最好实践 
description: Templates的最好实践 
categories:
- python
tags:
- Two_Scoops_of_Django_Best_Practices_for_Django_2017
---


#### 把模板文件放在 `templates`中

目录结构

```python

# 1 
templates/
├── base.html
├── ... (other sitewide templates in here)
├── freezers/
│   ├── ("freezers" app templates in here)


# 2
freezers/
├── templates/
│   ├── freezers/
│   │   ├── ... ("freezers" app templates in here)
templates/
├── base.html
├── ... (other sitewide templates in here)


```

#### `Templates` 结构设计

`flat is better than nested`

2层的设计

```python
templates/
├── base.html
├── dashboard.html # extends base.html
├── profiles/
│ ├── profile_detail.html # extends base.html
│ ├── profile_form.html # extends base.html
```

3层的设计

```python
templates/
├──base.html
├──dashboard.html # extends base.html
├──profiles/
│ ├──base_profiles.html # extends base.html
│ ├──profile_detail.html # extends base_profiles.html
│ ├──profile_form.html # extends base_profiles.html

```

#### 限制， 减少Templates中的运算


```python
# vouchers/models.py

from django.db import models
from django.urls import reverse

from .managers import VoucherManager

class Voucher(models.Model):
    """Vouchers for free pints of ice cream."""
    name = models.CharField(max_length=100)
    email = models.EmailField()
    address = models.TextField()
    birth_date = models.DateField(blank=True)
    sent = models.DateTimeField(null=True, default=None)
    redeemed = models.DateTimeField(null=True, default=None)
    
    objects = VoucherManager()

# vouchers/managers.py

from django.db import models
from django.utils import timezone
from dateutil.relativedelta import relativedelta

class VoucherManager(models.Manager):
    def age_breakdown(self):
    """Returns a dict of age brackets/counts."""
    age_brackets = []
    now = timezone.now()
    delta = now - relativedelta(years=18)
    count = self.model.objects.filter(birth_date__gt=delta).count()
    age_brackets.append(
        {'title': '0-17', 'count': count}
        )
    count = self.model.objects.filter(birth_date__lte=delta).count()
    age_brackets.append(
        {'title': '18+', 'count': count}
        )
    return age_brackets
```

- `Using Templates to Display Pre-Processed Data` 将运算放在model中

```python
{# templates/vouchers/ages.html #}
{  extends "base.html"  }
{  block content  }

<table>
    <thead>
        <tr>
            <th>Age Bracket</th>
            <th>Number of Vouchers Issued</th>
        </tr>
    </thead>
<tbody>

{  for age_bracket in age_brackets  }
    <tr>
        <td>{{ age_bracket.title }}</td>
        <td>{{ age_bracket.count }}</td>
    </tr>
{  endfor  }
</tbody>
</table>
{  endblock content   }

```

- `Filtering With Conditionals in Templates` 将过滤放在model中

```python

# bad example

<h2>Greenfelds Who Want Ice Cream</h2>
<ul>
{  for voucher in voucher_list  }
    {# Don't do this: conditional filtering in templates #}
    {  if 'greenfeld' in voucher.name.lower  }
        <li>{{ voucher.name }}</li>
    {  endif  }
{  endfor  }
</ul>
<h2>Roys Who Want Ice Cream</h2>
<ul>
{  for voucher in voucher_list  }
    {# Don't do this: conditional filtering in templates #}
    {  if 'roy' in voucher.name.lower  }
        <li>{{ voucher.name }}</li>
    {  endif  }
{  endfor  }
</ul>

```

应该把过滤等条件放在外部

```python

# vouchers/views.py

from django.views.generic import TemplateView

from .models import Voucher

class GreenfeldRoyView(TemplateView):
    template_name = 'vouchers/views_conditional.html'
    
    def get_context_data(self, **kwargs):
        context = super(GreenfeldRoyView, self).get_context_data(**kwargs)
        context['greenfelds'] = \
        Voucher.objects.filter(name__icontains='greenfeld')
        context['roys'] = Voucher.objects.filter(name__icontains='roy')
        return context
        


# template

<h2>Greenfelds Who Want Ice Cream</h2>
<ul>
{  for voucher in greenfelds  }
    <li>{{ voucher.name }}</li>
{  endfor  }
</ul>
<h2>Roys Who Want Ice Cream</h2>
<ul>
{  for voucher in roys  }
    <li>{{ voucher.name }}</li>   
{  endfor  }
</ul>     

```

- 连表的查询用 ORM’s `select_related()方法

```python

# bad    User.objects.all()

{# list generated via User.objects.all() #}
<h1>Ice Cream Fans and their favorite flavors.</h1>
<ul>
{  for user in user_list  }
    <li>
    {{ user.name }}:
    {# DON'T DO THIS: Generated implicit query per user #}
    {{ user.flavor.title }}
    {# DON'T DO THIS: Second implicit query per user!!! #}
    {{ user.flavor.scoops_remaining }}
    </li>
{  endfor  }
</ul>



# good  User.objects.all().select_related('flavors')
{  comment  }
List generated via User.objects.all().select_related('flavors')
{  endcomment  }
<h1>Ice Cream Fans and their favorite flavors.</h1>
<ul>
{   for user in user_list  }
    <li>
    {{ user.name }}:
    {{ user.flavor.title }}
    {{ user.flavor.scoops_remaining }}
    </li>
{  endfor  }
</ul>

```

- 减少模板中的API调用(`来自第三方包的api可能会很耗时`)


把所有`processing task` 耗时的任务， 计算 放在 `views`, `models` 或者异步的消息队列中(`Celery`, `Django Channels`)


#### 保持模板样式

```python

{# Use indentation/comments to ensure code quality #}
{# start of list elements #}
{  if list_type=='unordered'  }
    <ul>
{  else  }
    <ol>
{  endif  }
{  for syrup in syrup_list  }
    <li class="{{ syrup.temperature_type|roomtemp }}">
    <a href="{  url 'syrup_detail' syrup.slug  }">
    {  syrup.title  }
    </a>
</li>
{  endfor  }
{# end of list elements #}
{  if list_type=='unordered'  }
    </ul>
{  else  }
    </ol>
{  endif  }

```


#### 模板继承 `Template Inheritance`


`base.html` 基础模板

```python
{# simple base.html #}
{  load staticfiles  }
<html>
<head>
    <title>
        {  block title  }Two Scoops of Django{  endblock title  }
    </title>
    {  block stylesheets  }
        <link rel="stylesheet" type="text/css"
            href="{  static 'css/project.css'  }">
    {  endblock stylesheets  }
</head>
<body>
    <div class="content">
        {  block content  }
            <h1>Two Scoops</h1>
        {  endblock content  }
    </div>
</body>
</html>

```
    
    
    ➤ A title block containing “Two Scoops of Django”.
       标题
    ➤ A stylesheets block containing a link to a project.css file used across our site.
       静态文件
    ➤ A content block containing “<h1>Two Scoops</h1>”.
       内容
       

模板标签的用法

|标签|目的|
|---|---|
|`{  load  }`|引入静态文件 Loads the staticfiles built-in template tag library|
|`{  block  }`|定义子类可以继承的模板块 Since base.html is a parent template, these define which child blocks can be filled in by child templates. We place links and scripts inside them so we can override if necessary|
|`{  load  }`|解析静态文件 Resolves the named static media argument to the static media server.|


`继承模板`

```python
{  extends "base.html"  }
{  load staticfiles  }
{  block title  }About Audrey and Daniel{  endblock title  }
{  block stylesheets  }
    {{ block.super }}
    <link rel="stylesheet" type="text/css"
        href="{  static 'css/about.css'  }">
    {  endblock stylesheets  }
{  block content  }
    {{ block.super }}
    <h2>About Audrey and Daniel</h2>
    <p>They enjoy eating ice cream</p>
{  endblock content  }


# 得到的渲染结果

<html>
<head>
    <title>
        About Audrey and Daniel
    </title>
    <link rel="stylesheet" type="text/css"
        href="/static/css/project.css">
    <link rel="stylesheet" type="text/css"
        href="/static/css/about.css">
</head>
<body>
    <div class="content">
        <h1>Two Scoops</h1>
        <h2>About Audrey and Daniel</h2>
        <p>They enjoy eating ice cream</p>
    </div>
</body>
</html>


```
|标签|目的|
|---|---|
|`{  extends  } `|继承父类模板 Informs Django that about.html is inheriting or extending from base.html
|`{  block  }`|替换子类的模板代码块|
|`{{ block.super }}`|把对应的block中的父类代码 放进子类 When placed in a child template’s block, it ensures that the parent’s content is also included in the block. In the content block of the about.html template, this will render `<h1>Two Scoops</h1>`|

### 使用模板需要注意的地方

- 使用精确易懂的名称

```python

{# templates/toppings/topping_list.html #}
{# Using implicit names, good for code reuse #}
<ol>
{  for object in object_list  }
    <li>{{ object }} </li>
{  endfor  }
</ol>

{# Using explicit names, good for object specific code #}
<ol>
{  for topping in topping_list  }
    <li>{{ topping }} </li>
{  endfor  }
</ol>


```

- 使用 URL Name(`{  url  }` ) 而不是`hardcode path`

```python
# bad
<a href="/flavors/">

# good
<a href="{  url 'flavors:list'   }">


```

- 通过 settings 设置 `string_if_invalid`， 获取更多的 模板错误信息

```python
# settings/local.py
TEMPLATES = [
    {
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    'APP_DIRS': True,
    'OPTIONS':
        'string_if_invalid': 'INVALID EXPRESSION:  s'
    },
]

```

#### 处理异常的 模板

使用单独的静态文件服务器处理静态文件(`Nginx` + `Apache`), web项目挂掉不会影响, 异常处理

在 `PaaS`平台上， 用户可以自己设置静态文件

Github实例

[github.com/404](https://github.com/404)
[github.com/500](https://github.com/500)


