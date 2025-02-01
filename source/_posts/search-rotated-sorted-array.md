---
title: 二分搜索旋转数组问题
date: 2025-02-01 14:22:40
tags:
    - Leetcode
categories: Leetcode
---



## [33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/description/)

难度：中等

### 思路 & 题解

+ 二分搜索，将问题划分为朴素二分搜索和规模更小的子问题

### 实现

```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        n = len(nums)
        i, j = 0, n - 1
        def bisect(l, r):
            if l + 1 >= r:
                return r if nums[r] == target else -1
            mid = (l + r) >> 1
            if nums[mid] >= target:
                return bisect(l, mid)
            return bisect(mid, r)

        def sear(l, r):
            if l + 1 >= r:
                return r if nums[r] == target else -1
            if nums[l] < nums[r]:
                return bisect(l, r)
            mid = (l + r) >> 1
            if nums[mid] == target:
                return mid
            if nums[mid] > nums[l]:
                if nums[l] <= target <= nums[mid]:
                    return bisect(l, mid)
                return sear(mid, r)
            if nums[mid] < nums[r]:
                if nums[mid] <= target <= nums[r]:
                    return bisect(mid, r)
                return sear(l, mid)
        if nums[0] == target:
            return 0
        return sear(0, n - 1)
```



## [81. 搜索旋转排序数组 II](https://leetcode.cn/problems/search-in-rotated-sorted-array-ii/)

难度：中等

### 思路

+ 与上一题相比，数组中的重复元素可能分布在二分左端点和右端点；如果mid取值也与左右端点的取值一致，则无法区分向左搜索还是向右搜索
+ 分别向左、向右搜索，取两者的or值

### 实现

```python
class Solution:
    def search(self, nums: List[int], target: int) -> bool:
        def bi_sect(l, r):
            if l + 1 >= r:
                return target == nums[r]
            mid = (l + r) >> 1
            if nums[mid] >= target:
                return bi_sect(l, mid)
            return bi_sect(mid, r)
        def sear(l, r):
            if l + 1 >= r:
                return target == nums[r]
            mid = (l + r) >> 1
            if nums[mid] == target:
                return True
            ret = False
            if nums[l] <= nums[mid]:
                if nums[l] <= target <= nums[mid]:
                    ret = ret or bi_sect(l, mid)
                ret = ret or sear(mid, r)
            if nums[mid] <= nums[r]:
                if nums[mid] <= target <= nums[r]:
                    ret = ret or bi_sect(mid, r)
                ret = ret or sear(l, mid)
            return ret
        if nums[0] == target:
            return True
        n = len(nums)
        return sear(0, n - 1)
```

### 题解

+ 当二分搜索的left、right、mid端点取值相等时，执行 **right -= 1** 问题即简化为规模更小的子问题