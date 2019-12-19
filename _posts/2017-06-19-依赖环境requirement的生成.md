---
title: 项目依赖环境requirement.txt的生成
description: 项目规范
categories:
- 项目规范
tags:
- 项目规范
---

<br>


**在项目完成之后，需要生成项目依赖的所有 安装包**

<br>

###  方法一虚拟环境下 pip freeze > requirements.txt

<br>

**这样会把所有虚拟环境中的安装包全部写入,当虚拟环境中只有一个项目时，可以直接使用**

```python
1 在项目的虚拟环境中(确保每个项目对应一个虚拟环境)

2 进入项目后

3 使用 pip freeze > requirements.txt 生成依赖文件

4 再次使用的时候 pip install -r requirements.txt

```
	

<br>

###  方法二 项目下 pipreqs ./  

**安装 `pip install pipreqs`**

```python
1 进入项目后

2 使用  pipreqs ./ 生成项目所需的依赖包

3 再次使用的时候 pip install -r requirements.txt

```
