---
title: 协程库解读
date: 2025-04-17 20:44:09
tags:
---

# [greenlet](https://greenlet.readthedocs.io/en/stable/)

一个由C编写的库，对外提供pyd文件

greenlet是轻量级的协程，可以直接使用，但是作者还是更推荐用gevent这类提供了高级抽象和异步io方法的库

## greenlet和线程的区别

从理论上而言，多线程抢占式并行执行，greenlet作为实现协程的库，按照序列协作执行。统一时间只能有一个协程运行。 **所以协程内需要谨慎使用锁，以及带锁的数据结构例如queue，以免发生死锁。**

线程创建需要申请系统资源，协程不需要，因此可以创建更多。

## greenlet的功能

初衷：在一个同步的循环里执行异步GUI（同时读取输入和响应事件）
实现方法：在等待下次输入的时候可以将事件循环切换给别的任务

```python
from greenlet import greenlet
import time

def task1():
    start_time = time.time()
    print("Task 1 started")
    time.sleep(1)  # Simulate work
    print("Task 1 yielding")
    gr2.switch()  # Yield control to task2
    print("Task 1 resumed")
    time.sleep(1)  # Simulate more work
    end_time = time.time()
    print(f"Task 1 completed in {end_time - start_time:.2f} seconds")

def task2():
    start_time = time.time()
    print("Task 2 started")
    time.sleep(1)  # Simulate work
    print("Task 2 yielding")
    gr1.switch()  # Yield control to task1
    print("Task 2 resumed")
    time.sleep(1)  # Simulate more work
    end_time = time.time()
    print(f"Task 2 completed in {end_time - start_time:.2f} seconds")

# Create greenlets
gr1 = greenlet(task1)
gr2 = greenlet(task2)

# Start task1 and switch to task2
start_time = time.time()
gr1.switch()
gr2.switch()
end_time = time.time()

print(f"Total execution time: {end_time - start_time:.2f} seconds")
```

# gevent

## 组成部分

1. 基于libev或libuv实现的事件循环

2. 基于greenlet实现的轻量级执行单元

3. API重用了Python标准库的概念，例如events和queues

4. SSL support

5. Cooperative DNS queries

    ```python
    import gevent
    from gevent import socket
    urls = ['www.google.com', 'www.example.com', 'www.python.org']
    jobs = [gevent.spawn(socket.gethostbyname, url) for url in urls]
    _ = gevent.joinall(jobs, timeout=2)
    [job.value for job in jobs]
    ```

6. monkey patch 猴子补丁，将部分标准库替换成协程版本（例如socket、threading）

    ```python
    from gevent import monkey
    monkey.patch_all()  # 在代码开头打补丁
    ```

7. TCP/UDP/HTTP servers

    ```python
    import gevent
    from gevent.pywsgi import WSGIServer

    def app(environ, start_response):
        start_response('200 OK', [('Content-Type', 'text/html')])
        yield b'Hello World'

    server = WSGIServer(('0.0.0.0', 8080), app)
    server.serve_forever()
    ```

# asyncio

## 事件循环

运行机制： 

1. 维护任务队列，具体的调度顺序如下
    ```python
    # create_task的调度方式，将任务放到事件循环末尾
    # await的调度方式： 挂起当前协程，立即执行关键字后的协程任务；当新的协程任务执行完毕后立即唤醒挂起的当前协程

    import asyncio
    import time

    async def coro_a():
        print("A 开始")
        await asyncio.sleep(1)  # 挂起 coro_a，事件循环执行其他任务
        print("A 完成")


    async def coro_b():
        print("B 开始")
        time.sleep(2)  # 模拟耗时长的CPU密集任务
        print("B 完成")


    async def main():
        a = asyncio.create_task(coro_a())
        b = asyncio.create_task(coro_b())
        await asyncio.wait([a, b])


    asyncio.run(main())

    # A 开始
    # B 开始
    # B 完成
    # A 完成
    # 最终执行结果如上所示，
    # 1. 在main函数里创建了两个协程任务a和b最终一起等待，此时任务队列包括a，b；
    # 2. a执行到await时立即创建一个asyncio.sleep的任务并执行。
    # 3. sleep任务的功能是休眠指定的时间，期间让出任务调度
    # 4. b获取到执行，期间不涉及协程切换
    # 5. b执行完毕，且a的定时器到时，唤醒协程a直至执行完毕
    ```



2. 轮询检查就绪的IO事件（Selector、Linux 或 Proactor、Windows）

windows的io复用机制基于iocp，因此asyncio有两种时间循环：SelectorEventLoop和ProactorEventLoop


# uvloop
