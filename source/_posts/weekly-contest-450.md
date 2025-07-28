---
title: 力扣周赛450
date: 2025-05-18 15:33:31
tags:
---

能想得出LCA，但是想不出如何用LCA计算路径。。。最终274名，难道这就是极限了吗

[题二-3551. 数位和排序需要的最小交换次数](https://leetcode.cn/problems/minimum-swaps-to-sort-by-digit-sum/)

先排序算出正确的位置，之后根据正确的位置求得最小交换数量。

1. 计算置换环

```python
class Solution:
    def minMoves(self, matrix: List[str]) -> int:
        gates = {chr(_ + ord('A')):[] for _ in range(26)}
        m, n  = len(matrix), len(matrix[0])
        for i in range(m):
            for j in range(n):
                if 'A' <= matrix[i][j] <= 'Z':
                    gates[matrix[i][j]].append((i, j))
        dises = [[inf] * n for _ in range(m)]
        dises[0][0] = 0
        poses = [(0, 1), (0, -1), (1, 0), (-1, 0)]

        gates_used = {chr(_ + ord('A')):False for _ in range(26)}
        # @cache
        heap = [(0, 0, 0)]
        def bfs():
            while heap:
                v, i, j = heappop(heap)
                if i == m - 1 and j == n - 1:
                    return
                
                
                if not matrix[i][j] == '.' and not gates_used[matrix[i][j]]:
                    gates_used[matrix[i][j]] = True
                    
                    for gi, gj in gates[matrix[i][j]]:
                        if gi == i and gj == j:
                            continue
                        if dises[i][j] < dises[gi][gj]:
                            dises[gi][gj] = dises[i][j]
                            heappush(heap, (dises[gi][gj], gi, gj))
                            
                for di, dj in poses:
                    if 0 <= i + di <= m - 1 and 0 <= j + dj <= n - 1:
                        if not matrix[i + di][j + dj] == '#':
                            if dises[i][j] + 1 < dises[i + di][j + dj]:
                                dises[i + di][j + dj] = dises[i][j] + 1
                                heappush(heap, (dises[i + di][j + dj], i + di, j + dj))
        bfs()
        ret = dises[-1][-1]
        if ret == inf:
            return -1
        return ret
```

2. 连通块计算，从交换前的序号到交换后的序号连边，可以得到一个有向图，n - 连通块的个数即为最终答案

[题三-3552. 网格传送门旅游](https://leetcode.cn/problems/grid-teleportation-traversal/)

一开始用纯粹的bfs做的，结果给TLE了；后来用贪心的方式优化bfs过程，即优先搜索离起点最近的点，搜索到终点时停。

```python
class Solution:
    def minMoves(self, matrix: List[str]) -> int:
        gates = {chr(_ + ord('A')):[] for _ in range(26)}
        m, n  = len(matrix), len(matrix[0])
        for i in range(m):
            for j in range(n):
                if 'A' <= matrix[i][j] <= 'Z':
                    gates[matrix[i][j]].append((i, j))
        dises = [[inf] * n for _ in range(m)]
        dises[0][0] = 0
        poses = [(0, 1), (0, -1), (1, 0), (-1, 0)]

        gates_used = {chr(_ + ord('A')):False for _ in range(26)}
        # @cache
        heap = [(0, 0, 0)]
        def bfs():
            while heap:
                v, i, j = heappop(heap)
                if i == m - 1 and j == n - 1:
                    return
                
                
                if not matrix[i][j] == '.' and not gates_used[matrix[i][j]]:
                    gates_used[matrix[i][j]] = True
                    
                    for gi, gj in gates[matrix[i][j]]:
                        if gi == i and gj == j:
                            continue
                        if dises[i][j] < dises[gi][gj]:
                            dises[gi][gj] = dises[i][j]
                            heappush(heap, (dises[gi][gj], gi, gj))
                            
                for di, dj in poses:
                    if 0 <= i + di <= m - 1 and 0 <= j + dj <= n - 1:
                        if not matrix[i + di][j + dj] == '#':
                            if dises[i][j] + 1 < dises[i + di][j + dj]:
                                dises[i + di][j + dj] = dises[i][j] + 1
                                heappush(heap, (dises[i + di][j + dj], i + di, j + dj))
        bfs()
        ret = dises[-1][-1]
        if ret == inf:
            return -1
        return ret
```

题解的做法：边的权值只有0和1，用不着堆，双端队列即可解决问题。

## [题四-3553. 包含给定路径的最小带权子树 II](https://leetcode.cn/problems/minimum-weighted-subgraph-with-the-required-paths-ii/)

先计算每个节点到根节点的距离，再计算两点的最近父节点，A，B两点的树上距离就等于A和B到根节点的距离之和减去双倍的公共父节点到根节点的距离。

对于每个询问有结果值等于(|A - B| + |B - C| + | A - C |) / 2

按照意思写了一下：

```python
class Solution:
    def minimumWeight(self, edges: List[List[int]], queries: List[List[int]]) -> List[int]:
        
        conn = {}
        n = 0
        for a, b, v in edges:
            if a not in conn:
                conn[a] = []
            if b not in conn:
                conn[b] = []
            conn[a].append((b, v))
            conn[b].append((a, v))
            n = max(n, a, b)
        n += 1
        dises = [0] * n
        
        parents = [-1] * n
        def dfs(node, parent):
            parents[node] = parent
            for c, d in conn[node]:
                if not c == parent:
                    dises[c] = dises[node] + d
                    dfs(c, node)
        dfs(0, -1)

        @cache
        def lca(root, a, b):
            if root in [None, a, b]:
                return root
            r = []
            for child, _ in conn[root]:
                if len(r) == 2:
                    return root
                if not child == parents[root]:
                    ret = lca(child, a, b)
                    if ret:
                        r.append(ret)
            if len(r) == 2:
                return root
            
            if len(r) == 1:
                return r[0]

        def cal_dis(a, b):
            if a < b:
                a, b = b, a
            p = lca(0, a, b)
            return dises[a] + dises[b] - 2 * dises[p]
        
        ans = []
        for a, b ,c in queries:
            ans.append((cal_dis(a, b) + cal_dis(a, c) + cal_dis(c, b)) // 2)
        return ans
```

结果给TLE了，原来还是要树上倍增。。。

来来回回TLE好多次，终于。。。
```python
class Solution:
    def minimumWeight(self, edges: List[List[int]], queries: List[List[int]]) -> List[int]:
        
        conn = {}
        n = 0
        for a, b, v in edges:
            if a not in conn:
                conn[a] = []
            if b not in conn:
                conn[b] = []
            conn[a].append((b, v))
            conn[b].append((a, v))
            n = max(n, a, b)
        n += 1
        dises = [0] * n
        depth = [0] * n

        parents = [-1] * n
        def dfs(node, parent):
            parents[node] = parent
            for c, d in conn[node]:
                if not c == parent:
                    dises[c] = dises[node] + d
                    depth[c] = depth[node] + 1
                    dfs(c, node)
        dfs(0, 0)
        
        MAX_DEPTH = n.bit_length() + 1

        @cache
        def dfs_f(i, j):
            if i == 0:
                return 0
            if j == 0:
                return parents[i]
            else:
                return dfs_f(dfs_f(i, j - 1), j - 1)

        @cache
        def k_parent(a, k):
            for i in range(MAX_DEPTH - 1, -1, -1):
                if k >= 1 << i:
                    k -= 1 << i
                    a = dfs_f(a, i)
            return a


        def cal_dis(a, b):
            x, y = a, b
            if depth[a] < depth[b]:
                a, b = b, a
            a = k_parent(a, depth[a] - depth[b])
            if a == b:
                p = a
            else:
                for i in range(MAX_DEPTH - 1, -1, -1):
                    pa, pb = dfs_f(a, i), dfs_f(b, i)
                    if not pa == pb:
                        a, b = pa, pb
                p = parents[a]

            return dises[x] + dises[y] - 2 * dises[p]
        
        ans = []
        for a, b ,c in queries:
            ans.append((cal_dis(a, b) + cal_dis(a, c) + cal_dis(c, b)) // 2)
        return ans
```