---
title: python函数的几种特殊参数类型
date: 2025-07-14 14:54:18
tags:
  - Python
---


## 带默认值的参数

```Python
def greet(name, age=18):
    print(f"Hello, {name}! You are {age} years old.")

greet("Alice")  # 输出: Hello, Alice! You are 18 years old.
greet("Bob", 25)  # 输出: Hello, Bob! You are 25 years old.
```

## 可变参数

```Python
def example_function(*args, **kwargs):
    print("Positional arguments:", args)
    print("Keyword arguments:", kwargs)

example_function(1, 2, 3, a=4, b=5)
```

## 仅限关键字参数

* 之后的参数只能通过关键字方式传递

```Python
def greet(name, *, age=18):
    print(f"Hello, {name}! You are {age} years old.")

greet("Alice", age=25)  # 输出: Hello, Alice! You are 25 years old.
greet("Bob")  # 输出: Hello, Bob! You are 18 years old.
```

## 仅限位置参数

/ 之前的参数只能通过位置传递

```Python
def greet(name, /, age=18):
    print(f"Hello, {name}! You are {age} years old.")

greet("Alice", 25)  # 输出: Hello, Alice! You are 25 years old.
greet(name="Alice", age=25)  # 报错：name 参数必须通过位置传递。
```


函数的参数优先级

1. 位置参数
2. 默认参数
3. *args
4. 仅限关键字参数
5. **kwargs
