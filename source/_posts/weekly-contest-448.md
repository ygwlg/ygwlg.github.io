---
title: 力扣周赛448
date: 2025-05-04 16:17:18
tags:
    - Leetcode
---

坏消息：Q3Q4都做不出来；好消息：大家都做不出来，最终125名

[题三-3538. 合并得到最小旅行时间](https://leetcode.cn/problems/merge-operations-for-minimum-travel-time/)

恰好型问题，一开始往01背包方面想，但是01背包解法会将结果作为dp的一环，因此放弃了这一做法。之后就是往堆的方面想，算出来合并动作对结果的影响公式，但是又发现合并的右端点的time值也会受到合并操作的影响，堆就变得很难维护。

用dfs(r, i, pre)来代表遍历到i，还剩r次合并机会，i的合并到的最左端点为pre；函数返回值为剩余旅行时间

这个时候就有状态转移方程

dfs(r, i, pre) = min(dfs(r - (j - i - 1), j, i + 1) + presum[pre] * (position[j] - position[i]))

最终代码

```python
class Solution:
    def minTravelTime(self, l: int, n: int, k: int, position: List[int], time: List[int]) -> int:
        @cache
        def dfs(r, i, pre):
            s = list(accumulate(time, initial=0))

            if i == n - 1:
                return 0 if r == 0 else inf
            
            ret = inf
            for j in range(i + 1, min(i + 1 + r, n - 1) + 1):
                t = s[i + 1] - s[pre]
                ret = min(ret, dfs(r - (j - i - 1), j, i + 1) + (position[j] - position[i]) * t)
            return ret
        return dfs(k, 0 ,0)
```

~~仅仅15行代码，却是理解最困难的一集，去题单里刷两条类似的先。~~

