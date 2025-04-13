---
title: segment-tree
date: 2025-03-10 20:31:45
tags: 
    - Leetcode
    - contest
---

Leetcode周赛440遇到简单线段树没做出来。。。乘机总结一下线段树

线段树指的是用额外的空间为数组创建树结构，减少访问数组的次数

## 朴素线段树

以[307. 区域和检索 - 数组可修改](https://leetcode.cn/problems/range-sum-query-mutable/)为例，暴力算法一般会有两种想法：

+ 第一种是不对数组做额外的处理，每次计算left-right区间和的时候遍历，这样做每次计算的时间复杂度为O(n);

+ 第二种是用前缀和维护，此时每次计算区间和的时间复杂度为O(1)，但是单点更新的时间复杂度为O(n)

上述两种想法可以等同于给数组分片，方法一分片大小为1，方法二分片大小为n；那么也许有一种分片方法可以解题？尝试一下分片大小为

$$
\sqrt{n}
$$

此时计算和单点更新的时间复杂度都为O(n<sup>1.5</sup>),大致为 5*10<sup>6</sup>。也就是说本题居然是可以的。。。

那如果数据规模来到常见的10<sup>5</sup>，那么计算数据量大致为3*10<sup>7</sup>，是不可以通过的。此时可以大致猜到，简单的分片不能够解决此问题。

给一个数组建立一棵树，每次访问的时间复杂度就相当于O(logn)。这样就能够解决问题了。

从开点的方式来看，线段树有两种方法：

### 二分开点

这种方法比较容易想到，毕竟有序数组的查找等常见问题就是中间取点

```python
class SegmentTree:
    def __init__(self, l, r):
        self.l = l
        self.r = r
        self.s = 0
        self.left = None
        self.right = None
        self.parent = None 

class NumArray:
    def build(self, l, r):
        if l == r:
            st = SegmentTree(l, r)
            st.s = self.nums[l]
            return st
        mid = (l + r) >> 1
        left = self.build(l, mid)
        right = self.build(mid + 1, r)
        st = SegmentTree(l, r)
        st.s = left.s + right.s
        st.left= left
        st.right = right
        left.parent = st
        right.parent =st
        return st


    def __init__(self, nums: List[int]):
        self.nums = nums
        self.t = self.build(0, len(nums) - 1)

    def update(self, index: int, val: int) -> None:
        h = self.t
        r = val - self.nums[index]
        self.nums[index] = val
        while h and h.l <= index <= h.r:
            h.s += r

            if h.left and h.left.l <= index <= h.left.r:
                h = h.left

            elif h.right and h.right.l <= index <= h.right.r:
                h = h.right
            else:
                break
                

    def sumRange(self, left: int, right: int) -> int:
        def cal(target):
            h = self.t
            s = 0
            while True:
                if target == h.r:
                    s += h.s
                    break
                if h.left and h.left.l <= target <= h.left.r:
                    h = h.left
                else:
                    s += h.left.s
                    if h.right and h.right.l <= target <= h.right.r:
                        h = h.right
                    else:
                        break
            return s
        if left == 0:
            return cal(right)
        return cal(right) - cal(left - 1)

        
```

### lowbit开点

这种方法代码简洁但是理解少许困难些，连灵神都称为天才。。。

![示意图](https://pic.leetcode.cn/1717549976-yUVqsj-lc307.png)

大致想法：线段树SegmentTree[i]表示i - lowbit(i) + 1 ~ i之间的区间和

~~即便是第二次看完题解写代码，依旧没有自己写出来。。。~~

```python
class NumArray:

    def __init__(self, nums: List[int]):
        self.nums = [0] * (len(nums))
        self.tree = [0] *(1 + len(nums))
        for i, n in enumerate(nums):
            self.update(i, n)
        

    def update(self, index: int, val: int) -> None:
        i = index + 1
        delta = val - self.nums[index]
        self.nums[index] = val
        while i <= len(self.nums):
            self.tree[i] += delta
            i += i & -i

    def sumRange(self, left: int, right: int) -> int:
        def cal(target):
            i = target + 1
            s = 0
            while i > 0:
                s += self.tree[i]
                i -= i & -i
            return s
        return cal(right) - cal(left - 1)


```

## 动态开点线段树

[732. 我的日程安排表 III](https://leetcode.cn/problems/my-calendar-iii/)，本题的输入要求为0 <= startTime < endTime <= 10 ** 9。由于无法获取未来的查询数据，所以无法通过预处理的方式缩小数据规模。如果采取上一节中的预开点方式，会开 10<sup>9</sup>个点，时间和内存肯定都会超限。因此只能懒开点，即lazy线段树

lowbit开点方式的线段树由于自身的性质，父节点和子节点之间的位置关系比较跳跃，所以没法lazy开点。~~也许是迄今为止还没见到~~

相较于朴素线段树，lazy线段树节点多了一个懒惰标记，用于表示对区间内每个节点的操作（变量名一般叫add），当要对区间内的子元素单独操作时再执行开点动作，并把lazy标记传播下去。


~~看完题解扣扣搜搜半天终于调出来，还是得多练。。。~~


```python
N = 10 ** 9
class Node:
    def __init__(self, l, r):
        self.l = l
        self.r = r
        self.left = None
        self.right = None
        self.v = 0
        self.add = 0


class SegTree:
    def __init__(self):
        self.tree = Node(0, N)

    def create_sub(self, t):
        tmid = (t.l + t.r) >> 1
        left = Node(t.l, tmid)
        left.add = t.add
        left.v = t.v

        right = Node(tmid + 1, t.r)
        right.add = t.add
        right.v = t.v

        t.left = left
        t.right = right

        t.add = 0

    def update(self, t, l, r):

        if l <= t.l <= t.r <= r:
            t.v += 1
            t.add += 1
            return

        if not t.left and not t.l == t.r:
            self.create_sub(t)

        tmid = (t.l + t.r) >> 1
        if r > tmid and t.right:
            self.update(t.right, max(tmid + 1, l), r)

        if l <= tmid and t.left:
            self.update(t.left, l, min(r, tmid))

        t.v = max(t.left.v, t.right.v)

    def query(self, t, l, r):
        if l <= t.l <= t.r <= r:
            return t.v

        ret = 0
        tmid = (t.l + t.r) >> 1

        t.right.add += t.add
        t.right.v += t.add

        t.left.add += t.add
        t.left.v += t.add
        t.add = 0
        if r > tmid:
            ret = max(ret, self.query(t.right, l, r))
        if l <= tmid:
            ret = max(ret, self.query(t.left, l, r))
        t.v = max(t.left.v, t.right.v)
        return ret

class MyCalendarThree:

    def __init__(self):
        self.tree = SegTree()
        self.ret = 0

    def book(self, startTime: int, endTime: int) -> int:
        self.tree.update(self.tree.tree, startTime, endTime - 1)
        self.ret = max(self.ret, self.tree.query(self.tree.tree, startTime, endTime - 1))
        return self.ret



```


## 区间更新线段树

上一节提到的朴素线段树只提供了单点更新的方法，如果遍历区间内的每个值执行单点更新，那么区间更新的时间复杂度就来到了O(n*logn)，甚至超过了线性复杂度。那么有没有小于线性复杂度的方法呢？ 同上节解决方案。

