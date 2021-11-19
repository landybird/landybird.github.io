---
title: Django bulk_create 中 ignore_conflicts 主键自增 与 Mysql 主键的长度限制
description: Mysql 主键自增超过 `pk`的长度限制
categories: 
- python    
tags:
- python   
---

### 1 主键一直增加超过 `int(11)` 范围

#### 背景: 

使用 Django 的 `bulk_create` 批量创建数据
由于 `bulk_create(bulk_data, ignore_conflicts=True)` 会一直覆盖数据(unique重复)，导致主键会一直增加, 超过限制


[Django--bulk_create](https://docs.djangoproject.com/en/3.2/ref/models/querysets/)


[bulk_create](https://github.com/django/django/blob/main/django/db/models/query.py#463)


[SQLInsertCompiler](https://github.com/django/django/blob/main/django/db/models/sql/compiler.py)
最后执行的语句是

```
with self.connection.cursor() as cursor:
    for sql, params in self.as_sql():
        cursor.execute(sql, params)

```

```
INSERT IGNORE INTO `xxx` (`month`, `system_value`, `checked_value`,
`created_time`, `updated_time`, `account_id`) VALUES (%s, %s, %s, %s, %s, %s) ('2021-03
-01', '4281.36', None, '2021-05-08 03:12:26.530947', '2021-05-08 03:12:26.530947', 8)

```

`MySQL 5.1.22+` `innodb_autoinc_lock_mode = 0` (“traditional” lock mode)


    INSERT INTO ...  ON DUPLICATE KEY UPDATE ...

    INSERT IGNORE INTO  ...
    
    REPLACE ...
    
    都会让主键自增

[AUTO_INCREMENT Handling in InnoDB](https://dev.mysql.com/doc/refman/5.7/en/innodb-auto-increment-handling.html#innodb-auto-increment-lock-modes)


#### 报错:

    Duplicate entry '2147483647' for key 'PRIMARY'
    
    1467 - Failed to read auto-increment value from storage engine 超过int(11)的数据范围
    

        bigint
        
            从 -2^63 (-9223372036854775808) 到 2^63-1 (9223372036854775807) 的整型数据（所有数字）。存储大小为 8 个字节。
            
            P.S. bigint已经有长度了，在mysql建表中的length，只是用于显示的位数
        
        int
        
            从 -2^31 (-2,147,483,648) 到 2^31 – 1 (2,147,483,647) 的整型数据（所有数字）。存储大小为 4 个字节。int 的 SQL-92 同义字为 integer。
        
        smallint
        
            从 -2^15 (-32,768) 到 2^15 – 1 (32,767) 的整型数据。存储大小为 2 个字节。
        
        tinyint
        
            从 -128 到 128 的整型数据。存储大小为 1 字节。



```sql


SHOW CREATE TABLE `spend_for_month_not_bv`;

CREATE TABLE `spend_for_month_not_bv` (
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `created_time` DATETIME(6) NOT NULL,
  `updated_time` DATETIME(6) NOT NULL,
  `system_value` DECIMAL(16,2) NOT NULL,
  `checked_value` DECIMAL(16,2) NOT NULL,
  `account_id` INT(11) NOT NULL,
  `month` DATE DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `account,month` (`account_id`,`month`)
) ENGINE=INNODB AUTO_INCREMENT=2147483647  DEFAULT CHARSET=utf8

```


#### 解决方法  

[stackoverflow](https://stackoverflow.com/questions/7346934/mysql-failed-to-read-auto-increment-value-from-storage-engine)


      1 扩展主键的范围
      
      2 将表初始化 truncate, 重新录入数据
      
            DROP TABLE IF EXISTS `spend_for_month_copy` ;

            CREATE TABLE `spend_for_month_copy`  LIKE `spend_for_month`;

            INSERT INTO `spend_for_month_copy` 
            SELECT * FROM `spend_for_month`;

            TRUNCATE `spend_for_month`;


            INSERT INTO `spend_for_month`(created_time,updated_time,system_value,checked_value,account_id,sale_leader_id,MONTH) 
            SELECT created_time,updated_time,system_value,checked_value,account_id,sale_leader_id,MONTH 
            FROM `spend_for_month_copy`;

      
      
      3 不使用强制覆盖的方式， 只创建没有的数据
      
      
      


### 2 `auto-increment` < `当前最大ID` "Duliplicate key" 问题 (主从数据不一致) `mysql5.6+`


[深度解析auto-increment自增列"Duliplicate key"问题](https://mp.weixin.qq.com/s/YjPOayOFwCPwH084SG2Ggw)

[REPLACE操作导致主从库AUTO_INCREMENT不一致的分析](https://yangwuyuan.com/2019/05/10/REPLACE%E6%93%8D%E4%BD%9C%E5%AF%BC%E8%87%B4%E4%B8%BB%E4%BB%8E%E5%BA%93AUTO-INCREMENT%E4%B8%8D%E4%B8%80%E8%87%B4%E7%9A%84%E5%88%86%E6%9E%90/#REPLACE%E6%93%8D%E4%BD%9C%E5%AF%BC%E8%87%B4AUTO-INCREMENT%E5%80%BC%E4%B8%8D%E4%B8%80%E8%87%B4)


#### 背景: 

在某些情况下，`replace操作`将导致`主从库auto_increment值不一致`，
如果此时发生切换，将可能导致`数据无法插入`的问题


    replace的语义 ”It either inserts, or deletes and inserts.”
    
    Mysql对于replace into实际是通过delete + insert语句实现，但是在ROW binlog格式下，会向binlog记录update类型日志。
    
    Insert语句会同步更新autoincrement，update则不会




#### 报错原因:

`replace into`在`Master`上按照`delete+insert`方式操作， `autoincrement`就是正常的。
基于`ROW格式`复制到`slave`后，`slave`机上按照`update`操作回放，只更新行中自增键的值，不会更新`autoincrement`。
因此在`slave`机上就会出现`max(id)`大于`autoincrement`的情况。

此时在`ROW`模式下对于`insert`操作`binlog`记录了所有的列的值，
在`slave`上回放时并不会重新分配自增id，因此不会报错。但是如果`slave切master`，遇到`Insert操作`就会出现`”Duplicate key”`的错误。


#### 解决方法  

业务侧的可能解决方案：

    (1) binlog改为mixed或者statement格式
    (2) 用Insert on duplicate key update代替replace into

内核侧可能解决方案：
    
    (1) 在ROW格式下如果遇到replace into语句，则记录statement格式的logevent，将原始语句记录到binlog。
  
    (2) 在ROW格式下将replace into语句的logevent记录为一个delete event和一个insert event。
    
升级` mysql 8+`

    mysql8.0版本中不仅将AUTO_INCREMENT值做了持久化，且在做更新操作时，
    如果表上的自增列被更新为比auto_increment更大的值，auto_increment值也将被更新
