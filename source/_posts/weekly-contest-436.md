---
title: Leetcode周赛436
date: 2025-02-10 08:38:20
categories: Leetcode
tags: 
    - weekly-contest
---

题二毫无思路，直接转战题三；不枉费年后苦练DP，最终豪取207名😋

## [题一：3446. 按对角线进行矩阵排序](https://leetcode.cn/problems/sort-matrix-by-diagonals/)

1. 将所有元素按照对角线存储
2. 排序后填回到二维数组中

## [题二: 3447. 将元素分配给有约束条件的组](https://leetcode.cn/problems/assign-elements-to-groups-with-constraints/)

第一次见调和级数类题，思考了好久错误的方法（二分）最终放弃

### 题解：

对于规模为n的elements中的每个元素，枚举其小于等于mg = max(groups)的所有倍数，其时间复杂度为

$$
\sum\limits_{i=1}\limits^{n} \frac{1}{n}
$$

又有调和级数与ln(n)为等价无穷大 [知乎证明](https://www.zhihu.com/question/423077516)

所以该做法的时间复杂度为

$$
O(mg×ln_n)
$$

### 实现

```python
class Solution:
    def assignElements(self, groups: List[int], elements: List[int]) -> List[int]:
        mg = max(groups)
        targets = [-1] * (mg + 1)
        for i, e in enumerate(elements):
            if e < mg + 1 and targets[e] == -1:
                j = 1
                while j * e <= mg:
                    if targets[j * e] == -1:
                        targets[j * e] = i
                    j += 1

        ret = []
        for g in groups:
            ret.append(targets[g])
        return ret
```

## [题三：3448. 统计可以被最后一个数位整除的子字符串数目](https://leetcode.cn/problems/count-substrings-divisible-by-last-digit/)

模运算DP，其推导公式为:

$$
dp[i][res][(r*10 + letter) \% res]+=dp[i - 1][res][r]
$$

其中：

1. **i** 为字符串s的索引，**letter=int(s[i])**
2. **res** 为除数，**r** 为对**res**的余数
3. 状态转移方程的目标值**dp[i][res][r]**的含义是：以**s[i]**为结尾的数字，对**res**模运算余数等于**r**的子字符串数量


### 实现

```python
class Solution:
    def countSubstrings(self, s: str) -> int:
        t = [3,4,6,7,8,9] # 1,2,5为10的因子，无需运算
        dp = [{res: [0]* res for res in t} for i in range(len(s))]
        ret = 0
        # print(dp[0][1])
        for i, letter in enumerate(s):
            letter = int(letter)
            if i == 0:
                for res in t:
                    dp[i][res][letter % res] += 1
            else:
                for res in t:
                    dp[i][res][letter % res] += 1
                    for r in range(res):
                        dp[i][res][(r * 10 + letter) % res] += dp[i - 1][res][r]
            if letter in [1, 2, 5]:
                ret += i + 1
            elif not letter == 0:
                ret += dp[i][letter][0]
            
        return ret
                    
```

## [题四：3449. 最大化游戏分数的最小值](https://leetcode.cn/problems/maximize-the-minimum-game-score/)

不愧是2800的题，总能有些从未听说过的trick

### 题解

tricks:

1. 整除向上取整转化为向下取整

$$
\lceil \frac{a}{b} \rceil = \lfloor \frac{a-1}{b} \rfloor +1
$$

2. 数组中来来回回的访问方式，可以转化成两两左右横跳的访问方式（下图来自灵神题解）

![图片来自灵神题解](https://pic.leetcode.cn/1739098814-MZuCoO-lc3449-3-c.png)

[参见灵神题解](https://leetcode.cn/problems/maximize-the-minimum-game-score/solutions/3068672/er-fen-da-an-cong-zuo-dao-you-tan-xin-py-3bhl/?slug=maximize-the-minimum-game-score&region=local_v2&tab=description)

### 实现

```python
class Solution:
    def maxScore(self, points: List[int], m: int) -> int:
        def check(target):
            pre = 0
            r = m
            for i, p in enumerate(points):
                k = (target - 1) // p + 1 - pre
                if i == len(points) - 1 and k <= 0:
                    break
                if k < 1:
                    k = 1
                r -= 2 * k - 1
                if r < 0:
                    return False
                pre = k - 1
            return True

        l = 0
        r = m * min(points) + 1
        while l + 1 < r:
            mid = (l + r) >> 1
            if check(mid):
                l = mid
            else:
                r = mid
            
            print(l, r)
        return l
```
