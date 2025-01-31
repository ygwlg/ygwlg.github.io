---
title: leetcode-119
date: 2025-01-28 14:43:17
categories: Leetcode
tags:
    - Leetcode
---

## [119. 杨辉三角 II](https://leetcode.cn/problems/pascals-triangle-ii/)

难度：简单

### 思路

杨辉三角通项公式C(i, j)

### 实现

```python
class Solution:
    def getRow(self, rowIndex: int) -> List[int]:
        return [comb(rowIndex, _) for _ in range(rowIndex + 1)]
```

#### 题解

**C（i， j）递推式**

+ c[i][j]=c[i−1][j−1]+c[i−1][j]

```python
MX = 34
c = [[1] * (i + 1) for i in range(MX)]
for i in range(2, MX):
    for j in range(1, i):
        # 左上方的数 + 正上方的数
        c[i][j] = c[i - 1][j - 1] + c[i - 1][j]

class Solution:
    def getRow(self, rowIndex: int) -> List[int]:
        return c[rowIndex]
```