---
title: php中mod_php模式和mod_fastcgi模式
description: php中mod_php模式和mod_fastcgi模式
categories:
- php
tags:
- php
---

<br>


# php中mod_php模式和mod_fastcgi模式

<br>


## 1  mod_php模式 （apache 中使用）


`mod_php 模式`会将php模块安装到apache下面来运行，2者结合度较大。出问题时很难定位是php的问题还是apache的问题。

常见的web服务器	
    
    apache
    nginx
    IIS
    lighttpd
    tomcat
    
mod_php模式下，apache调用php执行的过程如下：

    apache -> httpd -> php5_module -> sapi -> php
 
    
> apache + php + mysql 的 完整的访问流程
 
 
 ![](https://landybird.github.io/landybird.github.io/assets/images/a1.jpg)

  
>`apache` 用 LoadModule 来加载 php5_module ( mod_php模式 )

    把php作为apache的一个子模块来运行。当通过web访问php文件时，apache就会调用php5_module来解析php代码

```php
         //加入以下2句
        LoadModule php5_module D:/php/php5apache2_2.dll
        AddType application/x-httpd-php .php
        
        
        //将下面的
        <IfModule dir_module>
            DirectoryIndex index.html
        </IfModule>
        //将其修改为：
        <IfModule dir_module>
            DirectoryIndex index.html index.htm index.php index.phtml
        </IfModule>
```
    
    


>`sapi` 是一个中间过程，SAPI提供了一个和外部通信的接口，有点类似于socket，使得PHP可以和其他应用进行交互数据（apache,nginx,cli等）。

    php默认提供了很多种SAPI，常见的给apache和nginx的php5_module，CGI，给IIS的ISAPI，还有Shell的CLI。  


 ![](https://landybird.github.io/landybird.github.io/assets/images/a2.jpg)
    



<br>

## 2  mod_fastcgi模式


`mod_fastcgi模式`则是作为`一个中间过程`，apache接收用户请求后，就发送给fastcgi, 再连接php来完成访问。

php的sapi的另一种方式就是提供 `cgi模式`

cgi就是专门用来和web 服务器打交道的。web服务器收到用户请求，就会把请求提交给cgi程序（php的fastcgi），
cgi的好处就是完全独立于任何服务器，仅仅是做为中间分子。提供接口给apache和php


    每一次web请求都会有启动和退出过程，--为人诟病的fork-and-execute模式，
    
    由于cgi比较老所以就出现了fastcgi来取代  启动多个cgi模块，在那里一直运行着等着，等着web发过来的请求，
    然后再给php解析运算完成生成html给web后，也不会退出，而且继续等着下一个web请求

 ![](https://landybird.github.io/landybird.github.io/assets/images/a3.png)



<br>

## 3  PHP-FPM 模式  （Nginx中使用 ）

PHP在 5.3.3 之后已经讲php-fpm写入php源码核心了


PHP专用的 fastcgi管理器， 来`辅助mode_fastcgi模式`     [地址](https://php-fpm.org/about/)





>  php-fpm安装

    要想使php支持php-fpm，只需要在编译的时候带上 --enable-fpm 就可以了。
 

    获取之前的编译参数
    
        php  -i  | grep 'Configure' 
        
        php -i | grep "Configure" | 'fpm'  查看是否有 fpm
        
    没有的话  需要重新编译  


>  启动 php-fpm

        
        /usr/local/php/sbin/php-fpm
        
        -->>   /usr/local/php/etc目录下将php-fpm.conf.default拷贝也一份成php-fpm.conf



>  查看启动的状态 ;	
 
        ps -ef | grep php-fpm
