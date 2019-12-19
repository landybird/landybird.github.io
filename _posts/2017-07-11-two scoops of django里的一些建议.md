---
title: two scoops of django里的一些建议
description: two scoops of django里的一些建议
categories:
 - Django
tags:
- Django
---


# two scoops of django里的一些建议

<br>

##  1  一个app中的model数量

**一个app中不超过5个model**

    If there are 20+ models in a single app, think about ways to break it down into smaller apps, as it
    probably means your app is doing too much. 
    
    In practice, we like to lower this number to no more  than five models per app.



<br>

##  2  model的继承问题


**django 中的 三种继承**

    abstract base classes, 
    multi-table inheritance, 
    proxy models


| Model Inheritance Style继承类型 | pros 优点 | cons缺点 |
| --- | --- | --- |
| 1  没有继承 （同一个字段每个表都拥有） | 容易理解 model与数据库的映射关系 | 不容易维护 |
| 2  抽象基类继承 （该类的衍生类 才会创建表，本身不会创建） | 写一个系统不会生成表的 TimeStampModel () ；而真正要生成的表用来继承 这个表 | 子类不能与父类的字段重复；不能单独使用父类；|
| 3 多表继承 （父类子类都会产生新表） | 自动创建的OneToOneField 关系； | 每次查询 都会 进行连表， 效率低  (不建议使用) |
| 4 代理继承 （为原始 model 创建一个代理(proxy） | 可以在代理 model 中改变默认的排序设置和默认的 manager 管理器 |  不能改变父类中的字段 |


<br>

##  3  什么时候使用数据库事务操作


|  动作 |   ORM method  | 通常 是否使用事务 |
| --- | --- | --- |
| Create Data | .create(), .bulk_create(), .get_or_create(),  | v |
|Retrieve Data | .get(), .filter(), .count(), .iterate(), .exists(),.exclude(), .in_bulk, etc. | x |
|Modify Data | .update() | v |
|Delete Data | .delete() | v |
              
                 
<br>

##  4  包导入时需要注意的问题

- 尽量避免在项目中使用 绝对路径导入， 可能项目会用到其他地方， 使用相对路径会好些
   
- 避免使用  import *  , 在不同的包中可能有命名冲突
    
    
>解决命名冲突的办法  
    
    from django.db.models import CharField as ModelCharField
    from django.forms import CharField as FormCharField
