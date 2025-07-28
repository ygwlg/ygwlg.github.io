---
title: 力扣周赛454
date: 2025-06-15 18:48:14
tags:
---

差16min就能ak了，最近的一次。。。

[题四-3585. 树中找到带权中位节点](https://leetcode.cn/problems/find-weighted-median-node-in-tree/)

树上倍增

```python
class Solution:
    def findMedian(self, n: int, edges: List[List[int]], queries: List[List[int]]) -> List[int]:
        parents = [0] * n
        conns = {_:[] for _ in range(n)}
        dises = {}
        heights = [0] * n
        weights = [0] * n
        for u, v, w in edges:
            conns[u].append((v, w))
            conns[v].append((u, w))
            dises[(u, v)] = w
            dises[(v, u)] = w
        def dfs(node, p):
            parents[node] = p
            for next, _ in conns[node]:
                if not next == p:
                    weights[next] = weights[node] + _
                    heights[next] = heights[node] + 1
                    dfs(next, node)
        dfs(0, -1)
        parents[0] = 0
        MAXP = n.bit_length() + 2
        parent_f = [[0] * MAXP for _ in range(n)]
        for i in range(n):
            parent_f[i][0] = parents[i]
            for j in range(1, MAXP):
                parent_f[i][j] = parent_f[parent_f[i][j - 1]][j - 1]

        def k_parent(a, k):
            for i in range(MAXP - 1, -1, -1):
                if k >= 1 << i:
                    k -= 1 << i
                    a = parent_f[a][i]
            return a

        def lca(na, nb):
            
            if heights[na] > heights[nb]:
                na, nb = nb, na
            
            nb = k_parent(nb, heights[nb] - heights[na])
            if na == nb:
                return na
            for i in range(MAXP - 1, -1, -1):
                pa, pb = parent_f[na][i], parent_f[nb][i]
                if not pa == pb:
                    na, nb = pa, pb
            p = parents[na]
            return p
            
        ans = []
        for a, b in queries:
            p = lca(a, b)
            # print(a, b, p)
            total = weights[a] - weights[p] + weights[b] - weights[p]
            if weights[a] == weights[b]:
                ans.append(p)
            elif weights[a] > weights[b]:
                start, target = a, total
                for i in range(MAXP - 1, -1, -1):
                    # print(i, start, total, weights[start], weights[parent_f[start][i]])
                    if ( weights[start] - weights[parent_f[start][i]]) * 2 < total:
                        total -= (weights[start] - weights[parent_f[start][i]] )  * 2
                        start = parent_f[start][i]
                # print(a, start, total)
                ans.append(parents[start])
            else:
                start = b
                for i in range(MAXP - 1, -1, -1):
                    # print(i, parent_f[start][i], weights[start])
                    if ( weights[start] - weights[parent_f[start][i]]) * 2 <= total:
                        total -= (weights[start] - weights[parent_f[start][i]]  )  * 2
                        start = parent_f[start][i]
                # print(b, start, '-----------')
                ans.append(start)
        return ans
                        
```
