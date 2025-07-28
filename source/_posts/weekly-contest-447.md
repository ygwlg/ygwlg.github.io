---
title: 力扣竞赛447
date: 2025-04-29 21:24:23
tags:
   - Leetcode
   - contest
---

幸亏题三想出来，不然刷这么久总觉得没什么提升。~~纪念一下为数不多能在竞赛中通过的hard~~

[题三-3533. 判断连接可整除性]

思路：nums最长长度为13，其阶乘超过10^6，因此无法遍历所有的排列。

以下是全排列+回溯的做法，果不其然TLE了

```python
class Solution:
    def concatenatedDivisibility(self, nums: List[int], k: int) -> List[int]:
        nums.sort()
        n = len(nums)
        choosed = [False] * n
        tmp = []
        if_continue = True
        def dfs(i, r):
            nonlocal if_continue
            if i == n and r == 0:
                if_continue = False
            if if_continue:
                for j in range(n):
                    if not choosed[j]:
                        choosed[j] = True
                        tmp.append(nums[j])
                        nr = (r * (10 ** len(str(nums[j])) % k) + nums[j]) % k
                        dfs(i + 1, nr)
                        if not if_continue:
                            return 
                        choosed[j] = False
                        tmp.pop(-1)
        dfs(0, 0)
        return tmp
```

因此需要跳过一些状态的计算，这时考虑到i向后遍历时对于相同的r，前面已选的数字并不考虑其顺序，只需要考虑选取了哪些。所以用一个二进制数字来表示已选元素的集合。就有了如下的代码，果真通过了

```python
class Solution:
    def concatenatedDivisibility(self, nums: List[int], k: int) -> List[int]:
        nums.sort()
        n = len(nums)
        tmp = []
        if_continue = True
        @cache
        def dfs(i, r, s):
            nonlocal if_continue
            if i == n and r == 0:
                if_continue = False
            if if_continue:
                for j in range(n):
                    c = (s >> j) & 1
                    
                    if not c:
                        tmp.append(nums[j])
                        nr = (r * (10 ** len(str(nums[j])) % k) + nums[j]) % k
                        dfs(i + 1, nr, s | (1 << j))
                        if not if_continue:
                            return 
                        tmp.pop(-1)
        dfs(0, 0, 0)
        return tmp
                        
```

其实上述代码不需要i，因为当所有数字被选取后，掩码s即为 (1 << n) - 1

## [3534. 针对图的路径存在性查询 II](https://leetcode.cn/problems/path-existence-queries-in-a-graph-ii/)

```python
class Solution:
    def pathExistenceQueries(self, n: int, nums: List[int], maxDiff: int, queries: List[List[int]]) -> List[int]:
        idx = list(range(n))
        idx.sort(key=lambda x: nums[x])
        xdi = [0] * n
        for i, j in enumerate(idx):
            xdi[j] = i

        left = 0
        m = int(log(n, 2)) + 2
        pa = [[0] * m for _ in range(n)]
        for i, j in enumerate(idx):
            while nums[j] - nums[idx[left]] > maxDiff:
                left += 1
            pa[i][0] = left
        
        for j in range(1, m):
            for i in range(n):
                pa[i][j] = pa[pa[i][j - 1]][j - 1]
        
        ret = []
        for l, r in queries:
            li = xdi[l]
            ri = xdi[r]
            
            if li > ri:
                li, ri = ri, li
            if pa[ri][-1] > li:
                ret.append(-1)
            elif li == ri:
                ret.append(0)
            else:
                c = 0
                for i in range(m - 1, -1, -1):
                    if li < pa[ri][i]:
                        ri = pa[ri][i]
                        c += 1 << i
                ret.append(c + 1)
        return ret
```