---
title: uwsgi配置与nginx配置
description: uwsgi配置与nginx配置
categories:
 - python
tags:
- python
---


# uwsgi配置与nginx配置

<br>


# 1  配置 uwsgi文件

```ini

uwsgi.ini 文件
    
[uwsgi]
#pythonpath=/usr/local/xxxx/xxxxx/project
#pythonpath=/usr/local/xxxx/xxxx/project/webroot
#static-map=/static=/usr/local/x/xx/project/static
chdir=/home/A/B/project
module=smb.wsgi
#env=DJANGO_SETTINGS_MODULE=conf.settings
#master=True
pidfile=logs/project.pid
vacuum=True
max-requests=1000
processes = 2
daemonize=logs/wsgi.log
http=0.0.0.0:8089
enable-threads=true

```

> 启动服务 `uwsgi --ini  uwsgi.ini`

> 使用 shell 启动服务 `sh uwsgi.sh start`

```javascript

#!/bin/bash                                           
                                                      
CURRDIR=`dirname "$0"`                                
BASEDIR=`cd "$CURRDIR"; pwd`                          
                                                      
source $BASEDIR/venv3.5/bin/activate    
              
                                                      
CMD=uwsgi                                             
                                                      
if [ "$1" = "-d" ]; then                              
    shift                                             
    EXECUTEDIR=$1'/'                                  
    shift                                             
else                                                  
    EXECUTEDIR=$BASEDIR'/'                            
fi                                                    
                                                      
if [ ! -d "$EXECUTEDIR" ]; then                       
    echo "ERROR: $EXECUTEDIR is not a dir"            
    exit                                              
fi                                                    
                                                      
if [ ! -d "$EXECUTEDIR"/conf ]; then                  
    echo "ERROR: could not find $EXECUTEDIR/conf/"    
    exit                                              
fi                                                    
                                                      
if [ ! -d "$EXECUTEDIR"/logs ]; then                  
    mkdir "$EXECUTEDIR"/logs                          
fi                                                    
                                                      
cd "$EXECUTEDIR"                                      
                                                      
PID_FILE="$EXECUTEDIR"/logs/project.pid            
                                                      
check_pid() {                                         
    RETVAL=1                                          
    if [ -f $PID_FILE ]; then                         
        PID=`cat $PID_FILE`                           
        ls /proc/$PID &> /dev/null                    
        if [ $? -eq 0 ]; then                         
            RETVAL=0                                  
        fi                                            
    fi                                                
}                                                     
                                                      
check_running() {                                     
    PID=0                                             
    RETVAL=0                                          
    check_pid                                         
    if [ $RETVAL -eq 0 ]; then                        
        echo "$CMD is running as $PID"                
        exit                                          
    fi                                                               
}                                                                    
                                                                     
start() {                                                            
    check_running                                                    
    "$CMD" --ini  "$EXECUTEDIR/conf/uwsgi.ini"                       
    sleep 1                                                          
    status                                                           
}                                                                    
                                                                     
stop() {                                                             
    echo "stopping .."                                               
    "$CMD" --stop $PID_FILE                                          
}                                                                    
                                                                     
status() {                                                           
    check_pid                                                        
    if [ $RETVAL -eq 0 ]; then                                       
        echo "uwsgi server is running as $PID ..."                   
    else                                                             
        echo "uwsgi server is not running"                           
    fi                                                               
}                                                                    
                                                                     
reload() {                                                           
    "$CMD" --reload $PID_FILE                                        
}                                                                    
                                                                     
RETVAL=0                                                             
case "$1" in                                                         
    start)                                                           
        start                                                        
        ;;                                                           
    stop)                                                            
        stop                                                         
        ;;                                                           
    restart)                                                         
        stop                                                         
    sleep 3                                                          
        start                                                        
        ;;                                                           
    status)                                                          
        status                                                       
        ;;                                                           
    reload)                                                          
        reload                                                       
        ;;                                                           
    *)                                                               
    echo "Usage: $0 {start|stop|restart|status|reload}"              
    RETVAL=1                                                         
esac                                                                 
exit $RETVAL                                                         

```

> 查看 logs/wsgi.log 信息

    *** Starting uWSGI 2.0.17.1 (64bit) on [Thu Jan 13 10:11:18 2018] ***
    .....
    机器配置信息
    .....
    
    
    current working directory: /home/aaaa/bbbbb/project
    writing pidfile to logs/project.pid
    detected binary path: /home/aaaa/.virtualenvs/project/bin/uwsgi
    chdir() to /home/aaaa/bbbbb/project
    
    *** WARNING: you are running uWSGI without its master process manager ***
    
    没有主进程管理
    
    your processes number limit is ...
    your memory page size is ... bytes
    detected max file descriptor number: ...
    lock engine: pthread robust mutexes
    thunder lock: disabled (you can enable it with --thunder-lock)
    
    uWSGI http bound on 0.0.0.0:8089 fd 4
    spawned uWSGI http 1 (pid: 31065)
    uwsgi socket 0 bound to TCP address 127.0.0.1:34015 (port auto-assigned) fd 3
    Python version: 3.5.2 (default, Aug  8 2016, 13:39:23)  [GCC 4.4.7 20120313 (CenOS 4.4.7-3)]
    Python main interpreter initialized at 0x1e25be0
    python threads support enabled
    your server socket listen backlog is limited to 100 connections
    your mercy for graceful operations on workers is 60 seconds
    mapped 145840 bytes (142 KB) for 2 cores
    
    *** Operational MODE: preforking ***
    WSGI app 0 (mountpoint='') ready in 1 seconds on interpreter 0x1e25be0 pid: 31062 (default app)
    *** uWSGI is running in multiple interpreter mode ***
    spawned uWSGI worker 1 (pid: 31062, cores: 1)
    spawned uWSGI worker 2 (pid: 31108, cores: 1)
    
    接收到的请求信息
    
    [pid: 31062|app: 0|req: 1/1] 请求IP () {40 vars in 1036 bytes} [Thu Jan 13 02:12:19 2018] GET /api/aaa/bbbb/cccccc/ => generated 14609 bytes in 1714 msecs (HTTP/1.1 200) 3 headers in 103 bytes (1 switches on core 0) 



# 配置 nginx文件



> 编写shell启动脚本


```javascript

#!/bin/bash                                                       
                                                                  
CURRDIR=`dirname "$0"`                                            
BASEDIR=`cd "$CURRDIR"; pwd`                                      
NAME="nginx"                                                      
CMD=/usr/local/aaa/prog.d/nginx-1.6.0/sbin/nginx                
                                                                  
if [ "$1" = "-d" ]; then                                          
    shift                                                         
    EXECUTEDIR=$1                                                 
    shift                                                         
else                                                              
    EXECUTEDIR=$BASEDIR                                           
fi                                                                
                                                                  
if [ ! -d "$EXECUTEDIR" ]; then                                   
    echo "ERROR: $EXECUTEDIR is not a dir"                        
    exit                                                          
fi                                                                
                                                                  
if [ ! -d "$EXECUTEDIR"/conf ]; then                              
    echo "ERROR: could not find $EXECUTEDIR/conf/"                
    exit                                                          
fi                                                                
                                                                  
if [ ! -d "$EXECUTEDIR"/logs ]; then                              
    mkdir "$EXECUTEDIR"/logs                                      
fi                                                                
                                                                  
cd "$EXECUTEDIR"                                                  
                                                                  
PID_FILE="$EXECUTEDIR"/logs/nginx.pid                             
                                                                  
check_pid() {                                                     
    RETVAL=1                                                      
    if [ -f $PID_FILE ]; then                                     
        PID=`cat $PID_FILE`                                       
        ls /proc/$PID &> /dev/null                                
        if [ $? -eq 0 ]; then                                     
            RETVAL=0                                              
        fi                                                        
    fi                                                            
}                                                                 
                                                                  
check_running() {                                                 
    PID=0                                                         
    RETVAL=0                                                      
    check_pid                                                     
    if [ $RETVAL -eq 0 ]; then                                    
        echo "$CMD is running as $PID, we'll do nothing"          
        exit                                                      
    fi                                                            
}                                                                 
                                                                  

                                                                                   
start() {                                                                          
    check_running                                                                  
    if [ $RETVAL -eq 1 ];then                                                      
       "$CMD" -c "$EXECUTEDIR/conf/nginx.conf" -p "$EXECUTEDIR/"                   
        PID=$(cat $PID_FILE)                                                       
        echo "nginx (pid $PID) is running..."                                      
        if [ $? -eq 0 ];then                                                       
           RETVAL=0                                                                
        fi                                                                         
   fi                                                                              
}                                                                                  
                                                                                   
stop() {                                                                           
     check_pid                                                                     
     if [ $RETVAL -eq 0 ];then                                                     
        "$CMD" -c "$EXECUTEDIR/conf/nginx.conf" -p "$EXECUTEDIR/" -s stop          
         if [ $? -eq 0 ];then                                                      
         echo "nginx shutting down is done..."                                     
         fi                                                                        
      else                                                                         
         echo "nginx is not running"                                               
     fi                                                                            
}                                                                                  
                                                                                   
status() {                                                                         
    check_pid                                                                      
    if [ $RETVAL -eq 0 ]; then                                                     
        echo "nginx (pid $PID) is running ..."                                     
    else                                                                           
        echo "nginx is not running"                                                
    fi                                                                             
}                                                                                  
                                                                                   
reload() {                                                                         
    "$CMD" -c "$EXECUTEDIR/conf/nginx.conf" -p "$EXECUTEDIR/" -t                   
    if [ $? -ne 0 ]; then                                                          
        echo "test nginx conf fail. please check it first, we won't reload it"     
        exit 1                                                                     
    fi                                                                             
    "$CMD" -c "$EXECUTEDIR/conf/nginx.conf" -p "$EXECUTEDIR/" -s reload            
}                                                                                  
                                                                                   
reopen() {                                                                         
    "$CMD" -c "$EXECUTEDIR/conf/nginx.conf" -p "$EXECUTEDIR/" -s reopen            
}   

RETVAL=0
case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    restart)
        stop
        sleep 1
        start
        ;;
    status)
        status
        ;;
    reload)
        reload
        ;;
    reopen)
        reopen
        ;;
    *)
    echo "Usage: $0 {start|stop|restart|status|reload|reopen}"
    RETVAL=1
esac
exit $RETVAL


```


> 编辑 `nginx.conf` 文件

```smartyconfig

                                                                     
#user  nobody;                                                       
worker_processes  1;                                                 
# ...
events {
    worker_connections  1024;
}

#...

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    #...
    include "sites-enabled/*.conf";  # 转移到下一个子配置文件
    }
```

> 子配置文件 

```apacheconfig

log_format projectLog '$remote_addr $http_x_forwarded_for - $remote_user  [$time_local]  '
                                   ' "$request"  $status  $body_bytes_sent  '
                                   ' "$http_referer"  "$http_user_agent"  "$request_time" "$upstream_response_time"';


upstream python_project {
    server 10.0.0.206:8089;
}


server {
listen 8087;
server_name  10.0.0.206;
access_log  /home/aaa/nginx/logs/project.log;

location /api/A {
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_pass http://project;
}
location /{
root /home/aaa/project/dist;
try_files $uri /index.html =404;
}

}
 
 ```
 
 
 # 启动项目 
    
    > 先用uwsgi 启动服务,
    
    > 启动nginx代理请求，处理静态文件
 
 
 
 
 













