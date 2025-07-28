---
title: 力扣周赛449
date: 2025-05-11 16:31:23
tags:
    - Leetcode
---

拼尽全力做出题三，一看名次300+，再一看题解说是错题，天塌了

## [（错题）题三-3547. 图中边值的最大和](https://leetcode.cn/problems/maximum-sum-of-edge-values-in-a-graph/)

并查集，优先填圈，后填线

```python
class Solution:
    def maxScore(self, n: int, edges: List[List[int]]) -> int:
        m = len(edges)
        values = [0] * n
        walked = [False] * n
        conns = {_: [] for _ in range(n)}
        for a, b in edges:
            conns[a].append(b)
            conns[b].append(a)

        lines = []
        circles = []

        def dfs1(node):
            walked[node] = True
            line.append(node)
            for next in conns[node]:
                if not walked[next]:
                    dfs1(next)

        def dfs2(node):
            walked[node] = True
            circle.append(node)
            for next in conns[node]:
                if not walked[next]:
                    dfs2(next)

        for i in range(n):
            if not walked[i] and len(conns[i]) == 1:
                line = []
                dfs1(i)
                lines.append(line)

        for i in range(n):
            if not walked[i] and len(conns[i]) > 1:
                circle = []
                dfs2(i)
                circles.append(circle)
        circles.sort(key=lambda x: len(x))
        lines.sort(key=lambda x: -len(x))
        start = n
        for circle in circles:
            l = len(circle)
            start -= l
            for i, c in enumerate(circle):
                if i < l >> 1:
                    values[c] = start + (i + 1) * 2
                else:
                    values[c] = start + 2 * l - 1 - 2 * i
        
        for line in lines:
            l = len(line)
            start -= l
            for i, c in enumerate(line):
                if i < l >> 1:
                    values[c] = start + (i + 1) * 2
                else:
                    values[c] = start + 2 * l - 1 - 2 * i

        ret = 0
        for a, b in edges:
            ret += values[a] * values[b]
        return ret
            
                
```

## [题四-3547. 图中边值的最大和](https://leetcode.cn/problems/maximum-sum-of-edge-values-in-a-graph/)

维护前缀集合即可，明明是这么简单的题，还剩20min却不敢下手，哈基米你这家伙

```python
class Solution:
    def canPartitionGrid(self, grid: List[List[int]]) -> bool:
        total = sum([sum(line) for line in grid])
        m, n = len(grid), len(grid[0])
        def search(g):
            s = {0}
            line_set = set()
            summary = 0
            for i in range(len(g)):
                for j in range(len(g[0])):
                    summary += g[i][j]
                    s.add(g[i][j])
                if len(g[0]) == 1:
                    if summary * 2 == total or summary * 2 - total == g[0][0] or summary * 2 - total == g[i][0]:
                        return True
                    continue
                if i == 0:
                    if 2 * summary - total in [g[0][0], g[0][-1], 0]:
                        return True
                else:
                    if 2 * summary - total in s:
                        return True
        
            return False
        
        g1 = grid
        if search(g1):
            return True
        
        g2 = [[0] * n for _ in range(m)]

        g3 = [[0] * m for _ in range(n)]
        g4 = [[0] * m for _ in range(n)]

        for i in range(m):
            for j in range(n):
                g2[i][j] = grid[m - i - 1][j]
                g3[j][i] = grid[i][j]
                g4[n - 1 - j][i] = grid[i][j]
        if search(g2):
            return True
        
        if search(g3):
            return True
        return search(g4)
```