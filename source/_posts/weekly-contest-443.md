---
title: weekly-contest-443
date: 2025-03-30 13:43:47
tags: 
    - weekly-contest
---

第三题纠结半天，最终解决仍是TLE。~~哎~还是练得太少了~~

## [题一-3502. 到达每个位置的最小费用](https://leetcode.cn/problems/minimum-cost-to-reach-every-position/)

计算数组前缀最小值

```python
class Solution:
    def minCosts(self, cost: List[int]) -> List[int]:
        t = inf
        ret = []
        for c in cost:
            t = min(t, c)
            ret.append(t)
        return ret
```

## [题二-3503. 子字符串连接后的最长回文串 I](https://leetcode.cn/problems/longest-palindrome-after-substring-concatenation-i/)

写个暴力的五重循环。。。

```python
class Solution:
    def longestPalindrome(self, s: str, t: str) -> int:
        ret = 1
        for i1 in range(len(s)):
            for j1 in range(i1, len(s) + 1):
                for i2 in range(len(t)):
                    for j2 in range(len(t) + 1):
                        target = s[i1: j1] + t[i2: j2]
                        l, r = 0, len(target) - 1
                        flag = True
                        while l < r:
                            if not target[l] == target[r]:
                                flag = False
                                break
                            l += 1
                            r -= 1
                        if flag:
                            ret = max(len(target), ret)
        return ret
```

## [题三-3504. 子字符串连接后的最长回文串 II](https://leetcode.cn/problems/longest-palindrome-after-substring-concatenation-ii/)

写了个dfs，最终结果是。。。TLE

