---
title: Leetcode周赛444
date: 2025-04-06 19:45:52
tags:
---

第三题是困难就做不出来。。。实力无提升

周三晚上打算看错题的时候发现rejudge了，第二题没过从187名掉到700多名。。。真是难受

[Q1. 移除最小数对使数组有序 I](https://leetcode.cn/contest/weekly-contest-444/problems/minimum-pair-removal-to-sort-array-i/)

数据规模只有50，暴力模拟

```python
class Solution:
    def minimumPairRemoval(self, nums: List[int]) -> int:
        def check():
            for j in range(len(nums) - 1):
                if nums[j] > nums[j + 1]:
                    return False
            return True

        if len(nums) == 1:
            return 0
        ret = 0
        while not check():
            tmps = []
            m, mi = inf, -1
            for i in range(len(nums) - 1):
                tmps.append(nums[i] + nums[i + 1])
                if tmps[-1] < m:
                    m = tmps[-1]
                    mi = i
            nums[mi] = m
            nums.pop(mi + 1)
            ret += 1
        return ret
```

[Q2. 设计路由器](https://leetcode.cn/contest/weekly-contest-444/problems/implement-router/)

字典散列存储、列表二分查找

```python
class Router:

    def __init__(self, memoryLimit: int):
        self.dis2mem = {}
        self.dis2ts = {}
        self.count = 0
        self.arr = []
        self.memoryLimit = memoryLimit
        

    def addPacket(self, source: int, destination: int, timestamp: int) -> bool:
        if destination not in self.dis2mem:
            self.dis2mem[destination] = set()
            self.dis2ts[destination] = list()
            
        if self.dis2mem[destination] and (source, destination, timestamp) in self.dis2mem[destination]:
            return False
            
        
        self.arr.append((source, destination, timestamp))
        self.dis2mem[destination].add((source, destination, timestamp))
        self.dis2ts[destination].append(timestamp)
        self.count += 1

        if self.count > self.memoryLimit:
            s, d, t = self.arr.pop(0)
            self.dis2mem[d].remove((s,d,t))
            self.dis2ts[d].pop(0)
            self.count -= 1
        return True
        

    def forwardPacket(self) -> List[int]:
        if self.arr:
            s, d, t = self.arr.pop(0)
            self.dis2mem[d].remove((s,d,t))
            self.dis2ts[d].pop(0)
            self.count -= 1
            return [s, d, t]
        return []
        

    def getCount(self, destination: int, startTime: int, endTime: int) -> int:
        l = bisect_left(self.dis2ts[destination], startTime)
        r = bisect_right(self.dis2ts[destination], endTime)
        return r - l
        
```

[Q3. 最大化交错和为 K 的子序列乘积](https://leetcode.cn/contest/weekly-contest-444/problems/maximum-product-of-subsequences-with-an-alternating-sum-equal-to-k/)

费劲巴拉往DP的方向思考，没写出来，最后写了个回溯给TLE了，如下是回溯的代码。一开始还给回溯加了个@cache，这样做是错误的，因为在cache里没有记录回溯值的状态，所以有回溯值不同但是参数相同而跳过计算的情况。

```python
class Solution:
    def maxProduct(self, nums: List[int], k: int, limit: int) -> int:
        n = len(nums)
        ret = -1
        
        def dfs(i, kk, counts):
            nonlocal mul, ret

            if kk == 0 and mul <= limit and counts:
                ret = max(ret, mul)
                
            if i == n:
                return 


            if counts & 1:
                mul_last = mul
                mul *= nums[i]
                a = dfs(i + 1, kk + nums[i], counts + 1)
                mul = mul_last
                
                b = dfs(i + 1, kk, counts)
            else:
                mul_last = mul
                mul *= nums[i]
                a = dfs(i + 1, kk - nums[i], counts + 1)
                mul = mul_last
                
                b = dfs(i + 1, kk, counts)

        mul = 1
        dfs(0, k, 0)
        return ret
```

上述算法的时间和空间复杂度均为 

$$
O(n^2 * 2^n)
$$

远远超过10<sup>6</sup>的上限

可以通过limit限制数据范围

由于得知limit最大值是5000，又有数组规模小于150且其中元素 ∈[1， 12]，因此可以通过打表得知乘积的可取集合只有不到400。又由于limit最大值为5000，所以可以得到大于1的数最多有L=log<sub>2</sub>limit个，约有12个，又可以得到交错和的绝对值<2L + (n - L) <= 162。所以这一题利用好limit的限制，可以把交错和sum和元素乘积mul作为深搜的维度。

除了这两个维度之外，还需要：

1. i 用于遍历数组

2. odd，基偶性，（比用count记录元素好一些）

3. empty 当前是否有选取元素加入到子序列（比count好一些，和上面的odd参数组合只要记录4个状态，而count最多有150种状态）

一开始按照灵神上述题解写的代码

```python
class Solution:
    def maxProduct(self, nums: List[int], k: int, limit: int) -> int:
        
        ret = -1
        n = len(nums)
        @cache
        def dfs(i, s, m, odd, empty):
            nonlocal ret

            if m > limit:
                return
            if s == k:
                if not empty:
                    ret = max(ret, m)
             
            if i == n:
                return
                
                
            dfs(i + 1, (s + nums[i]) if not odd else (s - nums[i]), m * nums[i], not odd, False)
            dfs(i + 1, s, m, odd, empty)
        dfs(0, 0, 1, False, True)
        return ret
```

[10,10,9,0], 1, 20的用例失败了，这是因为前面计算的子序列积超过了limit但是后面有0，将之前的超过limit的值变成了合法值。

这个时候优化

```python
if m > limit:
    m = -1
```

但是报错内存超限，这是因为cache会开辟内存存储函数结果，所以再次优化

```python
if m > limit or m < 0:
    m = -1
```

但是对于用例

```python
[12,9,8,9,0,0,6,12,7,10,12,2,11,3,8,6,5,11,7,6,7,7,3,9,4,1,5,8,6,1,12,1,7,0,1,12,0,2,2,3,7,5,10,5,6,3,4,8,10,6,0,7,10,2,9,6,5,9,2,7,12,8,4,9,3,11,7,5,7,12,7,2,5,3,0,5,7,2,6,0,10,6,11,2,5,6,9,2,7,2,1,12,12,2,9,1,5,8,11,7]
5704
5000
```
这说明还需要减枝，此时又有元素值不超过12，因此根据元素值再减枝

```python
mx = max(nums)
if k >= 0:
    if k > (n + 1) // 2 * mx:  # k 太大
        return -1
else:
    if -k > n // 2 * mx:  # k 太小（绝对值太大）
        return -1
```


最终的代码

```python
class Solution:
    def maxProduct(self, nums: List[int], k: int, limit: int) -> int:
        
        ret = -1
        n = len(nums)
        mx = max(nums)
        if k >= 0:
            if k > (n + 1) // 2 * mx:  # k 太大
                return -1
        else:
            if -k > n // 2 * mx:  # k 太小（绝对值太大）
                return -1

        @cache
        def dfs(i, s, m, odd, empty):
            nonlocal ret

            if m > limit or m < 0:
                m = -1
            if s == k:
                if not empty:
                    ret = max(ret, m)
             
            if i == n:
                return
                
                
            dfs(i + 1, (s + nums[i]) if not odd else (s - nums[i]), m * nums[i], not odd, False)
            dfs(i + 1, s, m, odd, empty)
        dfs(0, 0, 1, False, True)
        return ret
```

