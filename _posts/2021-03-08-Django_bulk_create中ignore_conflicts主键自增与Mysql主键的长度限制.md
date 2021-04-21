---
title: Django bulk_create 中 ignore_conflicts 主键自增 与 Mysql 主键的长度限制
description: Mysql 主键自增超过 `pk`的长度限制
categories: 
- python    
tags:
- python   
---

#### 背景: 

使用 Django 的 `bulk_create`, `bulk_update` 批量修改, 创建数据
由于 `bulk_create(bulk_data, ignore_conflicts=True)` 会一直覆盖数据， 导致主键会一直增加知道超过限制

[Django--bulk_create](https://docs.djangoproject.com/en/3.2/ref/models/querysets/)


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
        
            从 0 到 255 的整型数据。存储大小为 1 字节。



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
      
      
      
