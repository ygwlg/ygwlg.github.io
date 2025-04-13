---
title: leetcode周赛445
date: 2025-04-13 14:52:21
tags:
---

最有希望AK的一集，可惜最后一题数位DP没写出；以后不记录1,2两题了


# [题三-3518. 最小回文排列 II](https://leetcode.cn/problems/smallest-palindromic-rearrangement-ii/)

给出的字符串已经是回文串，重新排列这些字符再次组成的回文串都可以通过将回文中心左边的字符排列组成。因此问题可以转化成求取回文中心左边一半字符全排列的第k小的值。

第k小的全排列的计算方式：

从左向右遍历字符串，如果选取了一个字符i，剩余的字符可以组成的组合有

$$
new = \frac{(remaining - 1)!}{c1!c2!...(ci-1)!}
$$

又当前的组合数有

$$
current = \frac{remaining!}{c1!c2!...}
$$

因此可以得到递推关系

$$
new = \frac{ci * current}{remaining}
$$

得到选取某个数会产生的新组合数的关系后，可以从小到大遍历a-z，如果k大于新组合数，那么可以继续遍历当前位置选取的值直到z，否则意味着当前的数值不可向后迭代，因此开始遍历下一个位置的值。

```python
class Solution:
    def smallestPalindrome(self, s: str, k: int) -> str:
        n = len(s)
        if n & 1:
            mid = s[n >> 1]
            l = n >> 1
        else:
            mid = ''
            l = n >> 1

        count_letter = {chr(ord('a') + _): 0 for _ in range(26)}
        for i in range(l):
            count_letter[s[i]] += 1

        total = factorial(l)
        for c, cnt in count_letter.items():
            total //= factorial(cnt)
        if total < k:
            return ''
        left = []
        current = total
        remaining = l
        for _ in range(l):
            for c in count_letter:
                if count_letter[c] == 0:
                    continue
                branch = (count_letter[c] * current) // remaining
                if k > branch:
                    k -= branch
                    continue
                else:
                    left.append(c)
                    count_letter[c] -= 1
                    current = branch
                    remaining -= 1
                    break
        return ''.join(left) + mid + ''.join(left[::-1])
```

[题四-3519. 统计逐位非递减的整数](https://leetcode.cn/problems/count-numbers-with-non-decreasing-digits/)

明明之前练过数位DP，但是还是没a，甚至一开始没往数位dp的思路上靠。。。

一开始写的

``` python
class Solution:
    def countNumbers(self, l: str, r: str, b: int) -> int:
        def cal(num):
            ans = []
            while num:
                ans.append(num % b)
                num = num // b
            return ans[::-1]
        MOD = 10 ** 9 + 7
        bsl = cal(int(l))
        bsh = cal(int(r))
        bsl = [0] * (len(bsh) - len(bsl)) + bsl
        n = len(bsh)
        
        @cache
        def dfs(i, pre, limitlow, limithigh):
            if i == n:
                if limitlow and pre < bsl[-1] or limithigh and pre > bsh[-1]:
                    return 0
                else:
                    return 1
            if limitlow:
                lo = max(int(bsl[i]), pre)
            else:
                lo = pre

            ho = int(bsh[i]) if limithigh else b - 1
            
            
            ret = 0
            for j in range(lo, ho + 1):
                ret += dfs(i + 1, j, limitlow and j == lo, limithigh and j == ho)
            return ret % MOD
        return dfs(0, 0, True, True) % MOD
        

```

在用例 "5581", "64204", 6 中没通过。。。

这是把遍历的起始节点的计算逻辑与下一轮递归的参数计算逻辑搞混了，无论如何都有 lo = bsl[i] if limitlow else 0，这是由limitlow的含义决定的，与下一轮递归参数的起点无关。

```python
class Solution:
    def countNumbers(self, l: str, r: str, b: int) -> int:
        def cal(num):
            ans = []
            while num:
                ans.append(num % b)
                num = num // b
            return ans[::-1]
        MOD = 10 ** 9 + 7
        bsl = cal(int(l))
        bsh = cal(int(r))
        bsl = [0] * (len(bsh) - len(bsl)) + bsl
        n = len(bsh)
        
        @cache
        def dfs(i, pre, limitlow, limithigh):
            if i == n:
                return 1
            lo = bsl[i] if limitlow else 0

            ho = int(bsh[i]) if limithigh else b - 1
            
            
            ret = 0
            for j in range(max(lo, pre), ho + 1):
                ret += dfs(i + 1, j, limitlow and j == lo, limithigh and j == ho)
            return ret % MOD
        return dfs(0, 0, True, True) % MOD
        
```