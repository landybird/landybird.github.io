---
title: python 构建一个package包
description: python 构建一个package包
categories:
 - python
tags:
- python基础
---


# python 构建一个package包

<br>


文件结构：

    mypkg
        +--- mypkg
        |   +--- __init__.py  
        +--- README.md
        +--- LICENSE
        +--- setup.py
        
        
>  `__init__.py`
        
        
      name = 'mypkg'
      
        
>  `setup.py ` 
    
    
     build script for setuptools; 


```python
    
     import setuptools
     
     with open("README.md", "r") as fh:
         long_description = fh.read()
     
     setuptools.setup(
         name="mypkg",    #   
         version="0.0.1",
         author="Author",
         author_email="author@example.com",
         description="A small example package",
         long_description=long_description,
         long_description_content_type="text/markdown",
         url="https://github.com/pypa/sampleproject",
         packages=setuptools.find_packages(),
         classifiers=[
             "Programming Language :: Python :: 3",
             "License :: OSI Approved :: MIT License",
             "Operating System :: OS Independent",
         ],
     )

```     

>  `README.md`


        # Example Package
        
        This is a simple example package. You can use
        [Github-flavored Markdown](https://guides.github.com/features/mastering-markdown/)
        to write your content.  
    

>  `LICENSE`

[选择一个LICENSE]( https://choosealicense.com/)

    
    Copyright (c) 2018 The Python Packaging Authority
    
    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:
    
    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.
    
    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
    
    

>  编译

```python

 >> 保证 setuptools, wheel 是最新的版本
    python3 -m pip install --user --upgrade setuptools wheel

 >> 在setup.py的同级目录下 执行：
     python3 setup.py sdist bdist_wheel

 

```


> 在test PyPi 中测试上传 


[测试地址]( https://test.pypi.org/account/register/) 


       注册




>  测试上传


```python

    python3 -m pip install --user --upgrade twine
    
    python3 -m pip install --user --upgrade twine

```


>  正式上传， 类似测试上传


>  更新版本

    修改 setup.py 中的 version 
    
    重新上传
    
        python setup.py sdist bdist_wheel
        
        twine upload dist\lucky_choice_jinli_v1-0.0.3*


    更新本地module版本

        pip install --upgrade  lucky_choice_jinli 
            


## 实例

[获取最热的磁链 hot-magnet](https://pypi.org/project/hot-magnet/#description)




