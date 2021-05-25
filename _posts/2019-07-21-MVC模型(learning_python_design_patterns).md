---
title: MVC模型(learning_python_design_patterns)
description: MVC模型(learning_python_design_patterns)
categories:
- python
tags:
- learning python design patterns
---

<br>

一个app应用会随着需求，变得越来越庞大

`MVC  Model-View-Controller `模型可以更好的维护和修改

把一个应用分成三块

    Model    数据和处理
    
    View     视图 
    
    Controller  二者之间的联系
    
    
![](https://landybird.github.io/assets/images/lpdp1.png)


MVC模型在python web中的应用
    
    web2py, Pyramid, Django (uses a flavor of MVC called MVP),
    Giotto,

<br>

    smart models 
    
    thin controllers
    
    dumb views.


> Model 

Model是整个应用的基础, View 和 Controller 依赖于 Model


在`Model`中需要实现

      。创建数据模型 和 相关的接口  
      。验证数据， 并且将错误返回给 controller
 
    • Strive to perform the following for models:
        Create data models and interface of work with them
        Validate data and report all errors to the controller
        
    • Avoid working directly with the user interface



> View 

视图从 Controller中获取处理后的数据， 然后展示

不应该包含复杂的逻辑， 逻辑处理应该在 Model 和 Controller 中完成

在`View`中需要实现

    简化， 展示

    
    • Strive to perform the following for views:
       ° Try to keep them simple; use only simple comparisons and loops
    
    • Avoid doing the following in views:
       ° Accessing the database directly
       ° Using any logic other than loops and conditional statements (if-thenelse) because the separation of concerns requires all such complex
          logic to be performed in models
    

> Controller

从请求获取数据， 传递数据
    
    • Strive to perform the following in controllers:
        ° Pass data from user requests to the model for processing, retrieving
            and saving the data
        ° Pass data to views for rendering
        ° Handle all request errors and errors from models
   
    • Avoid the following in controllers:
        ° Render data
        ° Work with the database and business logic directly
        
实例：

`View`

```html

//views/main_page.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/shorten/">
 <label>
 <input type="text" name="url" value="" />
 Link to shorten
 </label>
 <input type="submit" value="OK"/>
</form>
</body>
</html>

//views/success.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
Congratulations! <br />
Your url: {{ short_url }}
</body>
</html>

```

`Model`

```python

# models.py


# coding: utf-8
import pickle

class Url(object):
    @classmethod
    def shorten(cls, full_url):
        """Shortens full url."""
        # Create an instance of Url class
        instance = cls()
        instance.full_url = full_url
        instance.short_url = instance.__create_short_url()
        Url.__save_url_mapping(instance)
        return instance
    @classmethod
    def get_by_short_url(cls, short_url):
        """Returns Url instance, corresponding to short_url."""
        url_mapping = Url.load_url_mapping()
        return url_mapping.get(short_url)
    def __create_short_url(self):
        """Creates short url, saves it and returns it."""
        last_short_url = Url.__load_last_short_url()
        short_url = self.__increment_string(last_short_url)
        Url.__save_last_short_url(short_url)
        return short_url
    def __increment_string(self, string):
        """Increments string, that is:
        a -> b
        z -> aa
        az -> ba
        empty string -> a
        """
        if string == '':
            return 'a'
        last_char = string[-1]
        if last_char != 'z':
            return string[:-1] + chr(ord(last_char) + 1)
        return self.__increment_string(string[:-1]) + 'a'

    @staticmethod
    def __load_last_short_url():
        try:
            return pickle.load(open("last_short.p", "rb"))
        except IOError:
            return ''

    @staticmethod
    def __save_last_short_url(url):
        pickle.dump(url, open("last_short.p", "wb"))

    @staticmethod
    def __load_url_mapping():
        try:
            return pickle.load(open("short_to_url.p", "rb"))
        except IOError:
            return {}

    @staticmethod
    def __save_url_mapping(instance):
        short_to_url = Url.__load_url_mapping()
        short_to_url[instance.short_url] = instance
        pickle.dump(short_to_url, open("short_to_url.p", "wb"))

``` 

`Contoller`

```python

# controller.py

# coding: utf-8
# Redirect function is used to forward user to full url if he came
# from shortened
# Request is used to encapsulate HTTP request. It will contain request
# methods, request arguments and other related information
from flask import redirect, render_template, request, Flask
from werkzeug.exceptions import BadRequest, NotFound
import models
# Initialize Flask application
app = Flask(__name__, template_folder='views')
@app.route("/")
def index():
    return render_template('main_page.html')

@app.route("/shorten/")
def shorten():
    # Validate user input
    full_url = request.args.get('url')
    if not full_url:
        raise BadRequest()
    # Model returns object with short_url property
    url_model = models.Url.shorten(full_url)
    # url_model.short_url
    # Pass data to view and call its render method
    short_url = request.host + '/' + url_model.short_url
    return render_template('success.html', short_url=short_url)

@app.route('/<path:path>')
def redirect_to_full(path=''):

    # Model returns object with full_url property
    url_model = models.Url.get_by_short_url(path)
    # Validate model return
    if not url_model:
        raise NotFound()
    return redirect(url_model.full_url)

if __name__ == "__main__":
    app.run(debug=True)

```