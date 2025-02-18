---
title: 0-1背包问题
date: 2025-02-07 08:32:12
categories: Leetcode
tags:
    - Leetcode
    - DP
---

## 介绍

0-1背包问题指的是：从n件物品中选取若干个物品放到空间为W的背包里，每个物品的体积为 M<sub>i</sub> ，其对应的价值为 P<sub>i</sub>。求取最大价值

$$
\begin{flalign*}
    &\sum\limits_{i=0}\limits^{n-1} \Phi(i)×M_i \leq W\\
    &maximize(\sum\limits_{i=0}\limits^{n-1} \Phi(i)×P_i)
\end{flalign*}
$$

常见问题：在多个约束条件下选取子集(子序列)

## 动态规划解法

+ 目标值作为状态转移方程的返回值
+ 状态转移方程最外层的维度为物品数组的规模
+ 为每条约束设定一个状态转移方程的维度

对于最朴素的0-1背包问题，状态转移方程为

$$
dp_{i,j} = \begin{cases}
 \max(dp_{i-1,j}, dp_{i-1, j-W_i}+P_i),
 & j \geq W_i
 \\ dp_{i-1,j}, & j < W_i
 \end{cases}
$$

### [Leetcode-2915. 和为目标值的最长子序列的长度](https://leetcode.cn/problems/length-of-the-longest-subsequence-that-sums-to-target)

+ 思路: 每个数字的权值为1，容器大小为target，状态转移方程目标值为最大权值

+ 代码实现:

```python
class Solution:
    def lengthOfLongestSubsequence(self, nums: List[int], target: int) -> int:
        n = len(nums)
        dp = [[-1] * (1 + target) for _ in range(n)]
        for i in range(n):
            dp[i][0] = 0
        for i in range(n):
            for j in range(1, target + 1):
                if i == 0:
                    if nums[i] == j:
                        dp[i][j] = 1
                else:
                    if j >= nums[i] and dp[i - 1][j - nums[i]] >= 0:
                        dp[i][j] = max(dp[i - 1][j], 1 + dp[i - 1][j - nums[i]])
                    else:
                        dp[i][j] = dp[i - 1][j]
        # print(dp)
        if dp[-1][-1] == 0:
            return -1
        return dp[-1][-1]
```

+ 优化1：拓展dp数组，不额外处理边界值i=0

```python
# i = 0时代表不选取数组
class Solution:
    def lengthOfLongestSubsequence(self, nums: List[int], target: int) -> int:
        n = len(nums)
        dp = [[-inf] * (target + 1) for _ in range(n + 1)]
        for i in range(n + 1):
            dp[i][0] = 0
        for i in range(1, n + 1):
            for j in range(target + 1):
                if j - nums[i - 1] >= 0:
                    dp[i][j] = max(dp[i - 1][j], 1 + dp[i - 1][j - nums[i - 1]])
                else:
                    dp[i][j] = dp[i - 1][j]

        return dp[-1][-1] if dp[-1][-1] > 0 else -1
```

+ 优化2：dp[i]的计算只依赖dp[i-1]，因此无需维护所有状态转移方程的值

```python
# 无需维护i，只维护j即可 
class Solution:
    def lengthOfLongestSubsequence(self, nums: List[int], target: int) -> int:
        n = len(nums)
        dp = [-inf] * (target + 1)
        dp[0] = 0
        for i in range(1, n + 1):
            for j in range(target, -1, -1):  # 正序遍历会导致同一个i迭代的值相互依赖
                if j - nums[i - 1] >= 0:
                    dp[j] = max(dp[j], 1 + dp[j - nums[i - 1]])

        return dp[-1] if dp[-1] > 0 else -1
```

### [Leetcode-1049. 最后一块石头的重量 II](https://leetcode.cn/problems/last-stone-weight-ii/)

**从结果考虑，无论一个石头被粉碎多少次，最终其权值对目标值的影响要么为+要么为-**。因此**该问题等价为将这堆石头分成差别尽量少的两堆**。~~要刷多少题才能想出来这么巧妙的解法。。~~

```python
class Solution:
    def lastStoneWeightII(self, stones: List[int]) -> int:
        s = sum(stones)
        n = len(stones)
        dp = [[0] * (s + 1) for _ in range(n)]
        for i in range(n):
            for j in range(s + 1):
                if i == 0:
                    if j == stones[i]:
                        dp[i][j] = 1
                else:
                    dp[i][j] = dp[i - 1][j]
                    if j - stones[i] >= 0 and dp[i - 1][j - stones[i]]:
                        dp[i][j] = dp[i - 1][j - stones[i]] 
        ret = s
        for j in range(s + 1):
            if dp[-1][j]:
                ret = min(ret, abs(s - j - j))
        return ret
```

### [Leetcode-879. 盈利计划](https://leetcode.cn/problems/profitable-schemes/)

+ 状态转移方程的目标值设定为**特定利润值对应的计划数目**时，时间复杂度超过本题时间限制 $group.length × n × totalProfit = group.length^2 × n × profit$
+ 目标值需要设定为 **大于等于该利润的计划数目**

```python
class Solution:
    def profitableSchemes(self, n: int, minProfit: int, group: List[int], profit: List[int]) -> int:
        m = len(group)
        p = minProfit
        M = 10 ** 9 + 7
        dp = [[[0] * (p + 1) for __ in range(n + 1)] for _ in range(m + 1)]
        dp[0][0][0] = 1
        for i in range(1, m + 1):
            for j in range(n + 1):
                for k in range(p + 1):
                    gr, pr = group[i - 1], profit[i - 1]
                    if j >= gr:
                        dp[i][j][k] = (dp[i - 1][j][k] + dp[i - 1][j - gr][max(0, k - pr)]) % M  # 如果k < pr，选取当前值已经满足k，因此前i-1项只需选取利润≥0的值
                    else:
                        dp[i][j][k] = dp[i - 1][j][k]
        # print(dp)
        return sum(dp[-1][j][-1] for j in range(n + 1)) % M
```

