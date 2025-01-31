---
title: leetcode-45
date: 2025-01-27 12:35:38
categories: Leetcode
tags: 
    - Leetcode
---

## [45. 跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/description/)

难度：中等

### 思路

不具体计算从哪个节点跳转到哪个节点，只关心何时需要跳转

1. 自左向右遍历，当前节点到达不了时，需要从先前节点跳转

2. 记录一个堆，遍历时记录最远跳转位置，需要跳转时出堆

### 实现

```python
class Solution:
    def jump(self, nums: List[int]) -> int:
        heap = []
        if len(nums) == 1:
            return 0
        right = 0
        n = len(nums)
        ret = 0
        for i, num in enumerate(nums):
            while right < i:
                right = - heappop(heap)
                ret += 1
            
            heappush(heap, - i - num)
        return ret
```

#### 题解

**不需要维护所有节点的最远跳转距离，只需要维护最大的跳转距离即可**

```python
class Solution:
    def jump(self, nums: List[int]) -> int:
        right = 0
        max_right = 0
        ret = 0
        for i, num in enumerate(nums):
            if i > right:
                right = max_right
                ret += 1
            max_right = max(max_right, i + num)
        return  ret
            
```