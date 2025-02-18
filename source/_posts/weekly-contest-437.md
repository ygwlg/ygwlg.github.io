---
title: Leetcode周赛437
date: 2025-02-16 13:39:42
tags: 
   - Leetcode
   - contest
categories: Leetcode

---
拼尽全力只做出题一，这把掉大分😢 

## [题一 3456. 找出长度为 K 的特殊子字符串](https://leetcode.cn/problems/find-special-substring-of-length-k)

简单模拟

## [题二 3457. 吃披萨](https://leetcode.cn/problems/eat-pizzas/)

贪心策略选取有误：先选取奇数天，再选取偶数天;竞赛的时候咋没想到，再也不在公司打周赛了（不是

```python
class Solution:
    def maxWeight(self, pizzas: List[int]) -> int:
        n = len(pizzas)
        pizzas.sort()
        days = n // 4
        ret = 0
        pizzas = pizzas[n // 2:]
        o, e =  days >> 1, days - (days >> 1)
        for i in range(e):
            ret += pizzas.pop(-1)
            pizzas.pop(0)
        for i in range(o):
            pizzas.pop(-1)
            ret += pizzas.pop(-1)
        return ret
```
## [题三 3458. 选择 K 个互不重叠的特殊子字符串](https://leetcode.cn/problems/select-k-disjoint-special-substrings/)

题二没做出来心态爆炸，题三更是一眼没思路😭

### 题解

1. 创建包含关系的有向图
2. 深搜有向图，对于每个字母，至少要达到的左右边界值
3. 上述边界值是s的字串，求取最长不交叉字串 [经典贪心问题](https://leetcode.cn/problems/non-overlapping-intervals/)
4. **特判**

### 实现

```python
class Solution:
    def maxSubstringLength(self, s: str, k: int) -> bool:
        min_max = {}
        for i, item in enumerate(s):
            if item not in min_max:
                min_max[item] = [inf, 0]
            min_max[item][0], min_max[item][1] = min(min_max[item][0], i), max(min_max[item][1], i)
        
        nexts = {}
        for i in range(26):
            item = chr(ord('a') + i)
            if item in min_max:
                nexts[item] = set()
                l, r = min_max[item]
                for _ in range(l + 1, r):
                    if not s[_] == item:
                        nexts[item].add(s[_])

        for item in min_max:
            walked = {chr(ord('a') + _): False for _ in range(26)}
            stack = [item]
            while stack:
                t = stack.pop()
                min_max[item][0] = min(min_max[item][0], min_max[t][0])
                min_max[item][1] = max(min_max[item][1], min_max[t][1])
                walked[t] = True
                for n in nexts[t]:
                    if not walked[n]:
                        stack.append(n)
        strs = list(min_max.values())
        if all([a == 0 and b == len(s) - 1 for a, b in strs]):
            return k == 0
        strs.sort()
        last = -1
        reminds = len(strs)
        for a, b in strs:
            if a < last:
                reminds -= 1
                if b < last:
                    last = b
            else:
                last = b
        return reminds >= k


                        
```

## [题四 3459. 最长 V 形对角线段的长度](https://leetcode.cn/problems/length-of-longest-v-shaped-diagonal-segment/)

题二题三没写出来心态爆炸，题四更是连题干都没看；其实只是难度一般的二维DP😂

```python
class Solution:
    def lenOfVDiagonal(self, grid: List[List[int]]) -> int:
        n = len(grid)
        m = len(grid[0])
        directions = [(-1, 1), (1, 1), (1, -1), (-1, -1)]
        ans = 0
        @cache
        def dfs(x, y, d, turn):
            dx, dy = directions[d]
            nx, ny = x + dx, y + dy

            ret = 0

            if 0 <= nx < n and 0 <= ny < m and grid[nx][ny] == 2 - grid[x][y]:
                ret = max(ret, dfs(nx, ny, d, turn))

                if not turn:
                    d = (d + 1) % 4
                    dx, dy = directions[d]
                    nx, ny = x + dx, y + dy
                    if 0 <= nx < n and 0 <= ny < m and grid[nx][ny] == 2 - grid[x][y]:
                        ret = max(ret, dfs(nx, ny, d, True))

                

            elif not turn:
                d = (d + 1) % 4
                dx, dy = directions[d]
                nx, ny = x + dx, y + dy
                if 0 <= nx < n and 0 <= ny < m and grid[nx][ny] == 2 - grid[x][y]:
                    ret = max(ret, dfs(nx, ny, d, True))
            
            return ret + 1


        for i in range(n):
            for j in range(m):
                if grid[i][j] == 1:
                    for di in range(4):
                        ni = i + directions[di][0]
                        nj = j + directions[di][1]
                        ans = max(ans, 1)
                        if 0 <= ni < n and 0 <= nj < m and grid[ni][nj] == 2: 
                            ans = max(ans, 1 + dfs(ni, nj, di, False))
        return ans
```
