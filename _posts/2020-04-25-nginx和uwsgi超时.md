---
title: nginx和uwsgi超时                        
description: nginx和uwsgi超时
categories:
- python
tags:
- python   
---

#### `uwsgi` 的设置

- `http-timeout`和`socket-timeout`（连接时间） 


         http-timeout=60 # 就是60秒
        
一般情况下，`uwsgi`都是配合`nginx`使用的，所以用的都是`socket-timeout`参数。

    uwsgi单独使用就用http 
    
    配合nginx就用socket


    例子:  
            服务器需要运行5分钟才能给前端返回响应，
	        但是http-timeout或者socket-timeout设置的是60s，
	        一分钟后，前端和后端开连接
	        ( 服务器还会坚持把这5分钟的任务执行完，但是不会给前端返回)
	
	
- `harakiri`(服务器响应时间）



          	  harakiri=60 # 就是60秒




		前端（客户端）向后端（服务器）发送到一个请求，等待服务器响应，服务器
			需要1分钟来计算数据，但是我的harakiri就设置了10秒，那么10秒一到，
			我们的服务器就强制终止了计算，前端肯定就得不到响应了。

请求等待服务器响应的时间


	
	
- `buffer-size`（前后端传输数据大小）


        
            buffer-size=1024 # 就是1024k，1M




#### `Nginx`的配置


使用`uwsgi`

```
 location / {
        # 注意这儿，一般这三个配套修改
        uwsgi_send_timeout 600;        # 指定向uWSGI传送请求的超时时间，完成握手后向uWSGI传送请求的超时时间。
        uwsgi_connect_timeout 600;     # 指定连接到后端uWSGI的超时时间。
        uwsgi_read_timeout 600;        # 指定接收uWSGI应答的超时时间，完成握手后接收uWSGI应答的超时时间。
        uwsgi_pass  127.0.0.1:8000;
        include /home/jason/code/interface/conf/uwsgi_params;       # the uwsgi_params file you installed
    }


# 
        # 注意这儿，一般这三个配套修改
        proxy_send_timeout 600;
        proxy_connect_timeout 600;
        proxy_read_timeout 600;




```






- keepalive_timeout

```
Syntax:	keepalive_timeout timeout [header_timeout];
Default:	
keepalive_timeout 75s;
Context:	http, server, location
```

The first parameter sets a timeout during which a keep-alive client connection will stay open on the server side.

The zero value disables keep-alive client connections. 
The optional second parameter sets a value in the “Keep-Alive: timeout=time” response header field. 
Two parameters may differ.

The “Keep-Alive: timeout=time” header field is recognized by Mozilla and Konqueror.
MSIE closes keep-alive connections by itself in about 60 seconds.

- proxy_connect_timeout

```
Syntax:	proxy_connect_timeout time;
Default:	
proxy_connect_timeout 60s;
Context:	http, server, location
```

Defines a timeout for `establishing a connection` with a proxied server.

It should be noted that this timeout cannot usually exceed `75 seconds`.

- proxy_read_timeout

```
Syntax:	proxy_read_timeout time;
Default:	
proxy_read_timeout 60s;
Context:	http, server, location
```

Defines a timeout for `reading a response` from the proxied server. 

The timeout is set only between two successive read operations, not for the transmission of the whole response.
 
If the proxied server does not transmit anything within this time, the connection is closed.


- proxy_send_timeout

``` 
Syntax:	proxy_send_timeout time;
Default:	
proxy_send_timeout 60s;
Context:	http, server, location
```


Sets a timeout for `transmitting a request` to the proxied server. 

The timeout is set only between two successive `write operations`, not for the transmission of the whole request.
 
If the proxied server does not receive anything within this time, the connection is closed.
