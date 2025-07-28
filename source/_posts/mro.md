---
title: mro
date: 2025-05-17 22:07:52
tags:
---

假设有
```python
class A:
   pass

class B:
    def __init__(self):
        print('init')

class C(A, B):
    pass

```

所有的类默认继承于object，所以多继承一定是菱形结构，所以基于深度搜索的mro会出现下面的问题。

深度搜索的话，最终会访问到object的init，而逻辑上是要访问B的。。。

因此新式类采用基于bfs的mro方式，mro顺序可以调用__mro__内建方法打印

```python
>> C.__mro__
>> (<class '__main__.C'>, <class '__main__.A'>, <class '__main__.B'>, <class 'object'>)
```
