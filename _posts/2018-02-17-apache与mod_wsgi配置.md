---
title: apache与mod_wsgi配置
description: apache与mod_wsgi配置
categories:
 - python
tags:
- python
---


# apache与mod_wsgi配置

<br>


# 1  配置 apache

```ini

apache_django.conf 文件

    ServerRoot "/usr/local/apache"                                                                          
    PidFile "/home/jiamin/logs/apache_django.pid"                                                           
    #PHPIniDir "/home/jiamin/logs/httpd.pid"                                                                
                                                                                                            
    LoadModule wsgi_module   modules/mod_wsgi.so  
    
    // 虚拟环境 >>  pip install mod-wsgi >> 指定 python home 和 python path
    // 注意： 出现没有apx异常， 需要sudo apt-get install apache2-dev                                                          
                                                                                                            
                                                                                                          
    Listen  18086                                                                                           
    #Listen 13666                                                                                           
                                                                                                            
    #ServerTokens Prod                                                                                      
    #ServerSignature Off                                                                                    
                                                                                                            
    MaxRequestsPerChild 1000                                                                                
                                                                                                                                                                                                                    
                                                                                                           
    ErrorLog "/home/jiamin/logs/error_log"                                                                  
    CustomLog "/home/jiamin/logs/test_access_log" combined (自定义log_format)                                                 
    LogLevel warn                                                                                           
                                                                                                            
    <VirtualHost *:18086>                                                                                   
                                                                                                            
        WSGIScriptAlias / /home/jiamin/jiamintest/apache_django/apache_django/wsgi.py                       
        ServerName 10.0.0.206                                                                               
        <Directory /home/jiamin/jiamintest/apache_django/apache_django>                                     
            <Files wsgi.py>                                                                                 
            Options FollowSymLinks                                                                          
            Order allow,deny                                                                                
            Allow from All                                                                                  
            </Files>                                                                                        
        </Directory>                                                                                        
    </VirtualHost>                                                                                          
```



  
 
 













