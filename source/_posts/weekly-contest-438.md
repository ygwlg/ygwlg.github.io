---
title: Leetcode周赛438
date: 2025-02-23 19:28:23
tags: 
   - Leetcode
   - contest
categories: Leetcode
---

周赛遭遇数论题，强大犹如怪物，拼尽全力无法战胜

## [题一 100579. 判断操作后字符串中的数字是否相等 I](https://leetcode.cn/problems/check-if-digits-are-equal-in-string-after-operations-i/)

简单模拟

## [题二 100576. 提取至多 K 个元素的最大总和](https://leetcode.cn/problems/maximum-sum-with-at-most-k-elements/)

+ 每行选取最大的limit[i]个数

+ 上面选取的数排序，选最大的k个

## [题三 3463. 判断操作后字符串中的数字是否相等 II](https://leetcode.cn/problems/check-if-digits-are-equal-in-string-after-operations-ii/)

思路：两个数字的取值为

$$
\sum\limits_{i=0}\limits^{n-1} C(n-1,i) * s[i]
$$

其中函数C为多项式系数函数，计算表达式为

$$
C(n,i)=\frac{(n)!}{(n-i)!*i!}
$$

于是乎，我志得意满的写下了以下一段代码。这段代码的时间复杂度为o(n+n)，最大时间复杂度为o(n)（其中n是字符串s的长度）。

```python
class Solution:
    def hasSameDigits(self, s: str) -> bool:
        
        n = len(s)
        layer = n - 1
        a = 0
        b = 0
        
        jiechen = [1]
        for i in range(1, layer):
            jiechen.append(jiechen[-1] * i)

        xishu = []
        for i in range(layer):
            if i <= (layer - 1) >> 1:
                
                xishu.append(jiechen[layer - 1]//jiechen[layer - 1 - i]// jiechen[i] % 10)
            else:
                xishu.append(xishu[layer - 1- i])
            a += int(s[i]) * xishu[i]
            b += int(s[i + 1]) * xishu[i]
        
        return a % 10 == b % 10
```

然而，回应我只有冰冷的TLE<span style="color=red">超出时间限制</span>

先学习一下为什么会超时

python语言在处理小整数乘法时直接在寄存器里完成运算，对于大整数乘法，Python使用Karatsuba算法计算。这一算法的时间复杂度为
$$
O(n^{log_{2}^{3}})
$$

大于线性复杂度，更不要说平时编写的时候都当做O(1)...

### 题解一：因式分解+欧拉定理

看题解之前需要先学习一下[费马小定理和欧拉函数](https://en.oi-wiki.org/math/fermat/)

不计算具体值的情况下快速计算MOD值

$$
C(n,i)\%10=\frac{(n)!}{(n-i)!*i!}\%10
$$

假设
$$
\begin{aligned}
&a = n! \\
&b = (n-i)! \\
&c = i!
\end{aligned}
$$
有
$$
C(n,i)\%10 = a * b^{-1} * c^{1-}\%10
$$

再做因式分解，有

$$
C(n,i)\%10 = a_{r} * b_{r}^{-1} * c_{r}^{1-}\% * 2^{...} * 5^{...} % 10
$$

此时b<sub>r</sub>和c<sub>r</sub>与10互质，可以通过欧拉函数计算其逆元

$$
\begin{aligned}
b^{-1} = b_{r}^{f(10) - 1}\\
c^{-1} = c_{r}^{f(10) - 1}
\end{aligned}
$$
其中f(10)=4

1. a<sub>r</sub>的计算方法: 遍历1-n，获取因式分解后的累乘

2. b<sub>r</sub><sup>-1</sup> * b<sub>r</sub><sup>-1</sup> 的计算方法: 求得a<sub>r</sub>[-1]的逆元，反向遍历后因式分解做累乘

3. 2(5)次数的计算方法：遍历时记录前缀和

### 代码实现一

```python
class Solution:
    def hasSameDigits(self, s: str) -> bool:
        s = [int(_) for _ in s]
        n = len(s)
        ret = 0

        e2 = [0]
        e5 = [0]
        fx = [1]
        for i in range(1, n):
            x = i
            c2, c5 = 0, 0
            while x & 1 == 0:
                x >>= 1
                c2 += 1
            while x % 5 == 0:
                x //= 5
                c5 += 1
            fx.append(x * fx[-1] % 10)
            e2.append(e2[-1] + c2)
            e5.append(e5[-1] + c5)
        
        inv = [0] * n
        inv[-1] = pow(fx[-1], 3, 10)
        for i in range(n-1, 0, -1):
            x = i
            while x & 1 == 0:
                x >>= 1
            while x % 5 == 0:
                x //= 5
            inv[i - 1] = inv[i] * x % 10
        
        # print(fx, inv, e2, e5)
        def comb(a, b):
            return fx[a] * inv[b] * inv[a-b] * pow(2, e2[a] - e2[b] - e2[a-b], 10) * pow(5, e5[a]-e5[b]-e5[a-b], 10) % 10
        


        for i in range(n - 1):
            ret += (s[i] - s[i + 1] + 10) * comb(n - 2, i) % 10
        return ret % 10 == 0
```

### 题解二： 扩展 Lucas

[卢卡斯定理与中国剩余定理](https://oi-wiki.org/math/number-theory/lucas/)

```python
class Solution:
    def hasSameDigits(self, s: str) -> bool:
        def lucas(n ,k, p):
            if k == 0:
                return 1
            return comb(n % p, k % p) * lucas(n // p, k //p, p)
        n = len(s)
        ret = 0
        s = [int(_) for _ in s]
        for i in range(n - 1):
            xishu = (5 * lucas(n - 2, i, 2) + 6 * lucas(n - 2, i, 5)) % 10
            ret = (ret + xishu * (s[i] - s[i + 1])) % 10
        return ret == 0
        
```

## [题四-3464. 正方形上的点之间的最大距离](https://leetcode.cn/problems/maximize-the-distance-between-points-on-a-square/)

冷静下来想一想，用逆时针旋转的距离作为绝对距离，每次都二分选取下一个点，即可ac。代码如下

```python
class Solution:
    def maxDistance(self, side: int, points: List[List[int]], k: int) -> int:
        def dis(x, y):
            if y == 0 or x == side:
                return x + y
            if y == side:
                return 3 * side - x
            return 4 * side - y
        points.sort(key = lambda point: dis(point[0], point[1]))
        n = len(points)
        dises = [dis(_[0], _[1]) for _ in points]
        # print(points)

        def check(target):
            for i in range(n):
                flag = True
                r = k - 1
                startx, starty = points[i]
                ni = i
                while r:
                    ni = bisect_left(dises, dises[ni] + target)
                    if ni == n:
                        flag = False
                        break
                    if abs(points[ni][0] - startx) + abs(points[ni][1] - starty) < target:
                        flag = False
                        break
                    r -= 1
                if flag:
                    return True
            return False

        l, r = 0, side + 1
        while l + 1 < r:
            mid = (l + r) >> 1
            if check(mid):
                l = mid
            else:
                r = mid
        return l

```