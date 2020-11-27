---
title: `Django_filter`支持表单提交数据以及`django_rest_framework`中request的参数修改    
description: Django_filter支持表单提交数据以及django_rest_framework中request的参数修改  
categories:
- python
tags:
- django   
---
    

#### `django-filter` 支持解析 `GET` form_data

[django-filter 使用`application/x-www-form-urlencoded`](https://github.com/carltongibson/django-filter/issues/454)

使用 [django-filter ](https://github.com/carltongibson/django-filter) 可以对 `GET` 请求的 `query_params` 条件进行过滤
    
    
     FilterSet typically works with GET params, which Django parses as a QueryDict
     
[源码](https://github.com/carltongibson/django-filter/blob/master/django_filters/views.py#L52)中限制 从 `request.GET`中获取

```python

class FilterMixin(metaclass=FilterMixinRenames):
    
     def get_filterset_kwargs(self, filterset_class):
            """
            Returns the keyword arguments for instantiating the filterset.
            """
            kwargs = {
                'data': self.request.GET or None,
                # 这里可以修改 getattr（self.request，self.request.method，None）
                # 以支持其他请求方法 POST 
                'request': self.request,
            }
```
 
 
 ```python
 
 # 基本使用 
class FApplication(FilterSet):

    account_id = Filter(
        field_name='account_id',
        lookup_expr='in',
        widget=widgets.QueryArrayWidget
    )
    ...

    class Meta:
        model = MAccount
        fields = ['account_id']
 
 
 
class VSA(CustomModelViewSet):
    queryset = MAccount.objects.all().filter(active=True)
    filter_class = FApplication
 
 ```
 
支持 GET 发送 `form_data` 并解析, 这时数据在`request.POST`中 
 
```python
# 重写FilterSet
# 这样 可以在 GET 请求的时候， 把过多的过滤条件 放在body 里面 (application/x-www-form-urlencoded) 
from django_filters.filterset import BaseFilterSet, FilterSetMetaclass

class FilterSetOwn(BaseFilterSet, metaclass=FilterSetMetaclass):
    def __init__(self, data=None, queryset=None, *, request=None, prefix=None):
        # 这里的data是  FilterMixin 初始化的  'data': self.request.GET or None,
        # 判断 query_params 是否为 False <QueryDict: {}>
        if bool(data) == False:
            data = request.POST or request.data
            # 没有GET传参 则解析 application/x-www-form-urlencoded
        super(FilterSetOwn, self).__init__(data=data, queryset=queryset, request=request, prefix=prefix)


# parser_classes = [JSONParser, MultiPartParser, FormParser]
# FormParser 用来解析  form data. 'application/x-www-form-urlencoded'

```



#### 修改 django, django_rest_framework 的request下的属性
    
    request.POST
    
    request.query_params
    
    
    setattr(request.query_params, '_mutable', True)

    ...

例如 `rest_framework`分页的时候, 请求的当前页大于最大数, 则重置为最大页



```python

from rest_framework.pagination import PageNumberPagination


class Pagination(PageNumberPagination):
    page_size = 10                       
    page_size_query_param = 'page_size'    
    page_query_param = 'current_page'     
    max_page_size = None                   

    def paginate_queryset(self, queryset, request, view=None):
        if view and hasattr(view, 'page_size') and view.page_size:
            self.page_size = view.page_size
        paginator = self.django_paginator_class(queryset, self.page_size)
        current_page_number = int(request.query_params.get(self.page_query_param, 1))
        max_page = paginator.num_pages
        if current_page_number > max_page:
            setattr(request.query_params, '_mutable', True)
            request.query_params[self.page_query_param] = max_page
            setattr(request.query_params, '_mutable', False)
        return super().paginate_queryset(queryset, request, view)

```

不对这个页码做处理 会抛异常

```python
        try:
            self.page = paginator.page(page_number)
        except InvalidPage as exc:
            msg = self.invalid_page_message.format(
                page_number=page_number, message=str(exc)
            )
            raise NotFound(msg)
```

