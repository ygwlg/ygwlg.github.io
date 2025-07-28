---
title: Python守护线程
date: 2025-05-19 21:37:08
tags:
---


threading模块支持守护线程：一般是一个等待客户请求的服务器，如果没有请求，他就在那里等着。如果设定一个线程为守护线程，那说明这个线程是不重要的，在进程退出的时候，不用等待这个线程退出。

```python

import threading
import time


def func1():
    while True:
        time.sleep(1)


threading.Thread(target=func1, daemon=True).start()  # 进程终止

threading.Thread(target=func1).start()  # 进程不会终止
```

