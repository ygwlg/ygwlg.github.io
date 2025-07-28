---
title: 偏函数
date: 2025-05-17 20:07:26
tags:
---


```python
from functools import partial
add1 = partial(add, 1)

baseTwo = partial(int, base=2)
```