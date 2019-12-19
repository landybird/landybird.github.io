---
title: python性能测试框架locust
description: python性能测试框架locust
categories:
- python
tags:
- python基础
---

<br>


# python性能测试框架locust：

<br>


## 1 性能测试的内容：
   
     > 压力产生器，也就是可以指定产生多大的压力，多少并发；
   
     > 数据统计，也就是结果的展示，要统计TPS是多少，响应时间多少等等，这些数据； 

     > 代理功能，代理功能呢说白了就一句话，分摊压力 
   
   
> 系统吞吐量的指标：

    QPS（TPS）：每秒钟request/事务 数量
    并发数： 系统同时处理的request/事务数
    响应时间： 一般取平均响应时间
       
    

<br>

## 2 下载安装：

Locust是python的一个第三方模块，安装很简单，直接`pip install locust`

[官网](www.locust.io)


<br>
## 3 测试demo：


```python

    from locust import HttpLocust, TaskSet, task
    
        #  HttpLocust 这个类的作用是用来发送http请求的
        #  TaskSet   这个类是定义用户行为的，相当于loadrunnerhttp协议的脚本，jmeter里面的http请求一样，要去干嘛的
        # task   这个task是一个装饰器，它用来把一个函数，装饰成一个任务，也可以指定他们的先后执行顺序
    
    class UserBehavior(TaskSet):
        def on_start(self):
            """ on_start is called when a Locust start before any task is scheduled """
            self.login()
    
        def on_stop(self):
            """ on_stop is called when the TaskSet is stopping """
            self.logout()
    
        def login(self):
            self.client.post("/login", {"username":"ellen_key", "password":"education"}) # 先验证登录
    
        def logout(self):
            self.client.post("/logout", {"username":"ellen_key", "password":"education"}) 
    
        @task(2)
        def index(self):
            self.client.get("/")
    
        @task(1)
        def profile(self):
            self.client.get("/profile")
    
    class WebsiteUser(HttpLocust):
        # 定义用户的请求
        task_set = UserBehavior # 指定用户任务
        min_wait = 5000
        max_wait = 9000
        
        # 超时时间范围
        
        # wait_function = lambda self: random.expovariate(1)*1000
        # 写成固定的超时时间


```

<br>

## 4 启动locust

    无界面  locust -f locust_test.py  --host=http://10.0.0.206:8086/ --csv=locustTest --no-web -c 20 -r 5 --run-time 1m
    
    有界面  locust -f locust_test.py  --host=http://10.0.0.206:8087/  --web-host=127.0.0.1 --web-port=8088
    
    
        -f 指定py文件
        --host  指定请求的域名
        --csv  指定生成 csv文件
        --no-web 指定不使用 UI 界面执行 -c client数量  -r 每个client请求的速度
    
    详细参数介绍参数
    
        locust --help
            
            Options:
              -h, --help            show this help message and exit
              -H HOST, --host=HOST  Host to load test in the following format:
                                    http://10.21.32.33
              --web-host=WEB_HOST   Host to bind the web interface to. Defaults to '' (all
                                    interfaces)
              -P PORT, --port=PORT, --web-port=PORT
                                    Port on which to run web host
              -f LOCUSTFILE, --locustfile=LOCUSTFILE
                                    Python module file to import, e.g. '../other.py'.
                                    Default: locustfile
              --csv=CSVFILEBASE, --csv-base-name=CSVFILEBASE
                                    Store current request stats to files in CSV format.
              --master              Set locust to run in distributed mode with this
                                    process as master
              --slave               Set locust to run in distributed mode with this
                                    process as slave
              --master-host=MASTER_HOST
                                    Host or IP address of locust master for distributed
                                    load testing. Only used when running with --slave.
                                    Defaults to 127.0.0.1.
              --master-port=MASTER_PORT
                                    The port to connect to that is used by the locust
                                    master for distributed load testing. Only used when
                                    running with --slave. Defaults to 5557. Note that
                                    slaves will also connect to the master node on this
                                    port + 1.
              --master-bind-host=MASTER_BIND_HOST
                                    Interfaces (hostname, ip) that locust master should
                                    bind to. Only used when running with --master.
                                    Defaults to * (all available interfaces).
              --master-bind-port=MASTER_BIND_PORT
                                    Port that locust master should bind to. Only used when
                                    running with --master. Defaults to 5557. Note that
                                    Locust will also use this port + 1, so by default the
                                    master node will bind to 5557 and 5558.
              --expect-slaves=EXPECT_SLAVES
                                    How many slaves master should expect to connect before
                                    starting the test (only when --no-web used).
              --no-web              Disable the web interface, and instead start running
                                    the test immediately. Requires -c and -r to be
                                    specified.
              -c NUM_CLIENTS, --clients=NUM_CLIENTS
                                    Number of concurrent Locust users. Only used together
                                    with --no-web
              -r HATCH_RATE, --hatch-rate=HATCH_RATE
                                    The rate per second in which clients are spawned. Only
                                    used together with --no-web
              -t RUN_TIME, --run-time=RUN_TIME
                                    Stop after the specified amount of time, e.g. (300s,
                                    20m, 3h, 1h30m, etc.). Only used together with --no-
                                    web
              -L LOGLEVEL, --loglevel=LOGLEVEL
                                    Choose between DEBUG/INFO/WARNING/ERROR/CRITICAL.
                                    Default is INFO.
              --logfile=LOGFILE     Path to log file. If not set, log will go to
                                    stdout/stderr
              --print-stats         Print stats in the console
              --only-summary        Only print the summary stats
              --no-reset-stats      Do not reset statistics once hatching has been
                                    completed
              -l, --list            Show list of possible locust classes and exit
              --show-task-ratio     print table of the locust classes' task execution
                                    ratio
              --show-task-ratio-json
                                    print json data of the locust classes' task execution
                                    ratio
              -V, --version         show program's version number and exit
            

 <br>
 
 
 
 ## 5.    




