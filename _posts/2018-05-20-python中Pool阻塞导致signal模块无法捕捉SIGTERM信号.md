---
title: python中Pool阻塞导致signal模块无法捕捉SIGTERM信号
description: python中Pool阻塞导致signal模块无法捕捉SIGTERM信号
categories:
- python
tags:
- python基础
---

<br>


# python中Pool阻塞导致signal模块无法捕捉SIGTERM信号


**遇到的问题**

    使用 进程池 multiprocessing.Pool 同时开启多个计算任务 , 但是关闭主进程的时候signal模块无法捕获SIGTERM信号 


注意在 `>=python3.3 `已经没有这个问题了



标准库中signal有如下描述：

A Python `signal handler` does not get executed inside the low-level (C) signal handler.

     Instead, the low-level signal handler sets a flag which tells the virtual machine to execute the corresponding 
     Python signal handler at a later point(for example at the next bytecode instruction). This has consequences:


    python中的信号处理函数不会被低级别的信号处理器触发调用。取而代之的是，低级别的信号处理程序会设置一个标志，
    用来告诉虚拟机在稍后（例如下一个字节代码指令）来执行信号处理函数。


It makes little sense to catch synchronous errors like SIGFPE or SIGSEGV that are caused by an invalid operation in C code.
 Python will return from the signal handler to the C code, which is likely to raise the same signal again, causing Python to apparently hang. 
 From Python 3.3 onwards, you can use the faulthandler module to report on synchronous errors.
 
 
     难以捕获C代码无效操作引起的同步异常，例如SIGFPE、SIGSEGV。Python将从信号处理返回到C代码，
     这很可能会再次提出同样的信号，导致python挂起。

 
 A long-running calculation implemented purely in C (such as regular expression matching on a large body of text) 
may run uninterrupted for an arbitrary amount of time, regardless of any signals received.
 The Python signal handlers will be called when the calculation finishes.


    用C实现的长时计算程序（比如正则表达式匹配大段文本）可能不被中断的运行任意长时间，而不管信号的接收。
    在计算完成时，python的信号处理函数将被执行。


> 原因:

调用pool.join使得`主进程（线程）阻塞在join处`，意味着它`阻塞在C方法pthread_join调用中`。pthread_join并不是一个long-running calculation的程序，而是一个`系统调用的阻塞`，如此，`直到它结束，否则信号处理函数无法被执行`。







> 1 方案一  os.killpg(os.getpid()) 杀死同进程组的进程  > python3.3

    # 使用 python3.3  主进程捕获SIGTERM信号，杀死同一进程组的进程
    
```python
        
      
        from multiprocessing import Pool
        import os
        import signal
        
        from conf import PARAM_CALCULATE
        from utils.tools import get_calculate_function
        from utils import logger
        
        class CalculateTask(object):
        
            def __init__(self):
                self._running = True
                self.process_pool = Pool(5)
        
            def run(self):
                signal.signal(signal.SIGTERM, self.get_terminate)
                for key, value in PARAM_CALCULATE.items():
                    func_name = get_calculate_function(value)
                    self.process_pool.apply_async(func_name, args=(key,))
                    logger.info("start to task [%s]",func_name.__name__)
                self.process_pool.close()
                self.process_pool.join()
        
        
            def get_terminate(self, signum, frame):
                pid = os.getpid()
                os.killpg(os.getpgid(pid), signal.SIGKILL)    //  杀死统一进程组的所有进程
                logger.info("killed 进程以及其子进程")
        
        




```
    
    


> 方案二   使用 event 来控制worker进程的退出



```python
    
    from multiprocessing import Pool, Manager
    import os
    import signal
    import functools
    import time
    
    from conf import PARAM_CALCULATE
    from utils.tools import get_calculate_function
    from utils import logger
    
    class CalculateTask(object):
    
        def __init__(self):
            self._running = True
            self.process_pool = Pool(5)
            self.manager = Manager()
            self.event = self.manager.Event()
            self.handler = functools.partial(self.get_terminate, self.process_pool, self.event, self.manager)
    
        def run(self):
            signal.signal(signal.SIGTERM, self.handler)
            for key, value in PARAM_CALCULATE.items():
                func_name = get_calculate_function(value)
                self.process_pool.apply_async(func_name, args=(key, self.event))
                logger.info("start to task [%s]",func_name.__name__)
    
            while not self.event.is_set():
                time.sleep(60)
    
        def get_terminate(self, pool, event, manager, signum, frame):
            if not event.is_set():
                event.set()
            pool.close()
            pool.join()
            manager.shutdown()
            logger.info("killed 进程以及其子进程")
            os._exit(0)
           
       
       
       //  os._exit(), 直接退出 Python 解释器, 不抛异常, 不执行相关清理工作，其后的代码都不执行
        // sys.exit()函数是通过抛出异常的方式来终止进程的，也就是说如果它抛出来的异常被捕捉到了的话程序就不会退出了，
            



    # 计算任务
    
    def calculate_xxxxx(key, event):
        while not event.is_set():
            id = rediscli.client.rpop(key)
            if id == None:
                continue
            unback = get_unback_amount(key, id)
            update_ensure_unback_amount(id, unback, key)


```