---
title: Python 的生成器与异步io
date: 2025-02-27 20:58:50
tags: 
    - Python
    - async
categories: Python
---

## 生成器与迭代器

带有yield的函数在调用时会返回生成器对象

### yield

```python
def count(total):
    c = 1
    while c <= total:
        yield c
        c += 1
```
上述代码创建了一个带有yield的函数，函数实例化不执行而是返回一个生成器对象。每次调用next或send时向后执行。

```python
cc = count(10)
print(cc)
print(cc.send(None))
print(cc.send('m1'))
print(cc.send('m2'))

-----------
<generator object count at 0x0000020740CC5EE0>
1
m1
2
m2
3

```

或者使用for调用生成器

```python
cc = count(10)
print(cc)
for i in cc:
    print(i)

---------
<generator object count at 0x0000024CDA155EE0>
1
None
2
None
3
None
4
None
5
None
6
None
7
None
8
None
9
None
10
None

```

那么看看常见的for干了什么

得先了解一下迭代器：可以被next函数调用并返回下一个值的对象成为迭代器

```python
print(isinstance(cc, Iterable))

------------
True
```
也就是说生成器对象本身也是迭代器

了解迭代器后就可以大致了解for循环的本质：不断调用next函数直到StopInteration异常

```python
list1 = [1,2,3,4,5]
while True:
    try:
        x = next(list1)
        print(x)
    except StopIteration:
        break
```
这个时候会报错说list不是可迭代类型。。。也就是for会先转化成迭代器

```python
list1 = [1, 2, 3, 4, 5]
    it = iter(list1)
    while True:
        try:
            x = next(it)
            print(x)
        except StopIteration:
            break
```

不过想一想也有道理。如果list本身是一个迭代器对象，那么即使是for循环多轮，也只能遍历一次。代码如下

```python
list1 = [1, 2, 3, 4, 5]
it = iter(list1)
for i in it:
    print(i)

for i in it:
    print(i)

----------
1
2
3
4
5

Process finished with exit code 0
```

这里有一篇博文讲的很清楚: [for循环的底层原理；迭代器(Iterator),可迭代对象(Iterable),生成器( generator)](https://www.cnblogs.com/guanghui-hua/p/18062292)

容器类型（list，dict甚至str等）都是Iterable但不是Iterator，它们可以被iter函数转换成迭代器用于遍历。

Iterable对象表示的是数据流，数据流可以是无限大的。和刚刚提到的容器各有各的用处。

既然了解了iter(Iterbale对象)的原理，再来看看Iterable(Iterator)干了什么，以list()为例

```python
cc = count(3)
print(cc)
print(list(cc))
------------
<generator object count at 0x000001FC65CA5EE0>
None
None
None
[1, 2, 3]
```
从打印结果可以大致猜出: 遍历了一遍放到list里

再看看str
```python

cc = count(3)
print(cc)
print(str(cc))
-------
<generator object count at 0x000002D1F8CB5EE0>
<generator object count at 0x000002D1F8CB5EE0>

```
这是咋回事。。。居然 cc is str(cc)吗？这里犯蠢了思考了好一会儿，终于意识到生成器本身可能没重写__str__函数。。。

## 异步io

上一节大致了解了生成器和迭代器的原理。接下来看看异步io与生成器到底有啥关系。本节内容参考了 [Python异步编程详解](https://blog.csdn.net/qq_20116223/article/details/116357403) ~~难以置信CSDN居然有这种干货文章~~

### io模型

对于一次IO访问(read)，数据会先被拷贝到操作系统内核的缓冲区，然后才从操作系统内核缓冲区拷贝到进程的地址空间

#### 阻塞访问(同步)

数据没拷贝过来，进程就先阻塞

#### 非阻塞IO

通过轮询或者其他机制检测，不释放CPU

#### IO多路复用

用一个进程同时处理监视多个IO请求的结果

#### 信号驱动

等待数据阶段就绪不阻塞，数据就绪后内核给进程发送信号，复制数据阶段阻塞

#### 异步IO

两个阶段都不阻塞，进程在接受到内核的信号前处理别的事务

### yield from

+ python允许一个生成器通过yield from将部分操作委派给另一个生成器