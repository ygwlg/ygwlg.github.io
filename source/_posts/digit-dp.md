---
title: 数位DP
date: 2025-03-17 21:18:47
tags:
---

用二进制表示集合，例如集合{0,2,3}表示为1101<sub>(2)</sub>

+ 判断元素d是否在集合x里: x >> d & 1 
+ 把元素d加入到集合x中: x| (1 << d)

[2376. 统计特殊整数](https://leetcode.cn/problems/count-special-integers/)

动态规划的状态：

1. i: 遍历到的字符

2. mask: 之前已经加入的元素集合

3. isLimit: 是否收到大小限制

4. isNum: 是否开始选取数字

状态转移方程有:

~~... 先摸了~~

```python
class Solution:
    def countSpecialNumbers(self, n: int) -> int:
        @cache
        def dfs(i, mask, islimit, isnum):
            if i == len(nums):
                return 1 if isnum else 0
            ret = 0
            if not isnum:
                ret += dfs(i + 1, mask, False, False)
            
            l = 0 if isnum else 1
            if islimit:
                r = int(nums[i])
                for j in range(l, r):
                    if not mask >> j & 1:
                        ret += dfs(i + 1, mask | (1 << j), False, True)
                if not mask >> r & 1:
                    ret += dfs(i + 1, mask | (1 <<r), True, True)
            else:
                for j in range(l, 10):
                    if not mask >> j & 1:
                        ret += dfs(i + 1, mask | (1 << j), False, True)

            return ret


        nums = str(n)
        return dfs(0, 0, True, False)
```


上下界数位dp
```python
class Solution:
    def numberOfPowerfulInt(self, start: int, finish: int, limit: int, s: str) -> int:
        @cache
        def dfs(i, limithigh, limitlow):

            if i == len(finish_str):
                return 1
            
            si, ei = int(start_str[i]), int(finish_str[i])
            
            l = 0 if not limitlow else si
            r = 9 if not limithigh else ei

            if i >= len(start_str) - len(s):
                if limithigh:
                    if int(s[i - len(start_str) + len(s)]) > min(ei, limit):
                        return 0
                if limitlow:
                    if int(s[i - len(start_str) + len(s)]) < si:
                        return 0
                return dfs(i + 1, limithigh and int(s[i - len(start_str) + len(s)]) == r, limitlow and int(s[i - len(start_str) + len(s)])==l)
                

            ret = 0
            for j in range(l, min(r, limit) + 1):
                ret += dfs(i + 1, limithigh and j == r, limitlow and j == l)
            return ret

            
        
        finish_str = str(finish)
        start_str = str(start)
        start_str = '0' * (len(finish_str) - len(start_str)) + start_str

        return dfs(0, True, True)

```

进阶版问题：
[3490. 统计美丽整数的数目](https://leetcode.cn/problems/count-beautiful-numbers/)

一开始按照上述模版写的

```python
class Solution:
    def beautifulNumbers(self, l: int, r: int) -> int:
        
        @cache
        def dfs(i, s, m, limitlow, limithigh):
            if i == n:
                return 1 if s and m % s == 0 else 0
            
            lo = int(strl[i]) if limitlow else 0
            ho = int(strr[i]) if limithigh else 9

            ret = 0
            for j in range(lo, ho + 1):
                ret += dfs(i + 1, s + j, m * j, limitlow and j == lo, limithigh and j == ho)
            return ret

        strr = str(r)
        strl = '0' * (len(str(r)) - len(str(l))) + str(l)
        n = len(strr)
        return dfs(0, 0, 1, True, True)
```

结果20,100的用例过不了，输出结果是81。也就是把区间内的每个值都当成了合法的结果，这是因为前导0被计算为乘积了。这时需要一个额外的递归变量来表述当前是否是一个合法的数字

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
        n = len(strr)
        return dfs(0, 0, 1, False, True, True)
```

再看时间复杂度和空间复杂度，即dfs函数搜索空间的大小
