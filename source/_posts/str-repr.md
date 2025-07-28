---
title: __str__和__repr__的区别
date: 2025-05-17 12:52:59
tags:
---

|内建函数|__str__|__repr__|
|--|--|--|
| 目的 | 用户友好的字符串表示 | 官方字符串表示，供开发者调试使用 |
| 使用场景 | print、str或输出给用户时使用 | repr、交互式解释器或调试时使用 |
| 返回值 | 便于阅读 | 能够通过eval重建对象 |
| 默认行为 | 如果没有定义，回退到__repr__ | 默认的对象表示 |

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __str__(self):
        return f"Person(Name: {self.name}, Age: {self.age})"

    def __repr__(self):
        return f"Person('{self.name}', {self.age})"


p = Person("Alice", 30)
print(p)
print(eval(repr(p)))
```