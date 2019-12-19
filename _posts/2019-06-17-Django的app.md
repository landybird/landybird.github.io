---
title: Django的app 
description: Django的app
categories:
- python
tags:
- Two_Scoops_of_Django_Best_Practices_for_Django_2017
---


> Django中的几个概念




`A Django project`
 
    is a web application powered by the Django web framework.
    
`Django apps` 

    are small libraries designed to represent a single aspect of a project. A Django project
    is made up of many Django apps. Some of those apps are internal to the project and will never
    be reused; others are third-party Django packages.
    
`INSTALLED_APPS`

    is the list of Django apps used by a given project available in its INSTALLED_APPS setting.
    
`Third-party Django packages`
 
     are simply pluggable, reusable Django apps that have been packaged
     with the Python packaging tools.


Django 中的 app 應該精简在自己的业务上

    each app should be tightly focused on its task
    
    
app的布局 0

```python
# Common modules
scoops/
├── __init__.py
├── admin.py
├── forms.py
├── management/
├── migrations/
├── models.py
├── templatetags/
├── tests/
├── urls.py
├── views.py

```


app的布局 1

```python

# uncommon modules
scoops/
├── api/
├── behaviors.py
├── constants.py
├── context_processors.py
├── decorators.py
├── db/
├── exceptions.py
├── fields.py
├── factories.py
├── helpers.py
├── managers.py
├── middleware.py
├── signals.py
├── utils.py
├── viewmixins.py
```
