---
title: python的GIL                                              
description: Python中的线程与GIL
categories:
- python
tags:
- python   
---

## Python 和 GIL

[Understanding GIL](http://www.dabeaz.com/python/UnderstandingGIL.pdf)

[GIL](http://www.dabeaz.com/python/GIL.pdf)

[NewGIL](http://www.dabeaz.com/python/NewGIL.pdf)


`C Python` 有一个 `全局解释器锁 (GIL)`
	
	它有时候会是线程“争用”的来源

	它对线程施加了各种限制(您不能使用多个 CPU), 限制了线程性能


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil0.png)



<br>

####  什么是线程？

- Python 线程是真正的系统线程


	  POSIX 线程（pthreads）
	
	  Windows 线程

- 完全由主机操作系统管理

	
	所有调度/线程切换

- Python的线程执行解释器过程（用C编写）


<br>

#### 线程创建


`Thread类`执行 `run` 方法

• `Python threads simply execute a "callable"`

• `The run() method of Thread (or a function)`


``` 
import time
import threading

class CountdownThread(threading.Thread):
	def __init__(self,count):
		threading.Thread.__init__(self)
		self.count = count
		
--> def run(self):
		while self.count > 0:
			print "Counting down", self.count
			self.count -= 1
			time.sleep(5)
		return
		
```


	Python创建一个小型数据结构 包含一些 解释器状态	(interpreter state)	

	启动一个新线程（pthread）

	线程调用 PyEval_CallObject

	最后，运行的 C 函数 调用 指定的 Python 可调用对象


<br>

#### 线程特定状态 `PyThreadState`


- 每个线程都有自己特定的解释器数据结构 (`PyThreadState`)
	

	• 当前堆栈框架 	Current stack frame (for Python code)
	• 当前递归深度 	Current recursion depth
	• Thread ID
	• 一些每线程异常信息    Some per-thread exception information
	• 可选的跟踪/分析/调试 hook    Optional tracing/profiling/debugging hooks


- 它是一个小型 C 结构（<100 字节）

``` 
typedef struct _ts {
	struct _ts *next;
	PyInterpreterState *interp;
	struct _frame *frame;
	int recursion_depth;
	int tracing;
	int use_tracing;
	Py_tracefunc c_profilefunc;
	Py_tracefunc c_tracefunc;
	PyObject *c_profileobj;
	PyObject *c_traceobj;
	PyObject *curexc_type;
	PyObject *curexc_value;
	PyObject *curexc_traceback;
	PyObject *exc_type;
	PyObject *exc_value;
	PyObject *exc_traceback;
	PyObject *dict;
	int tick_counter;
	int gilstate_counter;
	PyObject *async_exc;
	long thread_id;
} PyThreadState;

```

<br>

#### 线程执行


- 解释器有一个全局变量，简单地指向当前运行的`线程的 ThreadState 结构`

``` 
/* Python/pystate.c */
...
PyThreadState *_PyThreadState_Current = NULL;
```

- 解释器的操作(隐式的)依赖于这个变量，来确定当前执行工作的线程


<br>

####  GIL `global interpreter lock` 锁

	不是一个简单的 互斥锁 mutex lock

	它是一个二进制信号量，由 pthreads 互斥量和条件变量构成
	
		locked = 0 # Lock status
		mutex = pthreads_mutex() # Lock for the status
		cond = pthreads_cond() # Used for waiting/wakeup

	GIL 是这个锁的一个实例

``` 
release() {
	mutex.acquire()
	locked = 0
	mutex.release()
	cond.signal()
}

acquire() {
	mutex.acquire()
	while (locked) {
	cond.wait(mutex)
	}
	locked = 1
	mutex.release()
}

```

`GIL`确保确保每个线程运行时, 都得到对解释器内部的独占访问

线程在运行时持有 `GIL`, 他们在`阻塞 I/O`时释放它

	
	所有解释器锁定都基于信号
		要获取 GIL，请检查它是否 free。如果不行，去等待信号
		要释放 GIL，释放它并发出信号


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil.png)



处理 `CPU 密集线程`，这些线程从不执行任何 `I/O`，解释器定期执行“检查”

![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil1.png)

	默认情况下，check一次 每 100 个 ticks



`The Check Interval 检查间隔` 是一个全局计数器，它完全独立于线程的调度

![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil2.png)


<br>

`The Periodic Check` 定期检查期间会发生什么？


- 信号处理程序(`signal handlers`) 将执行待执行的 `signals` (仅在主线程中)

- 释放, 获取 `GIL`


``` 
/* Python/ceval.c */
...
if (--_Py_Ticker < 0) {
	 ...
	 _Py_Ticker = _Py_CheckInterval;
	 ...
	 if (things_to_do) {
		 if (Py_MakePendingCalls() < 0) {
		 ...
		 }
	 }
	 if (interpreter_lock) {
	 /* Give another thread a chance */
		 ...
		 PyThread_release_lock(interpreter_lock);
		 /* Other threads may run now */
		 PyThread_acquire_lock(interpreter_lock, 1);
		 ...
	}

```


<br>

`什么是 Interpreter Ticks`


	映射到 解释器

![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil4.png)


`Interpreter ticks` 不是基于时间的



``` 
x = (i for i in range(100000000))


print(-1 in x)

# Ctrl + C 不能打断 ticks
```


<br>


#### `Signals` 


为什么 Python线程编程 不能再被 `keyboard interrupt` 杀死?

	必须使用 kill -9 在单独的窗口中关闭

如果 `signal` 到达，解释器在每次 tick 之后 check, 直到主线程运行

	尝试尽可能快地切换线程， 可能希望 main 线程会运行

由于`信号处理程序 signal handlers`只能在主线程运行，解释器快速 获取/释放 GIL，直到它被调度
所以它只是尝试尽可能快地切换线程， 可能希望 main 线程会运行


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil5.png)




<br>

#### `Thread Scheduling` 线程调度 
	
- Python 没有`线程调度器`


- 没有`线程优先级`的概念， 抢占、循环调度等


	There is no notion of thread priorities,
	preemption, round-robin scheduling, etc.


- 所有线程调度都留给`操作系统`（例如 Linux、Windows 等)

	
	这就是信号变得如此奇怪的部分原因
	解释器无法控制调度所以 它只是尝试尽可能快地切换线程可能希望 main 会运行



<br>


#### `Thread Scheduling` 线程调度

![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil6.png)

- `信号`和`执行`之间的滞后 取决于操作系统


- 操作系统 会先执行最高“优先级”的线程
	

	• CPU-bound : low priority
	• I/O bound : high priority


如果一个信号被发送到一个低优先级的线程，并且CPU忙于更高的优先级任务，它不会运行，直到稍后的时间点



<br>

#### `CPU-Bound Threads` CPU 密集线程

GIL Battle

`两个 CPU-bund 型线程`

![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil7.png)

![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil8.png)


`一个 CPU-bund 一个 I/O -bund`

	会降低 I/O 绑定线程 的响应时间

	因为 I/O 线程不能足够快地醒来以获得 GIL (I/O bund 优先级反转)

	只发生在多核


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil9.png)

![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil10.png)

![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil11.png)






## 新的 GIL 实现 (`python3.2`)

	only available by svn checkout


不是`ticks`，现在有一个全局变量  `gil_drop_request`


``` 
/* Python/ceval.c */
...
static volatile int gil_drop_request = 0;
```

线程一直运行 直到该值设置为 1 , 此时，线程必须丢弃 `GIL`



#### 新 GIL 图解



![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil01.png)


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil02.png)


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil03.png)


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil04.png)


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil04.png)


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil05.png)


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil06.png)


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil07.png)


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil08.png)


![](https://raw.githubusercontent.com/landybird/landybird.github.io/master/assets/images/gil001.png)






