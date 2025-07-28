---
title: weekly-contest-452
date: 2025-06-01 14:09:41
tags:
---

## [题三-3568. 清理教室的最少移动](https://leetcode.cn/problems/minimum-moves-to-clean-the-classroom/)

多目标搜索问题：用一个目标掩码mask作为搜索状态之一。

旅行商问题！

写了一个bfs：

```python
class Solution:
    def minMoves(self, classroom: List[str], energy: int) -> int:
        dirs = [(1, 0), (-1, 0), (0, 1), (0, -1)]
        ls = dict()
        lc = 0
        n, m = len(classroom), len(classroom[0])
        for i in range(n):
            for j in range(m):
                if classroom[i][j] == 'S':
                    si, sj = i, j
                elif classroom[i][j] == 'L':
                    ls[(i, j)] = lc
                    lc += 1
        if lc == 0:
            return 0
        
        vis = [[[[False] * (1 << len(ls)) for e1 in range(energy + 1)] for j in range(m)] for i in range(n)]

        q = [(si, sj, energy, (1 << len(ls)) - 1)]
        ans = 0
        while q:
            nq = []
            for i, j, e, mask in q:
                if mask == 0:
                    return ans
                vis[i][j][e][mask] = True
                if classroom[i][j] == 'R':
                    e = energy
                
                if e > 0:
                    for di, dj in dirs:
                        if 0 <= i + di <= n - 1 and 0 <= j + dj <= m - 1 and not classroom[i + di][j + dj] == 'X':
                            if classroom[i + di][j + dj] == 'L':
                                nm = mask & ~(1 << ls[(i + di, j + dj)])
                            else:
                                nm = mask
                            if not vis[i + di][j + dj][e - 1][nm]:
                                nq.append((i + di, j + dj, e - 1, nm))
            ans += 1
            q = nq
        return -1
```

结果TLE了，按照教程的提前计算取反值试试？也不行，卡常真恶心。还是得再学一门新的语言作为常用语言。。。

## [3569. 分割数组后不同质数的最大数目](https://leetcode.cn/problems/maximize-count-of-distinct-primes-after-split/)

把计算两个部分不同质数个数的问题转化为计算总数组不同质数个数加划分后新增质数个数的问题。总的质数个数可以通过维护一个集合记录，新增的质数和每个质数的左右边界相关，每当划分范围在左右边界中间的时候，对应质数的个数就可以加一。因此可以维护一个lazy线段树，在每次询问后用质数的左右边界更新该线段树。

```python
MX = 100_001
is_p = [True] * MX
is_p[0] = is_p[1] = False
for i in range(2, MX):
    if is_p[i]:
        for j in range(i * i, MX, i):
            is_p[j] = False


class Node:
    __slots__ = 'val', 'todo'

class LazySegmentTree:
    # 懒标记初始值
    _TODO_INIT = 0

    def __init__(self, n: int, init_val=0):
        # 线段树维护一个长为 n 的数组（下标从 0 到 n-1），元素初始值为 init_val
        # init_val 可以是 list 或者 int
        # 如果是 int，那么创建一个 list
        if isinstance(init_val, int):
            init_val = [init_val] * n
        self._n = n
        self._tree = [Node() for _ in range(2 << (n - 1).bit_length())]
        self._build(init_val, 1, 0, n - 1)

    # 合并两个 val
    def _merge_val(self, a: int, b: int) -> int:
        return a if a > b else b

    # 把懒标记作用到 node 子树（本例为区间加）
    def _apply(self, node: int, l: int, r: int, todo: int) -> None:
        cur = self._tree[node]
        # 计算 tree[node] 区间的整体变化
        cur.val += todo
        cur.todo += todo

    # 把当前节点的懒标记下传给左右儿子
    def _spread(self, node: int, l: int, r: int) -> None:
        todo = self._tree[node].todo
        if todo == self._TODO_INIT:  # 没有需要下传的信息
            return
        m = (l + r) // 2
        self._apply(node * 2, l, m, todo)
        self._apply(node * 2 + 1, m + 1, r, todo)
        self._tree[node].todo = self._TODO_INIT  # 下传完毕

    # 合并左右儿子的 val 到当前节点的 val
    def _maintain(self, node: int) -> None:
        self._tree[node].val = self._merge_val(self._tree[node * 2].val, self._tree[node * 2 + 1].val)

    # 用 a 初始化线段树
    # 时间复杂度 O(n)
    def _build(self, a: List[int], node: int, l: int, r: int) -> None:
        self._tree[node].todo = self._TODO_INIT
        if l == r:  # 叶子
            self._tree[node].val = a[l]  # 初始化叶节点的值
            return
        m = (l + r) // 2
        self._build(a, node * 2, l, m)  # 初始化左子树
        self._build(a, node * 2 + 1, m + 1, r)  # 初始化右子树
        self._maintain(node)

    def _update(self, node: int, l: int, r: int, ql: int, qr: int, f: int) -> None:
        if ql <= l and r <= qr:  # 当前子树完全在 [ql, qr] 内
            self._apply(node, l, r, f)
            return
        self._spread(node, l, r)
        m = (l + r) // 2
        if ql <= m:  # 更新左子树
            self._update(node * 2, l, m, ql, qr, f)
        if qr > m:  # 更新右子树
            self._update(node * 2 + 1, m + 1, r, ql, qr, f)
        self._maintain(node)

    def _query(self, node: int, l: int, r: int, ql: int, qr: int) -> int:
        if ql <= l and r <= qr:  # 当前子树完全在 [ql, qr] 内
            return self._tree[node].val
        self._spread(node, l, r)
        m = (l + r) // 2
        if qr <= m:  # [ql, qr] 在左子树
            return self._query(node * 2, l, m, ql, qr)
        if ql > m:  # [ql, qr] 在右子树
            return self._query(node * 2 + 1, m + 1, r, ql, qr)
        l_res = self._query(node * 2, l, m, ql, qr)
        r_res = self._query(node * 2 + 1, m + 1, r, ql, qr)
        return self._merge_val(l_res, r_res)

    # 用 f 更新 [ql, qr] 中的每个 a[i]
    # 0 <= ql <= qr <= n-1
    # 时间复杂度 O(log n)
    def update(self, ql: int, qr: int, f: int) -> None:
        self._update(1, 0, self._n - 1, ql, qr, f)

    # 返回用 _merge_val 合并所有 a[i] 的计算结果，其中 i 在闭区间 [ql, qr] 中
    # 0 <= ql <= qr <= n-1
    # 时间复杂度 O(log n)
    def query(self, ql: int, qr: int) -> int:
        return self._query(1, 0, self._n - 1, ql, qr)


class Solution:
    def maximumCount(self, nums: List[int], queries: List[List[int]]) -> List[int]:
        n = len(nums)
        t = LazySegmentTree(n)
        ps = {}
        for i, num in enumerate(nums):
            if is_p[num]:
                if not num in ps:
                    ps[num] = SortedList()
                ps[num].add(i)
        ret = []
        for pv in ps.values():
            if len(pv) > 1:
                t.update(pv[0], pv[-1], 1)
        for a, b in queries:
            old = nums[a]
            if is_p[old]:
                if len(ps[old]) > 1:
                    t.update(ps[old][0], ps[old][-1], -1)

                ps[old].remove(a)
                if len(ps[old]) > 1:
                    t.update(ps[old][0], ps[old][-1], 1)
                elif len(ps[old]) == 0:
                    del ps[old]
                
            if is_p[b]:
                if not b in ps:
                    ps[b] = SortedList()
                else:
                    if len(ps[b]) > 1:
                        t.update(ps[b][0], ps[b][-1], -1)
                ps[b].add(a)
                
                if len(ps[b]) > 1:
                    t.update(ps[b][0], ps[b][-1], 1)
            nums[a] = b
            ret.append(len(ps) + t.query(0, n - 1))
        return ret

```

