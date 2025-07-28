---
title: 7月编程考试练习
date: 2025-07-19 20:12:19
tags: 
    - nowcoder
---


公司为了考察大伙的算法水平，特地搞了个算法考试。考试的难度很低，但是练习题挺多好题。乘着考试时间还没过，整理一下50道练习题。~~不少都是互联网大厂笔试题，可不能浪费公司的资源~~

## 1. 小苯的数字权值

思路：

对于x，如果x是某个质数p的k次方，那么就需要拆成k个p相乘，最终结果等于k * wt(p) = 2 * k；如果x的为多个质数p1, p2, p3的k1,k2,k3次方乘积，那么k就不需要拆分。其最大值为(k1 + 1)*(k2 + 1)*(k3 + 1)...

确定了上述贪心策略，就需要快速计算x的质数次数乘积表示。说是快速计算是因为T的最大值有1e4，x的最大值为2*1e5，也就是对于每一次询问，算法不能超过对数复杂度。

计算的x的质数次数乘积表示的算法思路如下：
1. 记录范围内所有质数
2. 对于每个质数，计算x的次数

如何计算范围内所有的质数？

1. 标记范围内的所有整数为质数
2. 从最小的质数2开始遍历迭代因子，将迭代因子的整数倍数字标记为非质数

假设范围为1~n，那么上述算法的计算规模为n(1/2 + 1/3 + 1/5 ...)，这样的计算规模和nlogn差不多，2*1e5是不会超时的。

接下来进行实现

```Python

def collect(n):
    """统计范围内的所有质数"""
    is_zhishu = [True] * (n + 1)
    is_zhishu[0] = False
    is_zhishu[1] = False
    for i in range(2, n + 1):
        if is_zhishu[i]:
            for step in range(i * i, n + 1, i):
                is_zhishu[step] = False
    zhishus = []
    for i, judge in enumerate(is_zhishu):
        if judge:
            zhishus.append(i)
    return zhishus

def cal(x, zhishus):
    yinzis = {}
    for zs in zhishus:
        if x < zs:
            break
        while x:
            if x % zs == 0:
                if zs not in yinzis:
                    yinzis[zs] = 0
                yinzis[zs] += 1
                x //= zs
            else:
                break
    return yinzis

zhishus = collect(2 * (10 ** 5) + 1)
T = int(input())

for _ in range(T):
    x = int(input())
    yinzis = cal(x, zhishus)
    
    yinzi_nums = list(yinzis.values())

    if len(yinzi_nums) < 2:
        print(2 * yinzi_nums[0])
    else:
        ret = 1
        for yn in yinzi_nums:
            ret *= yn + 1
        print(ret)

```

## 2. 小红的子序列逆序对

思路：

题目要求计算所有子序列的逆序对的数量之和，这肯定不能深搜所有子序列再统计逆序对。可以考虑数组中的每个逆序对对于最终计算结果的影响：除了这一逆序对两个元素之外还有n-2个元素，因此对计算结果的影响是2^(n-2)。最终的计算结果也就是逆序对个数*2^(n-1)。

如何求得数组的逆序对个数呢？对于每个元素，统计此前比该元素大的元素数量即可，以下通过维护一个临时数组实现。

```Python
import bisect

n = int(input())

ret = 0

array = [int(_) for _ in input().split()]

tmp = []

for a in array:
    bi = bisect.bisect_right(tmp, a)
    ret = (ret + len(tmp) - bi) % (10 ** 9 + 7)
    tmp.insert(bi, a)
    # print(tmp)
print(ret * 2 ** (n - 2) % (10 ** 9 + 7))
```

## 3. 小美的彩带

尚未解出

## 4. 小美和大富翁

直接使用状态机dp居然超时，先通过一个小trick快速计算10的倍数的状态跳转：无论通过什么样的顺序排列1,2,3,4 一轮一定是跳跃10步。

```Python

```