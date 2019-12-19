---
title: apscheduler使用
description: apscheduler使用
categories:
- python
tags:
- python基础
---

<br>


# apscheduler使用：

<br>

### 四个模块

- `触发器(trigger)` 

        包含调度逻辑，每一个作业有它自己的触发器，用于决定接下来哪一个作业会运行。
        除了他们自己初始配置意外，触发器完全是无状态的。

- `作业存储(job store)`

       存储被调度的作业，默认的作业存储是简单地把作业保存在内存中，其他的作业存储是将作业保存在数据库中。
       一个作业的数据讲在保存在持久化作业存储时被序列化，并在加载时被反序列化。调度器不能分享同一个作业存储。

- `执行器(executor)`

        处理作业的运行，他们通常通过在作业中提交制定的可调用对象到一个线程或者进城池来进行。
        当作业完成时，执行器将会通知调度器。

- `调度器(scheduler)`

        通常在应用只有一个调度器，应用的开发者通常不会直接处理作业存储、调度器和触发器，相反，调度器提供了处理这些的合适的接口。配置作业存储和执行器可以在调度器中完成，例如添加、修改和移除作业。





<br>

示例代码

<br>

```python


from apscheduler.schedulers.blocking import BlockingScheduler
import datetime

def test(param):
    print(datetime.datetime.now().strftime("%y-%m-%d %H:%M:%S"), "schedule{0} start".format(param))


scheduler = BlockingScheduler()

# 定时任务
scheduler.add_job(func=test, args=("cron",), trigger="cron", second="*/5" ) 
    
# 一次性任务  不需要trigger 在next_run_time 后只执行一次
scheduler.add_job(func=test, args=("date",), next_run_time=datetime.datetime.now() + datetime.timedelta(seconds=2) )

#循环执行
scheduler.add_job(func=test, args=("interval",),  trigger='interval', seconds=3)


#启动任务
scheduler.start()


```


> Triggers 触发器

            
        date表示具体的一次性任务   trigger不需要写
        
        interval表示循环任务
        
        cron表示定时任务
        
           cron 的参数：
            
            #表示2017年3月22日17时19分07秒执行该程序
            sched.add_job(my_job, 'cron', year=2017,month = 03,day = 22,hour = 17,minute = 19,second = 07)
             
            #表示任务在6,7,8,11,12月份的第三个星期五的00:00,01:00,02:00,03:00 执行该程序
            sched.add_job(my_job, 'cron', month='6-8,11-12', day='3rd fri', hour='0-3')
             
            #表示从星期一到星期五5:30（AM）直到2014-05-30 00:00:00
            sched.add_job(my_job(), 'cron', day_of_week='mon-fri', hour=5, minute=30,end_date='2014-05-30')
             
            #表示每5秒执行该程序一次，相当于interval 间隔调度中seconds = 5
            sched.add_job(my_job, 'cron',second = '*/5')

> Schedulers
    
    BlockingScheduler : 调度器在当前进程的主线程中运行，也就是会阻塞当前线程
    
    BackgroundScheduler : 调度器在后台线程中运行，不会阻塞当前线程。
    
    AsyncIOScheduler : 结合 asyncio 模块（一个异步框架）一起使用。
    
    GeventScheduler : 程序中使用 gevent（高性能的Python并发框架）作为IO模型，和 GeventExecutor 配合使用。
    
    TornadoScheduler : 程序中使用 Tornado（一个web框架）的IO模型，用 ioloop.add_timeout 完成定时唤醒。
    
    TwistedScheduler : 配合 TwistedExecutor，用 reactor.callLater 完成定时唤醒。
    
    QtScheduler : 你的应用是一个 Qt 应用，需使用QTimer完成定时唤醒。



> Job stores

`添加 job`

    有两种添加方法
     
        使用add_job(func=func_name)
        
        @scheduler.scheduled_job(func_name, 'interval', minutes=2)

`移除 job`

    scheduler.add_job(job_func, 'interval', minutes=2, id='job_one')
    
    # 移除的job_id
    scheduler.remove_job(job_one)
    
    job = add_job(job_func, 'interval', minutes=2, id='job_one')
    job.remvoe()


`获取 job 列表`

    scheduler.get_jobs() 方法能够获取当前调度器中的所有 job 的列表
    
 
 `修改 job`   
     
     scheduler.add_job(job_func, 'interval', minutes=2, id='job_one')
     scheduler.start()
     
     
     # 将触发时间间隔修改成 5分钟
     scheduler.modify_job('job_one', minutes=5)
     
     job = scheduler.add_job(job_func, 'interval', minutes=2)
     # 将触发时间间隔修改成 5分钟
     job.modify(minutes=5)
     
     # 注意 job的ID不能修改
   


`关闭 job` 
    
    默认情况下调度器会等待所有正在运行的作业完成后，关闭所有的调度器和作业存储。如果你不想等待，可以将 wait 选项设置为 False。
    
    scheduler.shutdown()
    
    scheduler.shutdown(wait=false)
 

    
> Executors

最常用的 executor 有两种：ProcessPoolExecutor 和 ThreadPoolExecutor
