---
title: Leetcode周赛440
date: 2025-03-10 19:49:06
tags: 
    - Leetcode
    - contest
---

题三绞尽脑汁也无思路，居然是线段树。原来线段树可以解决非区间和之外的问题，~~哈基线，你这家伙真是可怕~~

## [题一-Q1. 将水果放入篮子 II](https://leetcode.cn/contest/weekly-contest-440/problems/fruits-into-baskets-ii/)

双重循环简单模拟

```python
class Solution:
    def numOfUnplacedFruits(self, fruits: List[int], baskets: List[int]) -> int:
        fruits.sort()
        ret = 0
        for i in range(len(fruits)):
            tmp = False
            for j in range(len(baskets)):
                if baskets[j] >= fruits[i]:
                    tmp = True
                    baskets[j] = 0
                    break
            if not tmp:
                ret += 1
        return ret
```

## [题二-Q2. 选出和最大的 K 个元素](https://leetcode.cn/problems/choose-k-elements-with-maximum-sum/)

+ 离线计算
+ 小顶堆

```python
class Solution:
    def findMaxSum(self, nums1: List[int], nums2: List[int], k: int) -> List[int]:
        n = len(nums1)
        com = list(zip(nums1, nums2, range(n)))
        com.sort()
        ret = [-1] * n
        h = []
        s = 0
        for i, (n1, n2, index) in enumerate(com):
            if i == 0:
                ret[index] = 0
            else:
                if n1 == com[i - 1][0]:
                    ret[index] = ret[com[i - 1][2]]
                else:
                    ret[index] = s
            heappush(h, n2)
            s += n2
            if len(h) > k:
                s -= heappop(h)
        return ret
            
```

## [题三-3479. 将水果装入篮子 III]()

~~没想到除了区间和之外，区间最大值也能用到线段树解法。思维可能是被lowbit开点的线段树给局限了~~

```python
class SegTree:
    def __init__(self, l, r):
        self.left = None
        self.right = None
        self.l = l
        self.r = r
        self.max = -1
        self.parent = None
    
    


class Solution:
    def build(self, l, r, baskets):
        if l == r:
            st = SegTree(l, r)
            st.max = baskets[l]
            return st
        mid = (l + r) >> 1
        left = self.build(l , mid, baskets)
        right = self.build(mid + 1, r, baskets) 
        st = SegTree(l, r)
        st.left = left
        st.right = right
        left.parent = st
        right.parent = st
        st.max = max(left.max, right.max)
        return st


    def numOfUnplacedFruits(self, fruits: List[int], baskets: List[int]) -> int:
        n = len(baskets)
        head = self.build(0, n - 1, baskets)

        ret = 0
        for f in fruits:
            if head.max < f:
                ret += 1
                continue
            h = head
            while h.max >= f:
                if h.left and h.left.max >= f:
                    h = h.left
                elif h.right:
                    h = h.right
                else:
                    break
            h.max = -1

            while h.parent:
                h.parent.max = -1
                if h.parent.left:
                    h.parent.max = max(h.parent.left.max, h.parent.max)
                if h.parent.right:
                    h.parent.max = max(h.parent.right.max, h.parent.max)
                h = h.parent
        return ret
```