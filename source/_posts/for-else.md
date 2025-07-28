---
title: for else语法
date: 2025-05-17 13:22:18
tags:
---

for ... else ...语法：else后的代码仅在for循环完整结束后运行

```python
for i in range(10):
    pass
else:
    print('success')
>> success

for i in range(10):
    break
else:
    print('success')
>> 
```