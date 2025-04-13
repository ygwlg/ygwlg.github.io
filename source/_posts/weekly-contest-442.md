---
title: weekly-contest-442
date: 2025-03-26 21:12:21
tags: 
    - weekly-contest
---

最后一题难度不算高，最接近ak的一次，但是没有把握住机会

## [题一-3492. 船上可以装载的最大集装箱数量](https://leetcode.cn/problems/maximum-containers-on-a-ship/)

简单模拟

```python
class Solution:
    def maxContainers(self, n: int, w: int, maxWeight: int) -> int:
        return min(maxWeight // w, n * n)
```

## [题二-3493. 属性图](https://leetcode.cn/problems/properties-graph/)

深度搜索统计联通分支

```python
class Solution:
    def numberOfComponents(self, properties: List[List[int]], k: int) -> int:
        edges = {_:[] for _ in range(len(properties))}
        for i in range(len(properties)):
            for j in range(i + 1, len(properties)):
                inter = set(properties[i]).intersection(set(properties[j]))
                if len(inter) >= k:
                    edges[i].append(j)
                    edges[j].append(i)

        ret = 0
        n = len(properties)
        walked = [False] * n
        for i in range(n):
            if not walked[i]:
                ret += 1
                tmps = [i]
                while tmps:
                    t = tmps.pop()
                    walked[t] = True
                    for e in edges[t]:
                        if not walked[e]:
                            tmps.append(e)
        return ret
```

## [题三-3494. 酿造药水需要的最少总时间](https://leetcode.cn/problems/find-the-minimum-amount-of-time-to-brew-potions/)

一开始是用正反两次扫描，结果TLE了，最后就是想办法把第二次扫描的过程合并到第一次扫描中，浪费老半天。。。

```python
class Solution:
    def minTime(self, skill: List[int], mana: List[int]) -> int:
        n  = len(skill)
        m = len(mana)
        ts = [0] * n
        last_start = 0
        ll = [0] * n
        for i in range(m):

            if i == 0:
                start = 0
            else:
                start = last_start + skill[0] * mana[i - 1]
            t = 0
            for j in range(n):
                if start + t < last_start + ll[j]:
                    start = last_start + ll[j] - t
                ts[j] = start + t + skill[j] * mana[i]
                t += skill[j] * mana[i]
                ll[j] = t

            last_start = start
            
        return ts[-1]
```

## [题四-3495. 使数组元素都变为零的最少操作次数](https://leetcode.cn/problems/minimum-operations-to-make-array-elements-zero/)

由于floor(a / 4) = a >> 2， 除法计算就可以简化为加减运算

竞赛的时候没反应过来，苦思半小时如何使用打表和二分的方式降低时间复杂度。。。丧失一次难得的ak机会。。。

```python
class Solution:
    def minOperations(self, queries: List[List[int]]) -> int:
        def f(n):
            mx = n.bit_length()
            ret = 0
            for i in range(1, mx):
                ret += (i + 1) // 2 << (i - 1)
            ret += (n + 1 - (1 << mx >> 1)) * ( (mx + 1) // 2)
            return ret
        
        ans = 0
        for l, r in queries:
            ans += (f(r) - f(l - 1) + 1) // 2
        return ans
```