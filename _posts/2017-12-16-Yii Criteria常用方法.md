---
title: Yii Criteria常用方法
description: Yii Criteria的常用方法
categories:
- php
tags:
- Yii
---

<br>


# Yii Criteria常用方法：

<br>

> 1 $criteria = new CDbCriteria();   
 
    建立一个查询标准，可以加条件，排序等等。



<br>

> 2 select     
 
	$criteria->select = '*';//默认*  
    $criteria->select = 'id,name';//指定的字段  
    $criteria->select = 't.*,t.id,t.name';//连接查询时，第一个表as t,所以用t.*  
    $criteria->distinct = FALSE; //是否唯一查询



<br>

> 3 join     
 
	$criteria->join = 'left join table2 t2 on(t.id=t2.tid)'; //连接表   
	$criteria->alias = 'Invoice'; 
    $criteria->with = 'xxx'; //调用relations  
    // 注意 两个表重复的字段 需要区分 



<br>

>4 where 

    //where 查询数字字段  
    $criteria->addCondition("id=1"); //查询条件，即where id = 1    
    $criteria->addBetweenCondition('id', 1, 4);//between 1 and 4       
    $criteria->addInCondition('id', array(1,2,3,4,5)); //代表where id IN (1,23,,4,5,);    
    $criteria->addNotInCondition('id', array(1,2,3,4,5));//与上面正好相法，是NOT IN  
     
     
    //where 查询字符串字段  
    $criteria->addSearchCondition('name', '分类');//搜索条件，其实代表了。。where name like '%分类%'   
      
      
    //where 查询日期字段  
    $criteria->addCondition("create_time>'2012-11-29 00:00:00'");  
    $criteria->addCondition("create_time<'2012-11-30 00:00:00'");  
     
    

<br>

>5 传递变量 

    $criteria->addCondition("id = :id");  
    $criteria->params[':id']=1;  
    
    
<br>

>6 一些public属性 

    $criteria->select = 'id,parentid,name'; //代表了要查询的字段，默认select='*';  
    $criteria->join = 'xxx'; //连接表  
    $criteria->with = 'xxx'; //调用relations   
    $criteria->limit = 10;    //取1条数据，如果小于0，则不作处理  
    $criteria->offset = 1;   //两条合并起来，则表示 limit 10 offset 1,或者代表了。limit 1,10  
    $criteria->order = 'xxx DESC,XXX ASC' ;//排序条件  
    $criteria->group = 'group 条件';  
    $criteria->having = 'having 条件 ';  
    $criteria->distinct = FALSE; //是否唯一查询   


<br>

>7 compare 精确与模糊查询 

  $criteria->compare($key,$val,true);  //模糊查询
  $criteria->compare($key,$val);  //精确查询
  //  这个方法比较特殊，他会根据你的参数自动处理成addCondition或者addInCondition，  
