---
title: Django中的Model
description: Django中的Model
categories:
- python
tags:
- Two_Scoops_of_Django_Best_Practices_for_Django_2017
---

model相关的插件

➤ `django-model-utils`  [django-model-utils](https://github.com/jazzband/django-model-utils/stargazers)
 
     handle common patterns like TimeStampedModel.

➤ `django-extensions `[django-extensions](https://github.com/django-extensions/django-extensions/stargazers)

    has a powerful management command called shell_plus which
    autoloads the model classes for all installed apps. The downside of this library is that it
    includes a lot of other functionality which breaks from our preference for small, focused
    apps.
    

#### 基本的概念（model的继承）


- 1 一个app中models的数量要控制(`not over 20`)



- 2 注意Model的继承

    
    abstract base classes 抽象类
    
    multi-table inheritance 多表继承 
    
    proxy models  代理类继承
 
 
 例子 一个创建，更新自动生成的时间model基类：
 
 
 `core/models.py`
 ```python
from django.db import models
class TimeStampedModel(models.Model):
    """
    An abstract base class model that provides selfupdating ``created`` and ``modified`` fields.
    """
    created = models.DateTimeField(auto_now_add=True)
    modified = models.DateTimeField(auto_now=True)
    class Meta:
        abstract = True
 ```
 `flavors/models.py`
 ```python
# flavors/models.py
from django.db import models
from core.models import TimeStampedModel
    class Flavor(TimeStampedModel):
        title = models.CharField(max_length=200)
 ```
 

<table>
  <thead>
    <tr>
      <th>Model Inheritance Style继承类型</th>
      <th>pros 优点</th>
      <th>cons缺点</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1  没有继承 （同一个字段每个表都拥有）</td>
      <td>容易理解 model与数据库的映射关系</td>
      <td>不容易维护</td>
    </tr>
    <tr>
      <td>2  抽象基类继承 （该类的衍生类 才会创建表，本身不会创建）</td>
      <td>写一个系统不会生成表的 TimeStampModel () ；而真正要生成的表用来继承 这个表</td>
      <td>子类不能与父类的字段重复；不能单独使用父类；</td>
    </tr>
    <tr>
      <td>3 多表继承 （父类子类都会产生新表）</td>
      <td>自动创建的OneToOneField 关系；</td>
      <td>每次查询 都会 进行连表， 效率低  (不建议使用)</td>
    </tr>
    <tr>
      <td>4 代理继承 （为原始 model 创建一个代理(proxy）</td>
      <td>可以在代理 model 中改变默认的排序设置和默认的 manager 管理器</td>
      <td>不能改变父类中的字段</td>
    </tr>
  </tbody>
</table>


#### Database 迁移 `django.db.migrations`


注意

> 使用  `python manage.py makemigrations` 为 Model创建 `initial django.db.migrations `

> 在迁移的时候 做好数据备份

可以添加附加的动作`RunPython` or `RunSQL`

[runpython](https://docs.djangoproject.com/en/1.11/ref/migration-operations/#runpython)

[runsql](https://docs.djangoproject.com/en/1.11/ref/migration-operations/#runsql)

Use `RunPython.noop` to Do Nothing

```python
from django.db import migrations, models

def add_cones(apps, schema_editor):
    Scoop = apps.get_model('scoop', 'Scoop')
    Cone = apps.get_model('cone', 'Cone')
    for scoop in Scoop.objects.all():
        Cone.objects.create(
        scoop=scoop,
        style='sugar'
        )
        
        
class Migration(migrations.Migration):
    initial = True
    dependencies = [
    ('scoop', '0051_auto_20670724'),
    ]
    operations = [
        migrations.CreateModel(
            name='Cone',
            fields=[
            ('id', models.AutoField(auto_created=True, primary_key=True,
            serialize=False, verbose_name='ID')),
            ('style', models.CharField(max_length=10),
            choices=[('sugar', 'Sugar'), ('waffle', 'Waffle')]),
            ('scoop', models.OneToOneField(null=True, to='scoop.Scoop'
            on_delete=django.db.models.deletion.SET_NULL, )),
            ],
        ),
        # RunPython.noop does nothing but allows reverse migrations to occur
        migrations.RunPython(add_cones, migrations.RunPython.noop)
]

```

#### Model的设计建议

- 1 使用 `Null` 和 `Blank` 



<table>
  <thead>
    <tr>
      <th>字段类型</th>
      <th>Null=True</th>
      <th>Blank=True</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>CharField, TextField,
          SlugField,
          EmailField,
          CommaSeparatedIntegerField,
          UUIDField</td>
      <td>Okay if you also have set both
          unique=True and blank=True.
          In this situation, null=True is
          required to avoid unique
          constraint violations when saving
          multiple objects with blank
          values.</td>
      <td>Okay if you want the
          corresponding form widget
          to accept empty values. If
          you set this, empty values
          are stored as NULL in the
          database if null=True and
          unique=True are also set.
          Otherwise, they get stored
          as empty strings</td>
    </tr>
    <tr>
      <td>FileField,
          ImageField</td>
      <td>Don’t do this.
          Django stores the path from
          MEDIA_ROOT to the file or to the
          image in a CharField, so the same
          pattern applies to FileFields</td>
      <td>Okay.
          The same pattern for
          CharField applies here</td>
    </tr>
    <tr>
      <td>BooleanField</td>
      <td>Don’t do this. Use
          NullBooleanField instead.</td>
      <td>Don’t do this.</td>
    </tr>
    <tr>
      <td>IntegerField,
          FloatField,
          DecimalField,
          DurationField, etc</td>
      <td>Okay if you want to be able to set
          the value to NULL in the database</td>
      <td>Okay if you want the
          corresponding form widget
          to accept empty values. If
          so, you will also want to set
          null=True.
</td>
    </tr> 
      
  <tr>
      <td>DateTimeField,
          DateField, TimeField,
          etc.</td>
      <td>Okay if you want to be able to set
          the value to NULL in the database</td>
      <td>Okay if you want the
          corresponding form widget
          to accept empty values, or if
          you are using auto_now or
          auto_now_add. If it’s the
          former, you will also want
          to set null=True.
    </td>
    </tr>      
  <tr>
      <td>ForeignKey,
          ManyToManyField,
          OneToOneField</td>
      <td>Okay if you want to be able to set
          the value to NULL in the database.</td>
      <td>Okay if you want the
          corresponding form widget
          (e.g. the select box) to
          accept empty values. If so,
          you will also want to set
          null=True
    </td>
    </tr> 
     
 <tr>
      <td>GenericIPAddressField</td>
      <td>Okay if you want to be able to set
          the value to NULL in the database</td>
      <td>Okay if you want to make
          the corresponding field
          widget accept empty values.
          If so, you will also want to
          set null=True.
    </td>
    </tr>
  </tbody>
</table>


- 使用 `BinaryField`

    
     raw binary data, or bytes
     
不能使用 filter， excludes 或者是其他的SQL动作走

    
    ➤ MessagePack-formatted content.
    ➤ Raw sensor data.
    ➤ Compressed data e.g. the type of data Sentry stores as a BLOB, but is required to base64-
    encode due to legacy issues.
    
注意：

    不要保存文件到 BinaryField
    

- 不要使用 `Generic Relations` 泛型关系(指向某个model)

使用` models.field.GenericForeignKey `， 绑定一个model. 类似于使用没有主键的`NoSQL`
    
    ➤ Reduction in speed of queries due to lack of indexing between models.
    ➤ Danger of data corruption as a table can refer to another against a non-existent record.
    

- 使用 `Choices` 和 `Enum`

```python
# orders/models.py
from django import models
class IceCreamOrder(models.Model):
    FLAVOR_CHOCOLATE = 'ch'
    FLAVOR_VANILLA = 'vn'
    FLAVOR_STRAWBERRY = 'st'
    FLAVOR_CHUNKY_MUNKY = 'cm'
    
    FLAVOR_CHOICES = (
    (FLAVOR_CHOCOLATE, 'Chocolate'),
    (FLAVOR_VANILLA, 'Vanilla'),
    (FLAVOR_STRAWBERRY, 'Strawberry'),
    (FLAVOR_CHUNKY_MUNKY, 'Chunky Munky')
    
    )
    flavor = models.CharField(
        max_length=2,
        choices=FLAVOR_CHOICES
    )

```

更好的方式 `Enum` , `py2.7+` 和 `py3.4+`

```python
from django import models
from enum import Enum

class IceCreamOrder(models.Model):
    class FLAVORS(Enum):
        chocolate = ('ch', 'Chocolate')
        vanilla = ('vn', 'Vanilla')
        strawberry = ('st', 'Strawberry')
        chunky_munky = ('cm', 'Chunky Munky')

    flavor = models.CharField(
        max_length=2,
        choices=[x.value for x in FLAVORS]
        )
```

####  The Model `_meta` API

    
    ➤ Building a Django model introspection tool.
    ➤ Building your own custom specialized Django form library.
    ➤ Creating admin-like tools to edit or interact with Django model data.
    ➤ Writing visualization or analysis libraries, e.g. analyzing info only about fields that start with
    “foo”.
    ➤ Get a list of a model’s fields.
    ➤ Get the class of a particular field for a model (or its inheritance chain or other info derived
    from such).
    ➤ Ensure that how you get this information remains constant across future Django versions
    
[Model _meta docs](https://docs.djangoproject.com/en/1.11/ref/models/meta/)

#### Model Managers 管理器

每次使用 Django的ORM的时候, 都会调用 `model manager` 和数据库交互, Django提供了默认的管理器，可以自定义


```python

from django.db import models
from django.utils import timezone

class PublishedManager(models.Manager):
    use_for_related_fields = True
    def published(self, **kwargs):
        return self.filter(pub_date__lte=timezone.now(), **kwargs)
        
        
class FlavorReview(models.Model):
    review = models.CharField(max_length=255)
    pub_date = models.DateTimeField()
    
    # add our custom model manager
    objects = PublishedManager()



>>> from reviews.models import FlavorReview
>>> FlavorReview.objects.count()
35
>>> FlavorReview.objects.published().count()
31
 

```


#### 



