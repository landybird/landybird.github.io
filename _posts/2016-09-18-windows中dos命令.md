---
title: windows中dos命令
description: linux中用到的知识点汇总
categories:
- windows
tags:
- windows
---

<br>


# windows中dos命令：

<br>

#### 1windows 设置环境变量：
	

```mysql
    set  Path = ''xxx " 
    和 linux 中 export  path = ‘’xxx”  一样
```




	
#### 2 windows 切换管理员身份：
	
```mysql
1  先将管理员启用
    右击计算机  -->>  管理  -->>  本地用户和组 -->> 用户 -->> administrator -->>  账户启用


2  切换以 管理员身份 运行 命令行
          windows   服务的启动需要 以管理员的身份 
          net   start  
    
       开始 -->>  附件  -->> 命令操作符  --->> shift  +  右键  以管理员身份 打开  -->>  输入  net  user  administrator /active   : yes   -->>  注销或者是重启系统就可以 以管理员的身份运行
```
		
		
#### 3 查看开启的服务  ：


```mysql
    win+R -->>  services.msc
```
	
#### 4 查看端口：
	
```mysql
查看端口占用对应的PID进程号
    netstat     -ano|findstr      "6060"     -->>    TCP    0.0.0.0:6060           0.0.0.0:0              LISTENING       11708
```



#### 5 查看对应的PID对应的进程：

```mysql
查看对应的PID对应的进程

    tasklist|findstr      "11708"     -->>           godoc.exe                    11708 Console                    1      8,488 K
```


#### 6 杀死进程：

```mysql
taskkill /pid  11708  /f
```


#### ....：











