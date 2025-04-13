---
title: 力扣周赛439
date: 2025-03-02 17:00:07
tags: 
    - Leetcode
    - contest
---

猜到了dp，猜不出递推关系

## [题一-3471. 找出最大的几近缺失整数](https://leetcode.cn/problems/find-the-largest-almost-missing-integer/)

+ k=1或k=nums.len时，结果为nums.max
+ 否则结果为最左边元素，最右边元素或者-1

```python
class Solution:
    def largestInteger(self, nums: List[int], k: int) -> int:
        a = nums[0]
        b = nums[-1]
        ret = -1
        if k == len(nums):
            return max(nums)
        if k == 1:
            c = Counter(nums)
            r = -1
            for k, v in c.items():
                if v == 1:
                    r = max(r, k)
            return r
            
        if nums.count(a) == 1:
            ret = max(ret, a)
        if nums.count(b) == 1:
            ret = max(ret, b)
        return ret
```

## [题二-3472. 至多 K 次操作后的最长回文子序列](https://leetcode.cn/problems/longest-palindromic-subsequence-after-at-most-k-operations/)

一开始看成回文子串，用填充字符串、延伸臂长去做。。。

回文子序列一般都是dp

```python
class Solution:
    def longestPalindromicSubsequence(self, s: str, k: int) -> int:

        @cache
        def diff(a, b):
            return min(abs(ord(a) - ord(b)), 26 - abs(ord(a) - ord(b)))
        
        @cache
        def dfs(start, end, r):
            if start == end:
                return 1
            if end == start + 1:
                if diff(s[start], s[end]) <= r:
                    return 2
                return 1
            ret = max(dfs(start + 1, end, r), dfs(start, end - 1, r))
            
            de = diff(s[start], s[end])
            if de <= r:
                ret = max(ret, 2 + dfs(start + 1, end - 1, r - de))
            return ret
        return dfs(0, len(s) - 1, k)
```

## [题三-3473. 长度至少为 M 的 K 个子数组之和](https://leetcode.cn/problems/sum-of-k-subarrays-with-length-at-least-m/description/)

大概能猜到是dp，但是想不到状态转移关系。。。~~说好100条动态规划入门呢~~

状态转移方程

```python
class Solution:
    def maxSum(self, nums: List[int], k: int, m: int) -> int:
        pres = [0]
        s = 0
        n = len(nums)
        for num in nums:
            s += num
            pres.append(s)
        # @cache
        # def dfs(i, j):
        #     if i == 0:
        #         return 0
        #     if j + 1 < i * m:
        #         return -inf
        #     ret = dfs(i, j - 1)
        #     for l in range((i - 1) * m, j - m + 2):
        #         ret = max(ret, dfs(i - 1, l - 1) + pres[j + 1] - pres[l])

        #     return ret
        dp = [[-inf] * n for _ in range(k + 1)]

        for i in range(0, k + 1):
            for j in range(n):
                if i == 0:
                    dp[i][j] = 0
                elif j + 1 < i * m:
                    dp[i][j] = -inf
                else:
                    dp[i][j] = dp[i][j - 1]
                    for l in range((i - 1) * m, j - m + 2):
                        dp[i][j] = max(dp[i][j], dp[i - 1][l - 1] + pres[j + 1] - pres[l])
            

        return dp[-1][-1]


```

上面的时间复杂度为
$$
O(n^2*k)
$$

显然是会TLE的，~~从题解~~观察到遍历l的过程中的
```python
for l in range((i - 1) * m, j - m + 2):
    dp[i][j] = max(dp[i][j], dp[i - 1][l - 1] + pres[j + 1] - pres[l])
```
等于
```python
t = pres[j + 1]
for l in range((i - 1) * m, j - m + 2):
    dp[i][j] = t + max(dp[i][j], dp[i - 1][l - 1]  - pres[l])
```

提取出t之后剩余要求遍历计算最大值的表达式仅与l相关的函数（单次i循环内），为此可以在j循环内记录函数式的最大值

由于l的左端点固定，所以可以边遍历边维护最大值

```python
class Solution:
    def maxSum(self, nums: List[int], k: int, m: int) -> int:
        pres = [0]
        s = 0
        n = len(nums)
        for num in nums:
            s += num
            pres.append(s)

        dp = [[-inf] * n for _ in range(k + 1)]

        for i in range(0, k + 1):
            mx = -inf
            for j in range(n):
                if i == 0:
                    dp[i][j] = 0
                elif j + 1 < i * m:
                    dp[i][j] = -inf
                else:
                    dp[i][j] = dp[i][j - 1]
                    mx = max(mx, dp[i - 1][j - m] - pres[j - m + 1])
                    dp[i][j] = max(dp[i][j], mx + pres[j + 1])
            

        return dp[-1][-1]


```

## [题四-3474. 字典序最小的生成字符串](https://leetcode.cn/problems/lexicographically-smallest-generated-string/)

根本没思路，没敢动手。。。