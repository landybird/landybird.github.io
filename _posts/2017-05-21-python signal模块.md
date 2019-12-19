---
title: python signal模块
description: python signal模块
categories:
 - python
tags:
- python基础
---


# python--signal模块

<br>

##  singnal.signal(signalnum, handler)

- 当handler为signal.SIG_IGN时，信号被无视(ignore)。
- 当handler为singal.SIG_DFL，进程采取默认操作(default)。
- 当handler为一个函数名时，进程采取函数中定义的操作。


```python

    
    def init_signal_handler(self):
        signals = (signal.SIGTERM, signal.SIGINT)
        self.signal_handlers = {}
        for sig in signals:
            self.signal_handlers[sig] = signal.getsignal(sig)
            signal.signal(sig, self.handle_signal)
    
    def handle_signal(self, signal, frame):
        self.logger.info('Handle signal %d, stop service', signal)
        self.logger.info('Try to stop all workers.')
        self.stop()
        self.logger.info('Bye-bye.')
        sys.exit(0)

```