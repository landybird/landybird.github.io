---
title: 修改django中request的参数以及django_filter支持表单数据                     
description: 修改django中request的参数以及django_filter支持表单数据
categories:
- python
tags:
- python   
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
from enum import Enum

from django_filters.filterset import BaseFilterSet, FilterSetMetaclass


class ContentTypeEnum(Enum):
    Json = "application/json"
    FormData = "application/x-www-form-urlencoded"
    TextPlain = "text/plain"

class FilterSetOwn(BaseFilterSet, metaclass=FilterSetMetaclass):
    def __init__(self, data=None, queryset=None, *, request=None, prefix=None):
        # 根据content-type 更新query_dict
        content_type = request.content_type.lower()
        if content_type == ContentTypeEnum.Json.value:
            filter_data_ = request.data
            if not isinstance(filter_data_, dict):
                filter_data_ = {}
            filter_data_ = self._change_to_querydict(filter_data_)

        elif content_type == ContentTypeEnum.FormData.value:
            filter_data_ = request.POST
            if not isinstance(filter_data_, QueryDict):
                filter_data_ = {}
            filter_data_ = dict(filter_data_)
            for k, v in filter_data_.items():
                filter_data_[k] = v[0].split(',')
            filter_data_ = self._change_to_querydict(filter_data_)

        else:
            filter_data_ = {}
        if request.method.lower() == 'get':
            setattr(request.query_params, '_mutable', True)
            request.query_params.update(filter_data_)
            # 没有GET传参 则解析 application/x-www-form-urlencoded
            setattr(request.query_params, '_mutable', False)
        super(FilterSetOwn, self).__init__(data=data, queryset=queryset, request=request, prefix=prefix)

    def _change_to_querydict(self, filter_dict):
        query_dict = QueryDict('', mutable=True)
        for key, value in filter_dict.items():
            if isinstance(value, list):
                value = [str(item) for item in value]
            d = {key: value}
            query_dict.update(MultiValueDict(d) if isinstance(value, list) else d)
        return query_dict


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

#### `django-filter` 使用


```
# views.py
class VSDemo(CustomModelViewSet,  MixGetReqeustContext):
    queryset = Model.objects.all()
    serializer_class = Serializers
    filter_backends = (SearchFilter, DjangoFilterBackend)
    filter_class = FilterOwn
    search_fields = ['..', ...]
    
    @action(methods=['get'], url_path='a-s', detail=False)
    def list_count(self, request, *args, **kwargs):
        self.serializer_class = SerializersV2
        self.filter_class = FilterV2
        ret_data = super().list(request, *args, **kwargs)
        return ret_data

# filters.py
class FilterV2(FilterSet):
   
    field1 = filters.MultipleChoiceFilter(
        field_name='db_field1',
        choices=[],
        widget=widgets.QueryArrayWidget
    )
    field2 = CharInFilter(
        field_name='db_field2',
        lookup_expr='in',
        widget=widgets.QueryArrayWidget
    )
    field3 = filters.BooleanFilter(field_name='db_field3')

    field4 =  filters.CharFilter(method='filter_name', widget=widgets.QueryArrayWidget)
   
    def filter_name(self, queryset, name, value):
        qs = queryset
        print(name, value) #  field4, ['a', 'b']
        return qs



    class Meta(object):
        fields = [
           ...
        ]
        model = Model

```
