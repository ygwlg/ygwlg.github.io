---
title: leetcode-219
date: 2025-01-29 11:40:37
categories: Leetcode
tags:
    - Leetcode 
---

## [219. 存在重复元素 II](https://leetcode.cn/problems/contains-duplicate-ii/description/)

难度：简单

### 思路 & 题解

定长滑动窗口

### 实现

```python
class Solution:
    def containsNearbyDuplicate(self, nums: List[int], k: int) -> bool:
        n = len(nums)
        num_set = set()
        for i in range(k):
            if i >= n:
                return False
            if nums[i] in num_set:
                return True
            num_set.add(nums[i])

        for i in range(k, n):
            if nums[i] in num_set:
                return True
            num_set.add(nums[i])
            num_set.remove(nums[i - k])
        return False
            
            
```

