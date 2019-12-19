---
title: Yii框架中activeRecord的save
description: Yii框架中activeRecord的save
categories:
- php
tags:
- Yii
---

<br>


# Yii框架中activeRecord的save：



```php

    public function save($runValidation=true,$attributes=null)
    {
        if(!$runValidation || $this->validate($attributes))
            return $this->getIsNewRecord() ? $this->insert($attributes) : $this->update($attributes);
        else
            return false;

```


> 默认save 会执行 rules中定义的验证规则


```php
    public function rules()
        {
            // NOTE: you should only define rules for those attributes that
            // will receive user inputs.
            return array(
                array('a, b, c, d', 'numerical', 'integerOnly'=>true),
                array('a, v, d, d, x', 'length', 'max'=>11),
                array('email', 'length', 'max'=>128),
                array('dddd', 'length', 'max'=>255),
                array('email', 'checkEmail'),  
                //自定义验证规则
                array('xxx, email', 'required', 'message'=>'{attribute}是必填项'),
                array('xxx', 'checkCompanyIdExist'),
                array('ddd', 'in', 'range' => array(0, 1, 2)),
                
                // The following rule is used by search().
                // Please remove those attributes that should not be searched.
                array('xx, d, email, uid, create_time', 'safe', 'on'=>'search'),
            );
        }

```



> Yii 中 AR对象保存的时候 返回false， 原因可能是验证没有通过 ( 或者重写的生命周期中有问题 )


- AR生命周期方法的返回值有误


        yii\db\ActiveRecord::beforeValidate(): 
        yii\db\ActiveRecord::afterValidate() 
        yii\db\ActiveRecord::beforeSave() 
        yii\db\ActiveRecord::afterSave()



查看 错误信息 `$model->errors`

这时候需要 `$model->save(false)`
