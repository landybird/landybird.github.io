---
title: Django在FBV与CBV模式下如何实现登录控制和csrf控制
description: Django实现登录控制和csrf控制
categories:
- Django
tags:
- Django
---

<br>



# Django在FBV与CBV模式下如何实现登录控制和csrf控制

<br>

##  Django 实现 csrf验证的方式 

**去请求体 或者 cookie中获取 token 验证**

- **(1) csrf_token 中间件 -- 全局配置**
**基于中间件的process_view函数中实现**

- **(2) 视图函数中 -- 通过装饰器 局部实现**
**(在视图中是否被 csrf_exempt，csrf_protect装饰器 装饰)**


<br>

### FBV模式下的 csrf 验证与免验证

```python
函数 装饰器

  from django.views.decorators.csrf import csrf_exempt
  @csrf_exempt

  from django.views.decorators.csrf import csrf_protect
  @csrf_protect
```

<br>

### CBV模式下的 csrf 验证与免验证 method_decorators---需要装饰dispatch函数

**方式一  写在基类中，继承**

```python
	from django.utils.decorators import method_decorator
	
	class MyBaseView(object):
	    @method_decorator(csrf_exempt|csrf_protect)
	    def dispatch(self, request, *args, **kwargs):
	        print('before')
	        ret = super(MyBaseView,self).dispatch(request, *args, **kwargs)
	        print('after')
	        return ret
	
	class LoginView(MyBaseView,APIView):
	
	    def get(self,request,*args,**kwargs):
	
	        ret = {
	            'code':1000,
	            'data':'ddd'
	        }
	        response = JsonResponse(ret)
	        return response
```

	
**方式二  指定 name = 'dispatch' 属性**

```python
	@method_decorator(csrf_protect,name='dispatch')
	class LoginView(MyBaseView,APIView):
	
	    def get(self,request,*args,**kwargs):
	
	        ret = {
	            'code':1000,
	            'data':'ddd'
	        }
	        response = JsonResponse(ret)
	        return response
```

**方式三  直接在视图中 重写 dispatch ，加装饰器**
	
```python
	class LoginView(MyBaseView,APIView):
	    
	    @method_decorator(csrf_protect)
	    def dispatch(self, request, *args, **kwargs):
	        print('before')
	        ret = super(LoginView,self).disptch(request, *args, **kwargs)
	        print('after')
	        return ret
	    
	    def get(self,request,*args,**kwargs):
	
	        ret = {
	            'code':1000,
	            'data':'ddd'
	        }
	        response = JsonResponse(ret)
	        return response
```

##  Django 实现登录控制的方式 



### 基于FBV -- login_reqeuired 装饰器

```python
from django.contrib.auth.decorators import login_required  
 
@login_required  
def my_view(request):  
    ...  
```

<br>

### 基于CBV



**(1)  继承基类**
   
 ```python
        from django.contrib.auth.decorators import login_required
        from django.utils.decorators import method_decorator

        class LoginRequiredMixin(object):

            @method_decorator(login_required(login_url=reverse('login')))
            def dispatch(self,request,*args,**kwargs):
                return super(LoginRequiredMixin,self).dispatch(self,request,*args,**kwargs)

        class CourseView(LoginRequiredMixin,View):
                pass
```

**(2)  method_decorator + name = 'dispatch'**

```python
        @method_decorator(login_required(login_url=reverse('login')),name='dispatch')
        class CourseView(View):
                    pass
```

**(3)  直接写在视图中，重载 dispatch 函数**

```python
        class CourseView(View):
              @method_decorator(login_required(login_url=reverse('login')))
              def dispatch(self,request,*args,**kwargs):
                     return super(Course,self).dispatch(self,request,*args,**kwargs)
```


<br>

>**补充**

**反射的应用：**
    
- (1) CBV  -- dispatch方法中通过method获取 方法

- (2) 中间件的导入  'dasdsa.aDd.xxxmiddleware'  -- 导入settings
    
```python
            from importlib import import_module
            module = import_module(module_path) 
            cls = getattr(module,class_name)
            cls() # 类的实例化
```
