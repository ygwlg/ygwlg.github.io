---
title: 力扣周赛-456
date: 2025-06-29 13:55:14
tags:
---

划分型dp的标准做法忘记了，题三调半天最终还是TLE。。。

## [3599. 划分数组得到最小 XOR](https://leetcode.cn/problems/partition-array-to-minimize-xor/description/)

一开始的做法：用i，r和遗留的xor值作为状态转移的参数，结果给内存干超了。

```python
class Solution:
    def minXor(self, nums: List[int], k: int) -> int:
        n = len(nums)
        @cache
        def dfs(i, r, last):
            if i == n and r == 0:
                return 0
            elif i == n:
                return inf
            if r <= 0 and i < n:
                return inf
            ne = dfs(i + 1, r, last ^ nums[i])
            
            ne2 = max(last ^ nums[i], dfs(i + 1, r - 1, 0))
            return min(ne, ne2)
            
            
        return dfs(0, k, 0)
```
last值为划分的前缀，可能的取值范围有n * n，所以会爆内存

标准做法是枚举下一次划分的范围，而非上面的枚举是否划分

```python
class Solution:
    def minXor(self, nums: List[int], k: int) -> int:
        n = len(nums)
        @cache
        def dfs(i, r):
            if i == n and r == 0:
                return 0
            elif i == n:
                return inf
            if r <= 0 and i < n:
                return inf
            s = nums[i]
            ret = inf
            for j in range(i + 1, n + 1):
                
                t = max(s, dfs(j, r - 1))
                if t < ret:
                    ret = t
                if j < n :
                    s ^= nums[j]

            return ret
            
            
        return dfs(0, k)
```
这么写居然也超时了。。。max换成大小比较试试，终于通过了，如下

```python
class Solution:
    def minXor(self, nums: List[int], k: int) -> int:
        n = len(nums)
        @cache
        def dfs(i, r):
            if i == n and r == 0:
                return 0
            elif i == n:
                return inf
            if r <= 0 and i < n:
                return inf
            s = nums[i]
            ret = inf
            for j in range(i + 1, n + 1):
                
                t = dfs(j, r - 1)
                if s > t:
                    t = s
                if t < ret:
                    ret = t
                if j < n :
                    s ^= nums[j]

            return ret
            
            
        return dfs(0, k)
```

[3600. 升级后最大生成树稳定性](https://leetcode.cn/problems/maximize-spanning-tree-stability-with-upgrades/description/)

二分搜索答案，并查集合并节点

```python
class UnionFind:
    def __init__(self, n: int):
        # 一开始有 n 个集合 {0}, {1}, ..., {n-1}
        # 集合 i 的代表元是自己
        self._fa = list(range(n))  # 代表元
        self.cc = n  # 连通块个数

    # 返回 x 所在集合的代表元
    # 同时做路径压缩，也就是把 x 所在集合中的所有元素的 fa 都改成代表元
    def find(self, x: int) -> int:
        # 如果 fa[x] == x，则表示 x 是代表元
        if self._fa[x] != x:
            self._fa[x] = self.find(self._fa[x])  # fa 改成代表元
        return self._fa[x]

    # 把 from 所在集合合并到 to 所在集合中
    # 返回是否合并成功
    def merge(self, from_: int, to: int) -> bool:
        x, y = self.find(from_), self.find(to)
        if x == y:  # from 和 to 在同一个集合，不做合并
            return False
        self._fa[x] = y  # 合并集合。修改后就可以认为 from 和 to 在同一个集合了
        self.cc -= 1  # 成功合并，连通块个数减一
        return True

class Solution:
    def maxStability(self, n: int, edges: List[List[int]], k: int) -> int:
        must_uf = UnionFind(n)
        all_uf = UnionFind(n)
        min_target, max_target = inf, 0
        for u, v, s, must in edges:
            if must:
                if not must_uf.merge(u, v):
                    return -1
            all_uf.merge(u, v)
            min_target = min(s, min_target)
            max_target = max(s, max_target)
        
        if all_uf.cc > 1:
            return -1
        
        def check(target):
            uf = UnionFind(n)
            for u, v, s, must in edges:
                if must and s < target:
                    return False
                if must or s >= target:
                    uf.merge(u, v)
            
            r = k
            for u, v, s, must in edges:
                if uf.cc == 1:
                    break
                if not must and s < target <= 2 * s and not uf.find(u) == uf.find(v):
                    uf.merge(u, v)
                    r -= 1
            
            return uf.cc == 1 and r >= 0

        l, r = min_target, 2 * max_target + 1
        while l + 1 < r:
            mid = (l + r) >> 1

            if check(mid):
                l = mid
            else:
                r = mid
        return l
```