---
title: 倍增算法
date: 2025-05-02 11:53:03
tags: 
---

## 算法思路

在探索递推关系式时，如果搜索状态空间很大，使用线性递归就无法满足时间和空间复杂度的要求，这个时候可以通过城北增长的方式，只递推状态空间在2的整数幂位置上的值作为代表。当需要其他位置上的值时间，可以通过任何证书都可以表示成若干个2的次幂项的和的性质来求出所需要拼成的值。

## 基本用法

查找单调数组的数值，例如：查找数组a={2,5,7,11,19}中最大小于12的数字

倍增做法：设定增长长度p和已确定长度l，判断a[l + p]是否满足条件，若满足则p*=2，否则缩小范围

```python
a = [2, 5, 7, 11, 19]

l = 0
p = 1
while p:
    if l + p < len(a) and a[l + p] < 12:
        l += p
        p <<= 1
    else:
        p >>= 1
print(l, a[l])
```

## 区间最值（RMQ）问题

```python
# RMQ问题
from math import log

nums = [34, 1, 8, 123, 3, 2]

queries = [[1, 2], [1, 5], [3, 4], [2, 3]]
n = len(nums)

k = int(log(n, 2)) + 1
f = [[0] * k for _ in range(n)]

for i in range(n):
    f[i][0] = nums[i]

for j in range(1, k):
    for i in range(n):
        if i + (1 << (j - 1)) >= n:
            f[i][j] = f[i][j - 1]
        else:
            f[i][j] = max(f[i][j - 1], f[i + (1 << (j - 1))][j - 1])


def query(l, r):
    qk = int(log(r - l + 1, 2))
    return max(f[l][qk], f[r - (1 << qk)][qk])


ret = list()
for x, y in queries:
    ret.append((query(x, y)))
print(ret)

```

## LCR在线计算问题

### 基本的LCR问题

[LCR 194. 二叉树的最近公共祖先](https://leetcode.cn/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

自己写了个路径跟踪的方法

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def lowestCommonAncestor(self, root: TreeNode, p: TreeNode, q: TreeNode) -> TreeNode:
        count = 0
        tmps = {root: [count]}
        count += 1
        def dfs(node):
            nonlocal tmps, count
            tmps[node] = [count]
            count += 1
            if node.left:
                dfs(node.left)
            if node.right:
                dfs(node.right)
            tmps[node].append(count)
            count += 1
        dfs(root)
        ret = root
        for node in tmps:
            # print(node.val,tmps[node])
            l, r = tmps[node]
            if l <= tmps[p][0] <= tmps[p][1] <= r and l <= tmps[q][0] <= tmps[q][1] <= r and r - l < tmps[ret][1] - tmps[ret][0]:
                ret = node
        return ret
```

勉强过了，但是由于额外维护了路径数组tmps，所以空间复杂度较高；再看看题解

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def lowestCommonAncestor(self, root: TreeNode, p: TreeNode, q: TreeNode) -> TreeNode:
        if root in (None, p, q):
            return root
        left = self.lowestCommonAncestor(root.left, p, q)
        right = self.lowestCommonAncestor(root.right, p, q)
        if left and right:
            return root
        return left or right
```
非常简洁，lowestCommonAncestor函数的含义为寻找p或q的父节点，如果在左右两侧都找到了，说明root即为最近公共父节点，否则需要向左或者向右递归

### 如何优化在线计算的时间复杂度

对应一个查询queries=[(p1, q1), (p2, q2)...]而言，如果对于每个查询都跑一遍上面的dfs代码，那么整段算法的时间复杂度为n*m，其中n为数的规模，m为查询的规模。

**树上倍增**

```python
from math import log

adj = {
    0: [1, 2],
    1: [0, 3],
    2: [0],
    3: [1, 4],
    4: [3]
}
n = 5
root = 0

max_level = int(log(n, 2)) + 1

f = [[-1] * max_level for _ in range(n)]
f[root][0] = root
array = [root]

depth = [0] * n
depth[root] = 0
while array:
    na = []
    for node in array:
        for child in adj[node]:
            if not child == f[node][0]:
                depth[child] = depth[node] + 1
                f[child][0] = node
                na.append(child)
    array = na

for j in range(1, max_level):
    for i in range(n):
        f[i][j] = f[f[i][j - 1]][j - 1]


def lca(p, q):
    if depth[p] < depth[q]:
        p, q = q, p
    for i in range(max_level - 1, -1, -1):
        if depth[p] - (1 << i) >= depth[q]:
            p = f[p][i]
    if p == q:
        return p
    for i in range(max_level - 1, -1, -1):
        if not f[p][i] == f[q][i]:
            p = f[p][i]
            q = f[q][i]
    return f[p][0]


queries = [(0, 1), (4, 2), (3, 4)]
for a, b in queries:
    print(lca(a, b))

```

整个代码分为两个部分：

### 预处理

用函数f[i][j]代表节点i的2^j祖先，其有递推关系式f[i][j] = f[f[i][j - 1]][j - 1]，这一函数用于快速计算某个节点的第k祖先（这是由于任意一个数k都可以拆解为2的次幂和）。

### 查询

对于每个查询p，q，首先将它们保持到统一高度，例如p更深一点，就去计算depth[p]-depth[q]的各项二次幂的值，并根据f[i][j]数组找出p在q这一层的父节点；
接着同时将它们向上搜索父节点，如果在某一层搜索的父节点是一样的，那么就将p，q分别置为该层的父节点，否则不进行操作。

为什么两次循环都要是反向循环呢？

1. 对于第一次循环，需要找出p在q这一层的父节点，由于depth[p]-depth[q]是一个定值，例如6，其二进制表示为110(2)。如果是正向遍历，就不能够通过值比较来求，否则取值和剩余值的序列为1/5、2/3、0/3;之后再怎么遍历都会忽略这个3

2. 对于第二次循环，需要同时将p，q向上层寻找节点，由于一开始的p，q与最终要求的p，q的深度差是一个定值，也同样会面临上述的问题。此时由于这个值没法像上面的值一样提前计算，所以还没发通过移位来求。因此这里的更不能改成正序遍历

## 试试树上倍增 [1483. 树节点的第 K 个祖先](https://leetcode.cn/problems/kth-ancestor-of-a-tree-node/)

```python
class TreeAncestor:

    def __init__(self, n: int, parent: List[int]):
        m = int(log(n, 2)) + 1
        self.m = m
        self.f = [[-1] * m for _ in range(n)]
        for i, p in enumerate(parent):
            self.f[i][0] = p
        
        for j in range(1, m):
            for i in range(n):
                if self.f[i][j - 1] == -1:
                    self.f[i][j - 1] = -1
                else:
                    self.f[i][j] = self.f[self.f[i][j - 1]][j - 1]

    def getKthAncestor(self, node: int, k: int) -> int:
        for i in range(self.m - 1, -1, -1):
            if node == -1:
                return -1
            if k >= 1 << i:
                k -= 1 << i
                node = self.f[node][i]
        return node
        


# Your TreeAncestor object will be instantiated and called as such:
# obj = TreeAncestor(n, parent)
# param_1 = obj.getKthAncestor(node,k)
```

### [3534. 针对图的路径存在性查询 II](https://leetcode.cn/problems/path-existence-queries-in-a-graph-ii/)

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

一开始我在查询部分写的代码是
```python
c = 0
for i in range(m - 1, -1, -1):
    if li <= pa[ri][i]:
        ri = pa[ri][i]
        c += 1 << i
ret.append(c)
```

为啥老是不行，还会计算出很巨大的值？打印中间结果发现，这是把所有超过最左节点的跳跃重点都置为左节点，所以会重复计算很多次。

改掉之后又出现了一个问题，如下图

![](/images/leetcode/leetcode3534.jpg)

把m加1之后就正常了，这是为啥呢

题解代码里用的是

```python
m = n.bit_length()
```

和我代码里的计算结果一致，但是赋值为-1的方式不同，我在最开始就判断是否能抵达，忘记了最后一跳。。。

