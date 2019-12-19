---
title: Django REST Framework
description: Django REST Framework
categories:
- python
tags:
- Two_Scoops_of_Django_Best_Practices_for_Django_2017
---

使用 `Django REST Framework` 构建 `REST APIs`， 满足 Ajax 和 移动端接口的使用

    Representational State Transfer (REST) Application Programming Interface (API)
    

DRF  `Django REST Framework` 的优点：

    ➤ DRF leans heavily on object-oriented design and is designed to be easily extensible.
      大量使用面向对象编程思想, 容易扩展
      
    ➤ DRF builds directly off of Django CBVs. If you understand CBVs, DRF’s design feels like an
      understandable extension of Django.
       扩展 Django的CBV， 容易在CBV的基础上进行扩展
    
    ➤ It comes with a host of views for API generation, ranging from the
    djano.views.generic.View-like APIView to deep abstractions like generic API
    views and viewsets.
    类似 Django 的 djano.views.generic.View， 有大量类似的基础视图类实现， 方便使用
    
    ➤ The serializer system is extremely powerful, but can be trivially ignored or replaced.
    序列化系统完整， 也可以自己实现
    
    ➤ Authentication and Authorization are covered in a powerful, extendable way.
    认证系统
    
    ➤ If you really want to use FBVs for your API, DRF has you covered there too
    也支持FBV模式
    
    
[Django REST Framework](https://www.django-rest-framework.org/tutorial/quickstart/)


#### 使用`DRF`基本的API 设计 , 一些基本概念


HTTP协议提供不同的请求方法进行不同的动作， `REST APIs` 则依赖于这些请求方法

|Purpose of Request 目的|HTTP method|对应的SQL操作|
|---|---|---|
|Create a new resource 创建资源|`POST`|`INSERT`|
|Read an existing resource 获取资源|`GET`|`SELECT`|
|Update an existing resource 更新已有资源|`PUT`|`UPDATE`|
|Update part of an existing resource 更新已有资源的部分内容|`PATCH`|`UPDATE`|
|Delete an existing resource|`DELETE`|`DELETE`|
|Returns same HTTP headers as GET, but no body content 返回与`GET`相同的响应头 但是没有 `body content`|`HEAD`||
|Return the supported HTTP methods for the given URL 获取服务器对于某些资源的选项、支持情况()|`OPTIONS`||
|Echo back the request 使服务器原样返回任意客户端请求的任何内容|`TRACE`||


常见的HTTP请求状态码

|HTTP Status Code|Success/Failure|Meaning|
|---|---|---|
|200 OK|Success|`GET` - Return resource;  `PUT` - Provide status message or return resource|
|201 Created|Success|POST - Provide status message or return newly created resource|
|204 No Content|Success|`PUT or DELETE - Response to successful update or delete request|
|304 Not Modified|Redirect|ALL - Indicates no changes since the last request. Used for checking Last-Modified and ETag headers to improve performance.|
|400 Bad Request |Failure|ALL - Return error messages, including form validation errors..|
|401 Unauthorized |Failure|ALL - Authentication required but user did not provide credentials or provided invalid ones.|
|403 Forbidden |Failure|ALL - User attempted to access restricted content|
|404 Not Found |Failure|ALL - Resource is not found.|
|405 Method Not Allowed |Failure|ALL - An unallowed HTTP method was attempted..|
|410 Gone |Failure|ALL - A requested resource is no longer available and won’t be available in the future. Used when an API is shut down in favor of a newer version of an API. Mobile applications can test for this condition, and if it occurs, tell the user to upgrade.
|429 Too Many Requests |Failure|ALL - The user has sent too many requests in a given amount of time. Intended for use with rate limiting.|


#### 基本流程， 创建一个简单的API

限制API权限 ` rest_framework.permissions.IsAdminUser`

```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAdminUser',
    ),
}
```

> model

```python
# flavors/models.py

import uuid as uuid_lib

from django.db import models
from django.urls import reverse

class Flavor(models.Model):
    title = models.CharField(max_length=255)
    slug = models.SlugField(unique=True) # Used to find the web URL
    uuid = models.UUIDField( # Used by the API to look up the record
        db_index=True,
        default=uuid_lib.uuid4,
        editable=False)
    scoops_remaining = models.IntegerField(default=0)
    
    def get_absolute_url(self):
        return reverse('flavors:detail', kwargs={'slug': self.slug})

```

注意

尽量避免使用连续的键` Sequential Keys `作为查询条件，会有安全隐患，  上述的例子使用`uuid`

    avoid using sequential numbers for lookups.


> serializer

```python
# flavors/api/serializers.py

from rest_framework import serializers

from ..models import Flavor

class FlavorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Flavor
        fields = ['title', 'slug', 'uuid', 'scoops_remaining']

```

> API view

```python
# flavors/api/views.py

from rest_framework.generics import (
    ListCreateAPIView,
    RetrieveUpdateDestroyAPIView
    )
from rest_framework.permissions import IsAuthenticated

from ..models import Flavor
from .serializers import FlavorSerializer


class FlavorListCreateAPIView(ListCreateAPIView):
    queryset = Flavor.objects.all()
    
    permission_classes = (IsAuthenticated, )
    serializer_class = FlavorSerializer
    lookup_field = 'uuid' # Don't use Flavor.id!
    
    
class FlavorRetrieveUpdateDestroyAPIView(RetrieveUpdateDestroyAPIView):
    queryset = Flavor.objects.all()
    permission_classes = (IsAuthenticated, )
    serializer_class = FlavorSerializer
    lookup_field = 'uuid' # Don't use Flavor.id!

```
> urls

```python
# flavors/urls.py

from django.conf.urls import url

from flavors.api import views

urlpatterns = [
    # /flavors/api/
    url(
        regex=r'^api/$',
        view=views.FlavorListCreateAPIView.as_view(),
        name='flavor_rest_api'
    ),
    # /flavors/api/:slug/
    url(
        regex=r'^api/(?P<uuid>[-\w]+)/$',
        view=views.FlavorRetrieveUpdateDestroyAPIView.as_view(),
        name='flavor_rest_api'
    )
]

```

#### REST API的項目结构

- API 模块的命名与结构

```python
flavors/
├── api/
│   ├── __init__.py
│   ├── authentication.py
│   ├── parsers.py
│   ├── permissions.py
│   ├── renderers.py
│   ├── serializers.py
│   ├── validators.py
│   ├── views.py
│   ├── viewsets.py
```

- API 视图 与 路由

    
    api/flavors/ # GET, POST
    api/flavors/:uuid/ # GET, PUT, DELETE
    api/users/ # GET, POST
    api/users/:uuid/ # GET, PUT, DELETE




```python
# core/api_urls.py

"""Called from the project root's urls.py URLConf thus:
url(r'^api/', include('core.api_urls', namespace='api')),
"""
from django.conf.urls import url

from flavors.api import views as flavor_views
from users.api import views as user_views

urlpatterns = [
    # { url 'api:flavors'  }
    url(
        regex=r'^flavors/$',
        view=flavor_views.FlavorCreateReadView.as_view(),
        name='flavors'
    ),
    # {  url 'api:flavors' flavor.uuid  }
    url(
        regex=r'^flavors/(?P<uuid>[-\w]+)/$',
        view=flavor_views.FlavorReadUpdateDeleteView.as_view(),
        name='flavors'
    ),
    # {  url 'api:users'  }
    url(
        regex=r'^users/$',
        view=user_views.UserCreateReadView.as_view(),
        name='users'
    ),
    # {  url 'api:users' user.uuid  }
    url(
        regex=r'^users/(?P<uuid>[-\w]+)/$',
        view=user_views.UserReadUpdateDeleteView.as_view(),
        name='users'
    ),
]

```
- 控制好api的版本 (保證兼容性)


- 使用自己设计的验证
    
    
    ➤ If we’re creating a new authentication scheme, we keep it simple and well tested.
    简化
    
    ➤ Outside of the code, we document why existing standard authentication schemes are insufficient. See the tipbox below.
    注释清楚使用原因，DRF 自带的验证不能满足条件的原因
    
    ➤ Also outside of the code, we document in depth how our authentication scheme is designed
    to work. See the tipbox below.
    
    ➤ Unless we are writing a non-cookie based scheme, we don’t disable CSRF.
    保留CSRF验证 除非使用 非cookie
    



#### DRF使用的tips

- RPC调用
[RPC](https://en.wikipedia.org/wiki/Remote_procedure_call)

REST API 中 处理其他业务的方法
```python
# sundaes/api/views.py
from django.shortcuts import get_object_or_404

from rest_framework.response import Response
from rest_framework.views import APIView

from ..models import Sundae, Syrup
from .serializers import SundaeSerializer, SyrupSerializer

class PourSyrupOnSundaeView(APIView):
    """View dedicated to adding syrup to sundaes"""
    def post(self, request, *args, **kwargs):
        # Process pouring of syrup here,
        # Limit each type of syrup to just one pour
        # Max pours is 3 per sundae
        sundae = get_object_or_404(Sundae, uuid=request.data['uuid'])
        try:
            sundae.add_syrup(request.data['syrup'])
        except Sundae.TooManySyrups:
            msg = "Sundae already maxed out for syrups"
            return Response({'message': msg}, status_code=400)
        except Syrup.DoesNotExist
            msg = "{} does not exist".format(request.data['syrup'])
            return Response({'message': msg}, status_code=404)
        return Response(SundaeSerializer(sundae).data)
    
    def get(self, request, *args, **kwargs)
        # Get list of syrups already poured onto the sundae
        sundae = get_object_or_404(Sundae, uuid=request.data['uuid'])
        syrups = [SyrupSerializer(x).data for x in sundae.syrup_set.all()]
        return Response(syrups)


/sundae/ # GET, POST
/sundae/:uuid/ # PUT, DELETE
/sundae/:uuid/syrup/ # GET, POST
/syrup/ # GET, POST
/syrup/:uuid/ # PUT, DELETE
```

- 有嵌套关系的API 

        
        /api/cones/ # GET, POST
        /api/cones/:uuid/ # PUT, DELETE
        /api/scoops/ # GET, POST
        /api/scoops/:uuid/ # PUT, DELETE
            
                 ||
                 ||
                \||/       
        
        /api/cones/ # GET, POST
        /api/cones/:uuid/ # PUT, DELETE
        /api/cones/:uuid/scoops/ # GET, POST
        /api/cones/:uuid/scoops/:uuid/ # PUT, DELETE
        /api/scoops/ # GET, POST
        /api/scoops/:uuid/ # PUT, DELETE




#### 关闭不用的API

- 通知用户


- 使用 `410 Error View` 替换之前的 API


```python

# core/apiv1_shutdown.py

from django.http import HttpResponseGone

apiv1_gone_msg = """APIv1 was removed on April 2, 2017. Please switch to APIv2:
                <ul>
                    <li>
                        <a href="https://www.example.com/api/v3/">APIv3 Endpoint</a>
                    </li>
                    <li>
                        <a href="https://example.com/apiv3_docs/">APIv3 Documentation</a>
                    </li>
                    <li>
                        <a href="http://example.com/apiv1_shutdown/">APIv1 shut down notice</a>
                    </li>
                </ul>
                """
                
def apiv1_gone(request):
    return HttpResponseGone(apiv1_gone_msg)

```


#### 给API访问设置`Rate-limiting`


- 无限制的访问是危险的

- 使用DRF自带的Rate limit
 

也可以通过`Nginx`或者`apache`服务器设置
    
    
    It’s possible to use nginx or apache for rate limiting. 
    
        The upside is faster performance.
        优点： 性能更好 
        
        The downside is that it removes this functionality from the Python code
        缺点： 没有python代码逻辑控制
        
        
- rate limit 可以用于业务business 控制

    
    Developer tier is free, but only allows 10 API requests per hour.
    One Scoop is $24/month, allows 25 requests per minute.
    Two Scoops is $79/month, allows 50 requests per minute.
    Corporate is $5000/month, allows for 200 requests per minute.



#### 发布 `REST API`

- 注释， 文档

- 提供 客户端 SDKs

#### `Ajax` 与 `CSRF Token`

使用 Ajax与Django交互的时候， `CSRF` 会阻止 `POST`， `PATCH`， `DELETE` 的请求

[handling CSRF with AJAX](https://docs.djangoproject.com/en/1.11/ref/csrf/#ajax)


> 注意：Don’t Use AJAX as an Excuse to Turn Off CSRF


    Django core developer Aymeric Augustin says, “...CSRF protection is almost always disabled
    because the developers couldn’t quite figure out how to make it work. It’s fine to disable CSRF
    if the API only accepts JWT authentication; it’s wrong it if accepts cookie authentication.”
    Unless you are using django-rest-framework-jwt (django-rest-framework-JWT), don’t
    build a site with disabled CSRF. If you can’t figure out how to make it work, ask for help. No
    one’s going to make fun of someone trying to make their site more secure.


- 适当的设置 `settings.CSRF_COOKIE_HTTPONLY`

Placing a Hidden CSRF Form Element

```python
<html>
    <!-- Placed anywhere in the page, doesn't even need to
    be in a form as the input element is hidden -->
    {  csrf_token  }
</html>
```

Taking the CSRF Token from the DOM
```python
var csrfToken = $('[name=csrfmiddlewaretoken]').val();
var formData = {
    csrfmiddlewaretoken: csrfToken,
    name=name, age=age
};
$.ajax({
    url: '/api/do-something/'',
    data: formData,
    type: 'POST'
})



```
