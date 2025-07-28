---
title: GIL指北
date: 2025-05-03 16:04:38
tags: 
    - python
---

早就为python的GIL机制所扰，正好乘着五一详细了解下。

先看一下这篇文章：

## [python-wiki GlobalInterpreterLock](https://wiki.python.org/moin/GlobalInterpreterLock)

**定义**： GIL是cpython解释器中的锁机制，用于多线程执行时互斥访问Python字节码。

**作用**：GIL防止了竞态条件，确保线程安全。~~别的语言咋就没这问题，就你矫情。。。~~

GIL即便不是瓶颈，也会降低性能 [Inside the Python GIL](https://www.dabeaz.com/python/GIL.pdf) [youtube](https://www.youtube.com/watch?v=Obt-vMVdM8s)。这是由于GIL和线程切换带来的代价。

### GIL切换规则

1. IO阻塞时释放

![](/images/python/GIL-io.jpg)

2. 每100个解释器的tick

该值可以通过sys.setcheckinterval()设置，tick的含义是解释器指令，可以通过

```python
import dis
dis.dis(func)
```

来获取，也就是tick本身不是基于时间的，这也就导致有的tick执行时间长有的执行时间短。例如下面一段函数，第二行代码的复杂度是o(n)，但是无论n的大小，这行代码始终是一个tick

```python
def func(n):
    nums = list(range(n))
    -1 in nums
```

### 为什么信号无法终止python的线程

~~真是强者处处是惊喜，平常重开个terminal给kill -9，一直没关注过这个问题~~

+ 在信号到达之后，解释器会在每个tick之后运行check(线程切换)函数，直到主线程运行

+ 由于只有主线程上有信号处理器，解释器会在每个tick后快速申请释放GIL直到主线程被调度。这是因为Python自己不调度线程，所以只能寄希望于操作系统早点去调度主线程

**信号没用的通常原因：主线程阻塞在了线程join或者lock阶段，因此等不到被调度了。不仅如此，解释器还此时会变得很臃肿，因为在每个tick后增加了额外的线程切换操作**

![](/images/python/GIL-tick.jpg)

接下来看看这篇文章，解释了cpython有怎样的线程安全问题，又是如何通过GIL解决的

### GIL实现

1. GIL不是简单的互斥锁

2. POSIX或者pthread环境变量

3. 基于信号实现锁申请与释放

### 线程调度

1. 接收到信号和开始执行之间的时间间隔由操作系统决定

2. 线程调度的优先级：CPU密集型任务：低优先级；IO密集任务：高优先级。如果一个信号被发送给了低优先级的线程，且CPU正在运行高优先级的线程，那么并不会让出CPU


### 线程切换

每100个tick之后，解释器都会执行以下动作：

1. mutex加锁

2. 在阻塞线程的信号量上发送信号

3. 由于有线程正在等待，需要进行额外的pthread处理和系统调用来传递信号

因此，两个CPU密集型的任务串行会比线程运行更加节约时间

### 多核运行可能更加慢

c1和c2分别运行t1和t2，t1在RELEASE时通知t2。但是由于c1只在运行t1，所以又会去重新申请GIL，如果申请到了，t2获取GIL就会失败，重新进入阻塞态

这是python设计的问题？

1. 对于python来说，只想要单线程运行，不想调度线程

2. 对于操作系统来说，尽可能分配多个核来让程序跑的更快一些

即便只有一个CPU密集型任务也会降低效率，这是因为IO操作完成时，GIL可能被CPU密集型任务占用，只能等待释放，并且还有可能抢占速度不如CPU密集型线程。该类现象频繁出现可能会导致优先级逆转的问题：低优先级（CPU密集）的任务阻塞了高优先级（IO密集）的任务。原因是IO密集型线程醒的速度不够快

## 非cpython实现

+ Jython和ironPython没有GIL

+ PyPy有GIL

## 移除GIL的难点

1. 简洁：提案必须可实现且能够长远持续

2. 并发：能够提升多线程程序的效率

3. 速度：作者说他会拒绝所有降低单线程运行效率的提案。要达到这点非常困难，因为当前的引用计数机制在非并行情况下运行非常迅速，但同时意味着一个对象的所有引用都会对引用计数进行修改；而大多数垃圾回收算法都假定了这种修改很少见。

4. API适配：新的提案必须要支持当前cpython的__del__方法和弱引用，__del__本身也不是线程安全的；如果用锁来确保__del__的线程安全，则会可能导致意外的死锁。正确的方式是用额外的线程管理GC，但是这从python应用的视角来说过于复杂了。

5. 立即摧毁：引用计数归0是立即回收对象

6. 顺序摧毁： 除非循环引用， python现在还会先回收无法到达的对象X，再回收其余被X引用的对象，也就是说运行__del__之后对象的属性依然存在，许多垃圾回收机制做不到这一点。

## [The Python GIL (Global Interpreter Lock)](https://python.land/python-concurrency/the-python-gil#google_vignette)

上来第一句话就是Python受到GIL影响，异步编程较困难。没绷住。。。

### 线程安全问题

为什么需要额外的机制来确保线程安全呢，这是因为cpython的内存管理机制决定的

文章里举了两个例子

e.g. 1

```python
a = 2

# thread 1
a = a + 2

# thread 2
a = a * 3
```


e.g. 2

```python
# thread 1
a = 2
a = a + 2

# thread 2
a = 2
a = a * 3
```

如果线程1和线程2有着不同的推进推进顺序，则会造成不同的输出。所以在这里引入了GIL？？？读到这里还是很疑惑：

1. GIL难道就是为了让不懂race condition的开发者使用python？(原文： It’s exactly why Python has a GIL — to make life easier for the majority of Python users.)

2. 这俩线程在GIL释放、线程切换时不还是会有同样的问题吗。。。

先继续读

### 是否能够删掉GIL机制

能否删除或者关掉GIL机制？这不是很容易，python发展到现在，一些特征、库、包的功能都依赖于GIL，如果删掉这套环境将会被打破。

~~搞半天不删掉原来是历史遗留问题，真是一言难尽。。。不过为啥不在新的版本里删掉，然后强制要求社区重新开发。。。~~

### 真有神人从cpython的源码里删除了GIL机制！

[cpython without GIL](https://github.com/colesbury/nogil)


### 你咋还卖课呢。。。


# 实现原理

看看Cpython实现，

``` c++
// Python/ceval_gil.c
/*
   Notes about the implementation:

   - The GIL is just a boolean variable (locked) whose access is protected
     by a mutex (gil_mutex), and whose changes are signalled by a condition
     variable (gil_cond). gil_mutex is taken for short periods of time,
     and therefore mostly uncontended.
*/
```
这段注释说Gil就是一个布尔值的变量，这个变量被一个互斥锁保护gil_mutex,通过condition variable通知其余线程。