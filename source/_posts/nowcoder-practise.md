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

如何计算范围内所有的质数？（埃氏筛）

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
                    ret += item
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

## 18. 散落的金币

尚未解出

## 19. 相遇

思路：每个向左走的人会创到其左边向右走的人

实现：
```Python
n = int(input())

ps = []
for _n in range(n):
    xi, ai = [int(_) for _ in input().split()]
    ps.append((xi, ai))
ps.sort()

pre1 = 0
ret = 0
for xi, ai in ps:
    if ai == 1:
        pre1 += 1
    else:
        ret += pre1
print(ret)
```

## 20. 小红的red字符串

思路：r,e,d三种字符数一致，相当于 `r-e = r-d = 0`
记录前缀字符串每个位置的差值种类的数量；有相同差值的代表子字符串的三种字符相等，对其求和即为结果

```Python
s = input()
rme, rmd = 0, 0

pres = {(0, 0): 1}

ret = 0

for letter in s:
    if letter == 'r':
        rme += 1
        rmd += 1
    elif letter == 'e':
        rme -= 1
    else:
        rmd -= 1
    
    if (rme, rmd) in pres:
        ret += pres[(rme, rmd)]
    
    if (rme, rmd) not in pres:
        pres[(rme, rmd)] = 0
    pres[(rme, rmd)] += 1
print(ret)
```

## 21. 小红的位运算

从高位到低位遍历，逐渐减少候选集，如果当前遍历位能够满足有k个候选，则选取当前位并更新候选集

```Python
n, k = [int(_) for _ in input().split()]
a = [int(_) for _ in input().split()]
ma = max(a)

mb = ma.bit_length()

for i in range(mb, -1, -1):
    new_a = []
    for num in a:
        if (num >> i) & 1:
            new_a.append(num)
    if len(new_a) >= k:
        a = new_a
ret = a[0]
for i in range(1, len(a)):
    ret &= a[i]
print(ret)
```

## 22. 小红的v三元组

思路：对于每个中间元素，寻找左右两边大于该元素的相等元素组；这是个单点更新的区间和问题，用线段树解决

具体步骤：

1. 通过离散化将数据范围缩小至10^5,同时保留相对的大小关系

2. 预处理所有数出现的频率count

3. 建立FenWick线段树

4. 自左向右遍历数组，先统计大于x的区间和，再单点更新线段树的x值

```Python

import sys
data = sys.stdin.read().split()
n = int(data[0])
a = list(map(int, data[1:1 + n]))

# 1. 离散化
unique_vals = sorted(set(a))
comp_map = {}
for idx, val in enumerate(unique_vals):
    comp_map[val] = idx
for i in range(n):
    a[i] = comp_map[a[i]] + 1

M = len(unique_vals)
# 2. 预处理count
counts = [0] * (M + 1)
for i, ai in enumerate(a):
    counts[ai] += 1

# 3. 建立线段树
class FenwickTree:
    def __init__(self, n: int):
        self.tree = [0] * (n + 1)  # 使用下标 1 到 n

    # a[i] 增加 val
    # 1 <= i <= n
    # 时间复杂度 O(log n)
    def update(self, i: int, val: int) -> None:
        t = self.tree
        while i < len(t):
            t[i] += val
            i += i & -i

    # 计算前缀和 a[1] + ... + a[i]
    # 1 <= i <= n
    # 时间复杂度 O(log n)
    def pre(self, i: int) -> int:
        t = self.tree
        res = 0
        while i > 0:
            res += t[i]
            i -= i & -i
        return res

    # 计算区间和 a[l] + ... + a[r]
    # 1 <= l <= r <= n
    # 时间复杂度 O(log n)
    def query(self, l: int, r: int) -> int:
        if r < l:
            return 0
        return self.pre(r) - self.pre(l - 1)

# 作者：灵茶山艾府
# 链接：https://leetcode.cn/discuss/post/mOr1u6/
# 来源：力扣（LeetCode）
# 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

ret = 0
t = FenwickTree(M)
pre_counts = [0] * (M + 1)
for ai in a:
    if ai < M:
        ret += t.query(ai + 1, M)
    old_cmp = counts[ai] * pre_counts[ai]
    pre_counts[ai] += 1
    counts[ai] -= 1
    new_cmp = counts[ai] * pre_counts[ai]
    t.update(ai, new_cmp - old_cmp)
print(ret)

```

## 23. Monica的树

思路：每个结点的取值为其父节点取值 + 1（R）或 - 1（B），通过dp求解

步骤：先深搜找到每个节点的parent；再通过记忆化搜索求得结果

** 有坑：默认最大递归深度需要设置 **

~~ 到底还是被leetcode惯的，牛客需要自己设置递归深度 ~~

```Python
import sys
sys.setrecursionlimit(10 ** 9)
from functools import cache
n = int(input())
colors = input()

edges = {}
for _ in range(n - 1):
    a, b = [int(_) for _ in input().split()]
    
    if a not in edges:
        edges[a] = []
    if b not in edges:
        edges[b] = []
    edges[a].append(b)
    edges[b].append(a)

parents = [-1] * (n + 1)
parents[1] = 0
def find_p(node, parent):
    for child in edges[node]:
        if not child == parent and parents[child] < 0:
            parents[child] = node
            find_p(child, node)

find_p(1, 0)
# print(parents)
@cache
def dfs(node):
    if node == 0:
        return 0
    return dfs(parents[node]) + (1 if colors[node - 1] == 'R' else -1)

ret = 0
for i in range(1, n + 1):
    ret += abs(dfs(i))
print(ret)

```


## 24. 小欧的括号操作

思路：贪心，栈；自左向右遍历，如果是 ` ( `则放入栈中，如果是 ` ) ` 则从栈顶弹出尽可能多的 ` ( ` ，并将整个弹出的元素合并成 ` ( `， 如果栈顶一个 ` ( `都没有，意味着不能够合并，只能把 ` ) ` 压栈。

实现：

``` Python

braces = input()

stack = []

for b in braces:
    if b == '(':
        stack.append(b)
    else:
        flag = False
        while stack and stack[-1] == '(':
            flag = True
            stack.pop(-1)
        if flag:
            stack.append('(')
        else:
            stack.append(')')
print(len(stack))
```

## 25. 小欧的选数乘积


思路：数组去重后贪心，每次都选最大值


实现：
```Python
x, y = [int(_) for _ in input().split()]
n = int(input())
a = [int(_) for _ in input().split()]

a = list(set(a))
a.sort(reverse=True)

def cal():
    global x, y, n, a
    ret = 0
    while x < y and a:
        x *= a.pop(0)
        ret += 1
    if x >= y:
        return ret
    return -1

print(cal())
```

## 26. 
