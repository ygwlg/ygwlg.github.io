---
title: leetcode-47
date: 2025-02-06 08:52:13
categories: Leetcode
tags:
    - Leetcode
---


## [47. 全排列 II](https://leetcode.cn/problems/permutations-ii/)

难度：中等

### 思路

回溯 + 去重


### 实现

```python
class Solution:
    def permuteUnique(self, nums: List[int]) -> List[List[int]]:
        walked = [False] * len(nums)
        def dfs(index):
            if index == len(nums):
                if tmp not in ret:
                    ret.append(tmp[:])
            for i, w, num in zip(range(len(nums)), walked, nums):
                if not w:
                    walked[i] = True
                    tmp.append(num)
                    dfs(index + 1)
                    walked[i] = False
                    tmp.pop(-1)
        tmp = []
        ret = []
        dfs(0)
        return ret
```


#### 题解

相同的数字限制选用的顺序，只允许从前向后

+ 对nums排序
+ 如果连续的两个数字相等，但是前面的没有选取，则后面的也不可以选取

```python
class Solution:
    def permuteUnique(self, nums: List[int]) -> List[List[int]]:
        nums.sort()
        walked = [False] * len(nums)
        def dfs(index):
            if index == len(nums):
                # if tmp not in ret:
                ret.append(tmp[:])
            for i, w, num in zip(range(len(nums)), walked, nums):
                if w or i > 0 and nums[i] == nums[i - 1] and not walked[i - 1]:
                    continue

                walked[i] = True
                tmp.append(num)
                dfs(index + 1)
                walked[i] = False
                tmp.pop(-1)

        tmp = []
        ret = []
        dfs(0)
        return ret
```

