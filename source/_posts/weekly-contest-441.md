---
title: weekly-contest-441
date: 2025-03-17 20:42:12
categories: Leetcode
tags: 
    - weekly-contest
---

第三条看成差分了，差点没写出来

## [题一-3487. 删除后的最大子数组元素和](https://leetcode.cn/problems/maximum-unique-subarray-sum-after-deletion/)

简单模拟，懒得贴代码了

## [题二-3488. 距离最小相等元素查询](https://leetcode.cn/problems/closest-equal-element-queries/)

1. 把相同的值的索引放到同一个队列里
2. 二分查找

```python
class Solution:
    def solveQueries(self, nums: List[int], queries: List[int]) -> List[int]:
        n = len(nums)
        al = dict()
        for i, num in enumerate(nums):
            if num not in al:
                al[num] = []
            al[num].append(i)
        ret = []
        for q in queries:
            target = nums[q]
            if len(al[target]) == 1:
                ret.append(-1)
            else:
                
                index = bisect_left(al[target], q)
                l, r = index - 1, index + 1
                if index == 0:
                    l = len(al[target]) - 1
                if index == len(al[target]) - 1:
                    r = 0

                t = min(abs(q - al[target][l]), n - abs(q - al[target][l]), abs(q - al[target][r]), n - abs(q - al[target][r]))
                ret.append(t)
        return ret
```

## [题三-3489. 零数组变换 IV](https://leetcode.cn/problems/zero-array-transformation-iv/)

一开始没看到**正好**，给当成差分问题了

+ 对nums中的每个数字跑一遍0-1背包问题

```python
class Solution:
    def minZeroArray(self, nums: List[int], queries: List[List[int]]) -> int:

        

        ret = 0
        m = len(queries)
        for i, num in enumerate(nums):
            if num > 0:
                dp = [0] * (num + 1)
                dp[num] = 1
                
                for j in range(m):
                    l, r, v = queries[j]
                    if l <= i <= r and v <= num:
                        for k in range(num + 1):
                            if k + v < num + 1 and dp[k + v]:
                                dp[k] = 1
                        if dp[0]:
                            
                            ret = max(ret, j + 1)
                            break
                            
                        
                    else:
                        continue

                if not dp[0]:
                    return -1

                
        return ret
```

## [题四-3490. 统计美丽整数的数目](https://leetcode.cn/problems/count-beautiful-numbers/)

短短的题干，大大的疑惑。。。

```python
class Solution:
    def beautifulNumbers(self, l: int, r: int) -> int:
        
        @cache
        def dfs(i, s, m, isnum, limitlow, limithigh):
            if i == n:
                return 1 if s and m % s == 0 and isnum else 0
            
            lo = int(strl[i]) if limitlow else 0
            ho = int(strr[i]) if limithigh else 9

            ret = 0
            for j in range(lo, ho + 1):
                ret += dfs(i + 1, s + j, m * j if isnum or j else 1, isnum or j > 0, limitlow and j == lo, limithigh and j == ho)
            return ret

        strr = str(r)
        strl = '0' * (len(str(r)) - len(str(l))) + str(l)
        print(strl, strr)
        n = len(strr)
        return dfs(0, 0, 1, False, True, True)
```