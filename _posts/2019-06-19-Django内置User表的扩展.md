---
title: Django内置User表的扩展 
description: Django内置User表的扩展 
categories:
- python
tags:
- Two_Scoops_of_Django_Best_Practices_for_Django_2017
---


#### 查找当前的 `User` `Use Django’s Tools for Finding the User Model`

```python
# Stock user model definition
>>> from django.contrib.auth import get_user_model
>>> get_user_model()
<class django.contrib.auth.models.User>

# When the project has a custom user model definition
>>> from django.contrib.auth import get_user_model
>>> get_user_model()
<class profiles.models.UserProfile>


```

- 使用 `settings.AUTH_USER_MODEL` 增加 User外键

```python

from django.conf import settings
from django.db import models

class IceCreamStore(models.Model):
    owner = models.OneToOneField(settings.AUTH_USER_MODEL)
    title = models.CharField(max_length=255)
```


- 不要使用 `get_user_model()`， 而使用 `settings.AUTH_USER_MODEL` 


    会生成循环引用
    it tends to create import loops
    

#### 自定义 User 的字段

- 继承 `AbstractUser`

```python
# profiles/models.py

from django.contrib.auth.models import AbstractUser
from django.db import models

class KarmaUser(AbstractUser):
    karma = models.PositiveIntegerField(verbose_name='karma',
        default=0,
        blank=True)

```

然后在配置中加入

```python
AUTH_USER_MODEL = 'profiles.KarmaUser'
```

- 继承 `AbstractBaseUser`

    
    only 3 fields: 
        password
        last_login
        is_active.


- 连表查询

```python
# profiles/models.py

from django.conf import settings
from django.db import models

from flavors.models import Flavor


class EaterProfile(models.Model):
    # Default user profile
    # If you do this you need to either have a post_save signal or
    # redirect to a profile_edit view on initial login.
    user = models.OneToOneField(settings.AUTH_USER_MODEL)
    favorite_ice_cream = models.ForeignKey(Flavor, null=True, blank=True)
    
    
class ScooperProfile(models.Model):
    user = models.OneToOneField(settings.AUTH_USER_MODEL)
    scoops_scooped = models.IntegerField(default=0)
    
    
class InventorProfile(models.Model):
    user = models.OneToOneField(settings.AUTH_USER_MODEL)
    flavors_invented = models.ManyToManyField(Flavor, null=True, blank=True)


>> user.eaterprofile.favorite_ice_cream
```
