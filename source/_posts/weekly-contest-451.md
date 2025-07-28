---
title: 力扣周赛451
date: 2025-05-25 16:38:51
tags:
---

上周树上倍增，这周树上背包，是不是每个算法都能和树结合。。。

[题三-3562. 折扣价交易股票的最大利润](https://leetcode.cn/problems/maximum-profit-from-trading-stocks-with-discounts/)

```python
max = lambda a, b: b if b > a else a
class Solution:
    def maxProfit(self, n: int, present: List[int], future: List[int], hierarchy: List[List[int]], budget: int) -> int:
        subs =  {_: [] for _ in range(n + 1)}
        subs[0].append(1)
        parents = [0] * (n + 1)
        for p, c in hierarchy:
            subs[p].append(c)
            parents[c] = p
        

        def dfs(x):
            fx = [[0, 0] for _ in range(budget + 1)]
            for y in subs[x]:
                fy = dfs(y)
                for j in range(budget, -1, -1):
                    for jy in range(j + 1):
                        for k in range(2):
                            fx[j][k] = max(fx[j][k], fx[j - jy][k] + fy[jy][k])
            
            f = [[0, 0] for _ in range(budget + 1)]
            
            for j in range(budget + 1):
                for k in range(2):
                    cost = present[x - 1] // (k + 1)
                    if j >= cost:
                        f[j][k] = max(fx[j][0], fx[j - cost][1] + future[x - 1] - cost)
                    else:
                        f[j][k] = fx[j][0]
            return f
        
        return dfs(1)[-1][0]
```

[题四-3563. 移除相邻字符后字典序最小的字符串](https://leetcode.cn/problems/lexicographically-smallest-string-after-adjacent-removals/)

区间dp用于计算一段字符是否能被消除，对每个字符进行dp求得结果

```python
class Solution:
    def lexicographicallySmallestString(self, s: str) -> str:
        def is_conn(a, b):
            ret = abs(ord(a) - ord(b)) 
            return ret == 1 or ret == 25

        @cache
        def can_ep(i, j):
            if i > j:
                return True
            if is_conn(s[i], s[j]) and can_ep(i + 1, j - 1):
                return True
            for k in range(i + 1, j - 1, 2):
                if can_ep(i, k) and can_ep(k + 1, j):
                    return True
            return False
        
        @cache
        def dfs(i):
            if i == n:
                return ''
            ret = s[i] + dfs(i + 1)
            for j in range(i + 1, n):
                if can_ep(i, j):
                    t = dfs(j + 1)
                    if ret > t:
                        ret = t
            return ret
        n = len(s)
        return dfs(0)
```