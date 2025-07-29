---
title: python 异步编程
date: 2025-06-17 20:18:01
tags:
---

~~python线程进程协程用了好一阵，还没见过有文章能详细涵盖几种异步方式的使用场景和实现原理。自己尝试写一写，能略道一二就当做公开的blog，不然就当是note。~~

# 一、同步与异步

既然标题是异步编程，首先就得说明同步和异步。同步：调用者发起任务后，等待任务响应，然后继续执行；异步：调用者发起任务后可以执行下一条指令，无需等待任务执行完毕。挺好理解的，但是再看看阻塞与非阻塞的定义，阻塞：调用者发起一个任务，如果结果不能立即获得，则挂起等待；非阻塞：发起任务后无论是否能够获得结果，都立即返回。仅从概念来看，同步异步和阻塞非阻塞好像都一样。甚至在特定的上下文中，阻塞等同于同步，但是两者还是有所差别。

以下参考[asynchronous and non-blocking calls? also between blocking and synchronous](https://stackoverflow.com/questions/2625493/asynchronous-and-non-blocking-calls-also-between-blocking-and-synchronous)

同步异步：描述调用者的任务执行方式（是否等待调用结果）
阻塞非阻塞：描述调用者的等待状态（挂起/自旋、执行其他任务）

可以得到以下几个结论：

1. 阻塞：调用者已经挂起，也就是一定在等待任务的执行结果（同步）

2. 非阻塞：调用者可能在自旋等待该任务（同步），也可能去调用其他任务（异步）

3. 异步：调用者并没有在等待该任务的执行结果，更不用说挂起等待，所以一定是非阻塞的 

借用stackoverflow上的一个例子:

```plain text
任务X：“我”
任务Y：“书店”
X：我要一本C++ Primer的书

1. 阻塞：收到Y回答X之前被冻住不能动了

2. 非阻塞：收到Y回答之前X会离开并执行其他操作，可能是每两分钟回来看看有没有回复，也可能是Y给X打电话再回来。

3. 同步：收到Y回答之前，X一直等待。

4. 异步：收到Y回答钱，X会离开书店，直到Y给X打电话才会回来。
```

# 二、异步编程

上述介绍了异步的概念，现在来说明为什么需要异步编程

1. 提升系统资源利用率

2. 并发的多个任务需要通过异步实现

## 2.1 异步编程中可能存在的问题


### 2.1.1 竞争条件 (race condition)

race condition： 多个任务访问共享资源时，由于任务调度算法会在任何时候切换任务。共享资源的状态就会与调度顺序有关。

举个简单常见的例子：

```c
x = 1
x = x * 2
```
当单任务执行上述代码时，x的最终结果就会为2

但是如果有两个线程共享x变量，并发执行上述代码，x的最终结果就可能为2或4

```plaint text
线程A : x = 1
线程B ：x = 1
线程A ：x = x * 2
线程B : x = x * 2
```
最终结果x = 4

```plaint text
线程A : x = 1
线程A ：x = x * 2  
线程B ：x = 1
线程B : x = x * 2
```
最终结果x = 2

那么如何规避race condition，一般的方式是加锁，来看看几种锁的原理：

1. POSIX信号量

POSIX信号量是一种计数器，包括PV两种操作：
P：信号量值大于0，则将其减1，否则阻塞直到大于0
V：信号量值大于1

写个经典的生产者消费者试试

```c++
#include <semaphore.h>
#include <pthread.h>
#include <iostream>

#define BUFFER_SIZE 5

sem_t empty;  // 空
sem_t full; // 满

void* productor(void* args) {
    for (int i = 0; i<10; i++) {
        // 申请一个空，释放一个满
        sem_wait(&empty);
        std::cout << "生产" << i << "\n";
        sem_post(&full);
    }
}

void* consumer(void* args) {
    for (int i = 0; i<10; i++) {
        //申请满，释放空
        sem_wait(&full);
        std::cout << "消费" << i << "\n";
        sem_post(&empty);
    }
}

int main() {
    pthread_t prod, cons;
    sem_init(&empty, 0, BUFFER_SIZE);
    sem_init(&full, 0, 0);
    pthread_create(&prod, NULL, productor, NULL);
    pthread_create(&cons, NULL, consumer, NULL);
    
    pthread_join(prod, NULL);
    pthread_join(cons, NULL);
    
    sem_destroy(&empty);
    sem_destroy(&full);
}
```


2. POSIX互斥锁
pthread_mutex_lock，是一个二值锁，只允许一个线程获取

对于上段代码，创建一个buffer为资源，要求资源互斥访问（不允许生产者和消费者同时访问）

```c++
#include <semaphore.h>
#include <pthread.h>
#include <iostream>

#define BUFFER_SIZE 5
pthread_mutex_t mutex;

sem_t empty;  // 空
sem_t full; // 满

int buffer[BUFFER_SIZE];
int count = 0;

void* productor(void* args) {
    for (int i = 0; i<10; i++) {
        // 申请一个空，释放一个满
        sem_wait(&empty);
        pthread_mutex_lock(&mutex);
        buffer[count++] = i;
        std::cout << "生产" << count << "to" << i << "\n";
        pthread_mutex_unlock(&mutex);
        sem_post(&full);
    }
}

void* consumer(void* args) {
    for (int i = 0; i<10; i++) {
        //申请满，释放空
        sem_wait(&full);
        pthread_mutex_lock(&mutex);
        int item = buffer[--count];
        std::cout << "消费" << item << "from" << i << "\n";
        pthread_mutex_unlock(&mutex);
        sem_post(&empty);
    }
}

int main() {
    pthread_t prod, cons;
    sem_init(&empty, 0, BUFFER_SIZE);
    sem_init(&full, 0, 0);
    pthread_create(&prod, NULL, productor, NULL);
    pthread_create(&cons, NULL, consumer, NULL);
    
    pthread_join(prod, NULL);
    pthread_join(cons, NULL);
    
    sem_destroy(&empty);
    sem_destroy(&full);
}
```

3. condition variable 条件变量

条件变量允许线程基于某些条件而挂起或者唤醒，一般和互斥锁配合使用（对条件变量的访问需要用锁保证互斥）提供了三个主要操作：
    a. 等待：线程在条件变量上阻塞，并释放互斥锁
    b. 通知：唤醒在该条件变量上等待的一个线程
    c. 广播：唤醒在该条件变量上等待的所有线程

使用场景和信号量有些相似：[conditional-variable-vs-semaphore](https://stackoverflow.com/questions/3513045/conditional-variable-vs-semaphore)，不过信号量更加适合用在整数总量的共享资源上，例如经典的生产者消费者问题


### 2.1.2 死锁 deadlock

死锁和上面提到的锁没有直接关系，死锁是任务调度不合理

例子：任务A和B执行时需要资源X和Y；A持有X正在等待Y，B持有Y正在等待X

~~ 校招面试喜欢问 ~~

死锁的必要条件：
1. 互斥：资源不可被多个任务访问

2. 占有等待：等待资源Y时不释放资源X

3. 不抢占：任务A不能从任务B抢占资源Y

4. 循环等待：存在一个等待链，每个任务都在等待下一个任务的资源

死锁的避免方法（即破坏上述条件）：

1. 破坏 占有等待：任务需要一次性申请所有需要的资源，要么就一个也别申请。降低了资源的利用率，可以在初期申请运行需要的资源，在运行时释放已完成使用完的资源。

2. 破坏 不抢占：申请新的资源没有得到满足时需要释放所有已经持有的资源。实现起来复杂，而且会延长任务的时间，影响系统吞吐量。

3. 破坏 循环等待：给资源编号，要求只能按照编号顺序申请资源。这种方法非常低效。

死锁的预防方法：银行家算法

检查此次资源分配后，系统是否处于安全状态。安全状态：剩余资源足够所有任务完成。

死锁的检测方法：1、看看资源等待有无形成环路； 2、采取一种类似于银行家算法的策略：查找进程，假设同意该进程的所有请求并等待执行完毕再释放资源，如果能通过这样的方式标记所有进程，则认为不存在死锁，否则所有未标记的进程都是死锁进程

# 三、Python实现异步编程的方式

## 3.1 线程实现异步

~~ 从最简单最常用的实现方式开始（并非简单。。。）~~

程序只是存在计算机硬盘中的一段数据，只有加载导内存并被操作系统调用后才开始有自己的生命周期。程序的一次运行称作进程。

线程运行在进程中，共享进程的数据。


### 3.1.1 线程实现方式

[Python核心编程（第二版）](https://awesome-programming-books.github.io/python/python%E6%A0%B8%E5%BF%83%E7%BC%96%E7%A8%8B.pdf)

#### 3.1.1.1 _thread 模块

Python核心编程（第二版）是以cpython2为例介绍的线程，我这里用Python3介绍，实现方式可能会略有不同。

``` Python
import _thread
 
def func(): 
    print('abcd') 
_thread.start_new_thread(func, ()) # 开启子线程 
_thread.allocate_lock() # 分配锁
```

_thread模块不推荐使用，在Python3还多了个前置下划线。。。点进模块内部还能看到模块的docstring也推荐使用threading模块

```plain text
This module provides primitive operations to write multi-threaded programs.
The 'threading' module provides a more convenient interface.
```

#### 3.1.1.2 threading模块

threading实现线程的两种常见方式

1. 赋值target
```python
import threading


def func():
    print('abcd')


t = threading.Thread(target=func)
t.start()  # 开始线程
# t.run()  # 开始并等待线程执行结束
t.join()  # 等待线程执行结束

```

主线程默认会join所有子线程，所以哪怕不join也会等待所有子线程执行完毕；如果某个线程不想要等待进程执行完毕，可以设置参数daemon=True，将其标识为守护线程。

正好趁机看看线程创建时函数的含义：

| 参数名 | 参数含义 |
|-------|-------|
| group | *group* should be None; reserved for future extension when a ThreadGroup  <br/> class is implemented. 用来实现线程分组的功能，但是功能还没实现 |
| target | 可执行对象，一般是线程的目标函数 |
| name | 线程名，默认Thread-N |
| daemon | 守护线程 |



2. 重写run函数

```
import threading


class FuncThread(threading.Thread):
    def run(self) -> None:
        print('abcd')


ft = FuncThread()
ft.start()

```

Python线程的实现原理

cpython/modules/_threadmodule.c

```c
// 管理线程的handle
typedef struct {
    struct llist_node node;  // linked list node (see _pythread_runtime_state)

    // linked list node (see thread_module_state)
    struct llist_node shutdown_node;

    // The `ident`, `os_handle`, `has_os_handle`, and `state` fields are
    // protected by `mutex`.
    PyThread_ident_t ident;
    PyThread_handle_t os_handle;
    int has_os_handle;

    // Holds a value from the `ThreadHandleState` enum.
    int state;

    PyMutex mutex;

    // Set immediately before `thread_run` returns to indicate that the OS
    // thread is about to exit. This is used to avoid false positives when
    // detecting self-join attempts. See the comment in `ThreadHandle_join()`
    // for a more detailed explanation.
    PyEvent thread_is_exiting;

    // Serializes calls to `join` and `set_done`.
    _PyOnceFlag once;

    Py_ssize_t refcount;
} ThreadHandle;
```

可见cpython通过pthread实现，pthread又是啥？pthread 是 POSIX 标准线程库，主要用于类 Unix 系统的多线程编程。那么cpython如何在windows实现多线程？

```c 
int PyThread_start_joinable_thread(void (*func)(void *), void *arg, PyThread_ident_t* ident, PyThread_handle_t* handle) { HANDLE hThread; 
    unsigned threadID; callobj *obj; if (!initialized)
        PyThread_init_thread(); obj = (callobj*)HeapAlloc(GetProcessHeap(), 0, sizeof(*obj)); if (!obj) return -1; obj->func = func; obj->arg = arg; 
    PyThreadState *tstate = _PyThreadState_GET(); size_t stacksize = tstate ? tstate->interp->threads.stacksize : 0; hThread = 
    (HANDLE)_beginthreadex(0,
                      Py_SAFE_DOWNCAST(stacksize, Py_ssize_t, unsigned int), bootstrap, obj, 0, &threadID); if (hThread == 0) { /* I've seen errno == 
        EAGAIN here, which means "there are
         * too many threads". */ HeapFree(GetProcessHeap(), 0, obj); return -1;
    }
    *ident = threadID;
    // The cast is safe since HANDLE is pointer-sized
    *handle = (PyThread_handle_t) hThread; return 0;
}
```


线程如何规避race condition问题：线程锁,看看线程锁的cpython实现

```c++
/* cpython/Python/thread_pthread.h */
#ifdef USE_SEMAPHORES
PyThread_type_lock
PyThread_allocate_lock(void)
{
    sem_t *lock;
    int status, error = 0;

    if (!initialized)
        PyThread_init_thread();

    lock = (sem_t *)PyMem_RawMalloc(sizeof(sem_t));

    if (lock) {
        status = sem_init(lock,0,1);
        CHECK_STATUS("sem_init");

        if (error) {
            PyMem_RawFree((void *)lock);
            lock = NULL;
        }
    }

    return (PyThread_type_lock)lock;
}

else

PyThread_type_lock
PyThread_allocate_lock(void)
{
    pthread_lock *lock;
    int status, error = 0;

    if (!initialized)
        PyThread_init_thread();

    lock = (pthread_lock *) PyMem_RawCalloc(1, sizeof(pthread_lock));
    if (lock) {
        lock->locked = 0;

        status = pthread_mutex_init(&lock->mut, NULL);
        CHECK_STATUS_PTHREAD("pthread_mutex_init");
        /* Mark the pthread mutex underlying a Python mutex as
           pure happens-before.  We can't simply mark the
           Python-level mutex as a mutex because it can be
           acquired and released in different threads, which
           will cause errors. */
        _Py_ANNOTATE_PURE_HAPPENS_BEFORE_MUTEX(&lock->mut);

        status = _PyThread_cond_init(&lock->lock_released);
        CHECK_STATUS_PTHREAD("pthread_cond_init");

        if (error) {
            PyMem_RawFree((void *)lock);
            lock = 0;
        }
    }

    return (PyThread_type_lock) lock;
}
```
可见在linux平台会优先使用posix信号量来实现lock，如果不支持则使用posix互斥锁实现

再看看windows平台的实现

```c++
#if _PY_USE_CV_LOCKS
/*
 * Lock support. It has to be implemented as semaphores.
 * I [Dag] tried to implement it with mutex but I could find a way to
 * tell whether a thread already own the lock or not.
 */
PyThread_type_lock
PyThread_allocate_lock(void)
{
    PNRMUTEX mutex;

    if (!initialized)
        PyThread_init_thread();

    mutex = AllocNonRecursiveMutex() ;

    PyThread_type_lock aLock = (PyThread_type_lock) mutex;
    assert(aLock);

    return aLock;
}
else

static PNRMUTEX
AllocNonRecursiveMutex(void)
{
    return CreateSemaphore(NULL, 1, 1, NULL);
}

```

可见在windows平台会优先使用condition variable实现线程锁，否则会使用信号量

不同平台线程锁实现的底层原理都不一样，同一份代码真能无痛跨平台运行吗？这里我有几个问题探究一下：

1. 线程锁是公平的吗？(即有没有FIFO一类的调度规则，保证申请锁的任务的推进)
对于POSIX平台，信号量和互斥锁从定义上看来都不是公平的，锁的获取没有一个公平的调度机制；对于Windows平台而言，condition variable和信号量也都不是公平的。

2. 线程没释放锁就终止了，锁会自动释放吗？
无论是信号量、互斥锁还是条件变量，本身并没有机制确保锁的释放；
（这里有个疑问：为什么cpython这些大神不在线程消亡的时候回调锁的释放？找了个AI问了问：1. 宁可因为不合法的推进而死锁，也不能让数据污染；2.谁申请谁释放）

talk is cheap，写一段试试

```python

import threading
import time

mutex = threading.Lock()
def t():
    mutex.acquire()
    print('123')
if __name__ == '__main__':
    t1 = threading.Thread(target=t)
    t1.start()
    time.sleep(2)
    mutex.acquire()
    print('2222')

```
为了保证子线程t1运行，执行了time.sleep。为啥sleep就能保证子线程由于主线程执行？这个等下说GIL的时候会提及。

这样还有个问题，Python锁对象的生命周期结束时会自动释放锁吗？

```python

import threading
import time

mutex = threading.Lock()


def t():
    # 这里global是为了防止将mutex识别成局部变量
    global mutex
    mutex.acquire()
    print('123')
    del mutex


if __name__ == '__main__':
    t1 = threading.Thread(target=t)
    t1.start()
    time.sleep(2)
    mutex.acquire()
    print('22222')

```
运行结果和上一段程序一致，看来Cpython没有确保锁的释放。

提到线程不得不品的一环：GIL(global interpret lock, 全局解释锁)

[GIL指北](./GIL.md)

### 进程实现异步编程

## 进程实现异步

上一节介绍了Python通过线程实现异步编程，同时也说明了GIL为线程异步带来的性能瓶颈。想要突破这一瓶颈，就需要通过多进程来实现异步。

首先看看Python有哪些实现进程的方式

### multiprocess模块

1. 指定target参数

```Python
import multiprocessing


def func():
    print('subprocess')


if __name__ == '__main__':
    p1 = multiprocessing.Process(target=func)
    p1.start()
    p1.join()

```

2. 重写run函数

```Python
import multiprocessing


class FuncProcess(multiprocessing.Process):
    def run(self) -> None:
        print('subprocess')


if __name__ == '__main__':
    p1 = FuncProcess()
    p1.start()
    p1.join()

```

3. multiprocess.Pool进程池

```Python
from multiprocessing import Pool


def func(x):
    print('subprocess')


if __name__ == '__main__':
    with Pool(processes=3) as p:
        p.map(func, [1,2])

```

Pool进程池参数

| 参数名称 | 参数含义 |
|--------|--------|
| processes | 进程池中允许的最大进程数，默认值为 os.cpu_count()，即逻辑处理器数量 |
| initializer | 初始化函数 |
| initargs | 初始化函数参数 |
| maxtasksperchild | 每个子进程能够完成的最大任务数，完成任务后自动销毁并创建新的进程替代 |
| context | 创建进程的上下文 |

进程是预创建还是懒创建：预创建

```Python
@staticmethod
def _repopulate_pool_static(ctx, Process, processes, pool, inqueue,
                            outqueue, initializer, initargs,
                            maxtasksperchild, wrap_exception):
    """Bring the number of pool processes up to the specified number,
    for use after reaping workers which have exited.
    """
    for i in range(processes - len(pool)):
        w = Process(ctx, target=worker,
                    args=(inqueue, outqueue,
                            initializer,
                            initargs, maxtasksperchild,
                            wrap_exception))
        w.name = w.name.replace('Process', 'PoolWorker')
        w.daemon = True
        w.start()
        pool.append(w)
        util.debug('added worker')
```


### ProcessPoolExecutor进程池

```Python
import multiprocessing
from concurrent.futures import ProcessPoolExecutor


def func():
    print('subprocess')


if __name__ == '__main__':
    pool = ProcessPoolExecutor(
        max_workers=3,
        mp_context=multiprocessing.get_context('spawn')
    )

    pool.submit(func)

```

进程池参数：

| 参数名称 | 参数含义 |
|--------|--------|
| max_workers | 进程池中允许的最大进程数，默认值为 os.cpu_count()，即逻辑处理器数量 |
| mp_context | 多进程上下文，控制进程的启动方式 <br/> ** fork ** ：在Unix系统中常用，通过进程复制创建子线程<br/> ** spawn ** ：用全新的Python解释器创建子进程<br/>  ** forkserver ** : 启动一个独立的服务器进程来管理子进程 |
| initializer | 初始化函数，对于每个进程调用 |
| initargs | 初始化函数的参数 |

上面的max_workers是预申请还是懒申请? 懒申请


** 接下来看看Python实现进程的原理 **

#### 1. 对于spawn上下文

** windows平台 **

```Python
# cpython/Lib/multiprocessing/popen_spawn_win32.py

class Popen(object):
    def __init__(self, process_obj):
        # 1. 创建父子进程通信管道
        rhandle, whandle = _winapi.CreatePipe(None, 0)
        wfd = msvcrt.open_osfhandle(whandle, 0)

        # 2. 准备子进程启动命令行
        cmd = spawn.get_command_line(parent_pid=os.getpid(), pipe_handle=rhandle)
        python_exe = spawn.get_executable()

        # 3. 调用Windows API创建进程
        hp, ht, pid, tid = _winapi.CreateProcess(
            python_exe, cmd,
            None, None, False, 0, env, None,
            STARTUPINFO(dwFlags=STARTF_FORCEOFFFEEDBACK))

        # 4. 向子进程发送初始化数据
        with open(wfd, 'wb', closefd=True) as to_child:
            reduction.dump(prep_data, to_child)
            reduction.dump(process_obj, to_child)
```
1. 通过_winapi.CreatePipe创建进程间通信管道
2. 通过_winapi.CreateProcess创建新进程
3. 在新进程中初始化Python解释器并执行目标函数

## Unix实现

```Python
# cpython/Lib/multiprocessing/popen_fork.py
class Popen(object):
    def _launch(self, process_obj):
        code = 1
        parent_r, child_w = os.pipe()
        child_r, parent_w = os.pipe()
        self.pid = os.fork()
        if self.pid == 0:
            try:
                atexit._clear()
                atexit.register(util._exit_function)
                os.close(parent_r)
                os.close(parent_w)
                code = process_obj._bootstrap(parent_sentinel=child_r)
            finally:
                atexit._run_exitfuncs()
                os._exit(code)
        else:
            os.close(child_w)
            os.close(child_r)
            self.finalizer = util.Finalize(self, util.close_fds,
                                           (parent_r, parent_w,))
            self.sentinel = parent_r
```
可见通过系统调用os.fork()创建子进程

两个Popen是如何实现跨平台统一调用的，context

```Python
# cpython/Lib/multiprocessing/context.py
if sys.platform != 'win32':
    _concrete_contexts = {
        'fork': ForkContext(),
        'spawn': SpawnContext(),
        'forkserver': ForkServerContext(),
    }
    
    if sys.platform == 'darwin':
        # bpo-33725: running arbitrary code after fork() is no longer reliable
        # on macOS since macOS 10.14 (Mojave). Use spawn by default instead.
        _default_context = DefaultContext(_concrete_contexts['spawn'])
    else:
        _default_context = DefaultContext(_concrete_contexts['fork'])
else:
    _concrete_contexts = {
        'spawn': SpawnContext(),
    }
    _default_context = DefaultContext(_concrete_contexts['spawn'])

```

### 进程锁

进程锁通过0-1信号量实现，其中

```Python
# Lib/multiprocessing/synchronize.py
class SemLock(object):
    ...
```

不过如果教程锁通过信号量实现，如何传给创建的新进程呢，这就需要了解进程间通信方式

### 进程间通信方式

1. 管道通信

使用示例

```Python
import multiprocessing


def worker(conn):
    conn.send("Hello from the child process!")
    conn.close()


if __name__ == '__main__':
    # 创建管道连接
    parent_conn, child_conn = multiprocessing.Pipe()

    # 创建子进程
    p = multiprocessing.Process(target=worker, args=(parent_conn,))
    p.start()

    # 从父进程接收消息
    print(child_conn.recv())  # 输出: Hello from the child process!

    child_conn.close()
    p.join()
```

实现原理

```Python
# Lib\multiprocessing\connection.py
if _winapi:
    ...
    h1 = _winapi.CreateNamedPipe
    ...
else:
    if duplex:  # 双向
        s1, s2 = socket.socketpair()
    else:
        fd1, fd2 = os.pipe()
```

2. 队列通信

示例

```Python
...
```

基于管道的封装

```Python
class Queue(object):
    def __init__(self):
        ...
        self._reader, self._writer = connection.Pipe(duplex=False)
```

3. 共享内存

示例

```Python
...
```

实现原理
共享堆内存
```Python
# Lib\multiprocessing\sharedctypes.py 支持的类型包括
typecode_to_type = {
    'c': ctypes.c_char,     'u': ctypes.c_wchar,
    'b': ctypes.c_byte,     'B': ctypes.c_ubyte,
    'h': ctypes.c_short,    'H': ctypes.c_ushort,
    'i': ctypes.c_int,      'I': ctypes.c_uint,
    'l': ctypes.c_long,     'L': ctypes.c_ulong,
    'q': ctypes.c_longlong, 'Q': ctypes.c_ulonglong,
    'f': ctypes.c_float,    'd': ctypes.c_double
    }

```
其中RawValue和RawArray无锁，Value和Array带锁


多进程如何进行复杂对象的传输

```Python
import multiprocessing
from multiprocessing import Manager
from multiprocessing.managers import SyncManager
import time


class A:
    def a(self):
        print('aaa--', multiprocessing.current_process().name)


def worker1(shared_obj):
    shared_obj.a()

    print('woker1', multiprocessing.current_process().name)


def worker2(shared_obj):
    shared_obj.a()

    print('woker2', multiprocessing.current_process().name)

if __name__ == '__main__':
    SyncManager.register('A', A)
    # 创建Manager对象
    manager = SyncManager()
    # 创建共享对象
    manager.register('A', A)
    # a = A()
    manager.start()
    ma = manager.A()
    # 创建进程
    p1 = multiprocessing.Process(target=worker1, args=(ma,))
    p2 = multiprocessing.Process(target=worker2, args=(ma,))

    # 启动进程
    p1.start()
    p2.start()

    # 等待进程结束
    p1.join()
    p2.join()

# aaa-- SyncManager-1
# woker1 Process-2
# aaa-- SyncManager-1
# woker2 Process-3
```

本质是额外启动Manager进程，其余进程为每个对象创建对象代理，通过管道通信调用访问对象。