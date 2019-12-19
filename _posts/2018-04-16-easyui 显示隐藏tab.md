---
title: easyui 显示隐藏tab
description: easyui 显示隐藏tab
categories:
- 前端
tags:
- 前端
---

<br>


# easyui 显示隐藏tab


```php

        <?php
        
        $userId  = Yii::app()->user->getId();
        $re = Yii::app()->authManager->getRoles($userId);
        
        $is_ae = array_key_exists('role_ae',$re);        
        
        if($is_ae){
            echo "<script>$('#tt').tabs('getTab','待确认').panel('options').tab.hide();</script>";
            //  tab组件的id #tt，
            echo "<script>$('#tt').tabs('getTab','已撤销').panel('options').tab.hide();</script>";
            
            echo "<script>$('#待确认').hide();</script>"; //隐藏tab标签下的内容
        }
        
        ?>
        
        
        




```
