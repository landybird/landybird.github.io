---
title: Yii 获取当前的用户和用户的权限 getId() getRoles()
description: Yii 获取当前的用户和用户的权限
categories:
- php
tags:
- Yii
---

<br>


# Yii 获取当前的用户和用户的权限 getId() getRoles()：

<br>


```php

    <?php
    
        $userId  = Yii::app()->user->getId();   //获取当前的用户ID
        $username  = Yii::app()->user->name;   //获取当前的用户名
        
        $re = Yii::app()->authManager->getRoles($userId);    // 获取当前的用户的权限
        
        var_dump(array_key_exists('role_super',$re));  //	判断是否在权限数组中
    
    ?>

```
