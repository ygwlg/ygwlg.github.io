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

直接使用状态机dp居然超时，先通过一个小trick快速计算10的倍数的状态跳转：无论通过什么样的顺序排列1,2,3,4 一轮一定是跳跃10步。所以可以每10步计算最大值

1. 计算[1,2,3,4]的全排列

2. 对于最后不足10个的元素使用状态机dp

```Python
from math import inf
n = int(input())
a = [int(_) for _ in input().split()]

a.insert(0, 0)

def permute(nums):
    ret = []
    if len(nums) == 1:
        return [nums]
    for i in range(len(nums)):
        new = nums[:]
        item = new.pop(i)
        sub = permute(new)

        for s in sub:
            ret.append([item] + s)
    return ret
     
permute4 = permute([1,2,3,4])
# print(permute4)
def skip10(starti, starts):
    ret = -inf
    for p in permute4:
        ss = starts
        si = starti
        for pp in p:
            si += pp
            ss += a[si]
            if ss < 0:
                ss = -inf
                break
        else:
            ret = max(ret, ss)
    return ret
def solution():
    start = 0
    ret = 0
    for i in range(n // 10):
        ret = skip10(i * 10, ret)
        if ret < 0:
            return -1
    else:
        i = n // 10
        reminds = n - i * 10
        if reminds:
            dp = [[-inf] * 0b10000 for _ in range(reminds + 1)]
            dp[0][0b1111] = ret
            for k in range(1, reminds + 1):
                x = a[i * 10 + k]
                for j in range(0b10000):
                    for m in range(4):
                        if not j & (1 << m):
                            last = j | (1 << m)
                            if k >= m + 1 and dp[k - m - 1][last] >= 0 and dp[k - m - 1][last] + x >= 0:
                                dp[k][j] = dp[k - m - 1][last] + x
            # print(dp)
            ret = max(dp[-1])
            
        
    if ret < 0:
        return -1
    return ret

print(solution())

```

## 5. 小美的数组删除

思路：正向计算MEX列表比较麻烦，可以逆向计算，看作是往一个空列表中逐渐添加元素

```Python
T = int(input())


def solution():

    n, k, x = [int(_) for _ in input().split()]

    a = [int(_) for _ in input().split()]

    MEXs = [0] * n

    vis = set()

    m = 0

    for i in range(n - 1, -1, -1):
        vis.add(a[i])
        while m <= n:
            if m not in vis:
                MEXs[i] = m
                break
            m += 1

    ret = x * n
    
    for i in range(n):
        ret = min(ret, x * i + k * MEXs[i])
    return ret

for _ in range(T):
    print(solution())
```

## 6. 小红的数字删除

复杂模拟。。。不过公司发的卡在了95.45%。[牛客原题](https://www.nowcoder.com/practice/46a73f7cb2ab4a56bdb372b282b23c1e?tpId=376&tqId=10858409&ru=/exam/oj&qru=/ta/15-days-help/question-ranking&sourceUrl=%2Fexam%2Foj)倒可以100%通过。。。

```Python
T = int(input())

for _ in range(T):
    rs = {0: 0, 1: 0, 2: 0}
    total = 0
    count = 0
    count0 = 0
    num = input()
    for item in num:
        item = int(item)
        total += item
        count += 1
        rs[item % 3] += 1

    if total % 3 == 0:
        if rs[0] == count:
            print(count - 1)
        else:
            print(rs[0])

    else:
        r = total % 3
        if not rs[r]:
            print(0)
        else:
            if rs[r] == 1 and int(num[0]) % 3 == r:
                index = 1
                while index < len(num) and num[index] == "0":
                    rs[0] -= 1
                    index += 1
                    count0 += 1
            if 1 + count0 + rs[0] == len(num):
                print(rs[0])
            else:
                print(1 + rs[0])

```

## 7. 小红的数字串

滑动窗口，或者暴力解法也能通过

```Python
import sys

num_str = input()
k = int(input())

res = 0

for i in range(len(num_str)):
    for j in range(i, len(num_str)):
        if int(num_str[i:j+1]) < k:
            res += 1
        else:
            break
print(res)
```

## 8. 小红的01串

对于每个101或010，那么对其进行一次操作即可

```Python
import sys

res,idx =0,0
num_str = input()
n = len(num_str)
while idx <= n-3:
    if num_str[idx:idx+3] in ['101','010']:
        idx += 3
        res += 1
    else:
        idx += 1
print(res)

```

## 9. 小红的爆炸串

滑动窗口：r指针越大越会爆炸，l指针越大越不会爆炸

```Python
n, k = [int(_) for _ in input().split()]

s = input()

l, r = 0, 0

tmp = 0

ret = 0

for l in range(n):
    
    while r < n and tmp < k:
        if r < n - 1 and not s[r] == s[r + 1]:
            tmp += 1
        r += 1
    

    # print(l, r)
    ret += r - l

    if l < n - 1:
        tmp -= 1 if not s[l] == s[l + 1] else 0

print(ret)

```

## 10. 小红拼图

模拟：对于每块拼图判断右方和下方的拼图是否匹配即可

```Python
edges = {
    'W': [1, 1, 0, 1],
    'D': [1, 1, 1, 0],
    'S': [0, 1, 1, 1],
    'A': [1, 0, 1, 1]
}

t = int(input())


def judge(metrix, n, m):
    for i in range(n):
        for j in range(m):
            if i < n - 1:
                if not metrix[i][j] == '*' and not metrix[i + 1][j] == '*':
                    if not edges[metrix[i][j]][2] ^ edges[metrix[i + 1][j]][0]:
                        return False
            if j < m - 1:
                if not metrix[i][j] == '*' and not metrix[i][j + 1] == '*':
                    if not edges[metrix[i][j]][1] ^ edges[metrix[i][j + 1]][3]:
                        return False
    return True

for _ in range(t):
    n, m = [int(_) for _ in input().split()]
    mtx = []
    for __ in range(n):
        mtx.append(input())
    if judge(mtx, n, m):
        print('Yes')
    else:
        print('No')
```

## 11. 小红的奇偶抽取


简单模拟

```Python
import sys

num_str = input()
num1 = '0'
num2 = '0'
for num in num_str:
    if int(num) %2==0:
        num1 += num
    else:
        num2 += num
print(abs(int(num1)-int(num2)))

```

## 12. 游游的整数切割

简单模拟

```Python
import sys

num_str = input()
res = 0
for i in range(len(num_str)-1):
    if (int(num_str[i]) + int(num_str[-1])) %2 ==0:
        res +=1
print(res)
```

## 13. 小红的数位操作

深搜回溯

```Python
n, p = [_ for _ in input().split()]
p = int(p)
reminds = int(n) % p

ret = -1

nums = list(n)
nlen = len(nums)
def dfs(i):
    global reminds, ret
    if i == nlen:
        return 
    dfs(i + 1)

    if not nums[i] == '0':
        nums[i] = chr(ord(nums[i]) - 1)

        old = reminds
        reminds = (reminds - 10 ** (nlen - i - 1)) % p

        if reminds == 0:
            ret = ''.join(nums)
        dfs(i + 1)
        reminds = old
        nums[i] = chr(ord(nums[i]) + 1)

    if not nums[i] == '9':
        nums[i] = chr(ord(nums[i]) + 1)
        old = reminds
        reminds = (reminds + 10 ** (nlen - i - 1)) % p
        
        if reminds == 0:
            ret = ''.join(nums)
        
        dfs(i + 1)
        reminds = old
        
        nums[i] = chr(ord(nums[i]) - 1)

dfs(0)
print(ret)
```

## 14. 小美走公路

简单模拟：顺着编号走或者逆着编号走

```Python
n = int(input())

a = [int(_) for _ in input().split()]

x, y = [int(_) for _ in input().split()]

total = sum(a)

if y < x:
    x, y = y, x

r1 = sum(a[x - 1: y - 1])
r2 = total - r1
print(min(r1, r2))
```

## 15. 小美的排列询问

简单模拟

```Python
n = int(input())
a = [int(_) for _ in input().split()]

x, y = [int(_) for _ in input().split()]

flag = False
for i in range(n - 1):
    if a[i] == x and a[i + 1] == y or a[i] == y and a[i + 1] == x:
        flag = True
        break

if flag:
    print('Yes')
else:
    print('No')
```


## 16. 函数

用islimit来判断是否依旧受到限制，通过判断后续是否大于全1来判断当前位是否要退位

```Python
def f(n):
    islimit = True
    ret = ''
    nlen = len(n)
    for i, item in enumerate(n):
        if not islimit:
            ret += '3'
        else:
            if n[i + 1:] < '1' * (nlen - 1 - i):
                t = chr(ord(item) - 1)
                
                islimit = False
                if item > '3':
                    ret += '3'
                else:
                    # print(item, t)
                    ret += t
            else:
                if item > '3':
                    ret += '3'
                    islimit = False
                else:
dp,                     ret += item
    return ret



T = int(input())

for _ in range(T):
    num = input()
    print(int(f(num)))
```


## 17. 子序列

dp, 记忆化搜索，时间复杂度o(k^2)

```Python
import functools

MOD = 1000000007

n, k = [int(_) for _ in input().split()]
s = input()
counts = {}

for i, item in enumerate(s):
    if item not in counts:
        counts[item] = 0
    counts[item] += 1
ks = list(counts.keys())

kn = len(ks)


@functools.cache
def dfs(i, c):
    if kn - i < c:
        return 0
    if c == 0:
        return 1
    r1 = ((1 << counts[ks[i]]) - 1) * dfs(i + 1, c - 1)
    r2 = dfs(i + 1, c)
    return (r1 + r2) % MOD

print(dfs(0, k))
```

