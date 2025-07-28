---
title: 力扣周赛453
date: 2025-06-08 21:10:37
tags:
---

题三这真是中等吗。。。不过好在压哨通过了；为了学c++，以后算法笔记用c++写

[题三-3578. 统计极差最大为 K 的分割方式数](https://leetcode.cn/problems/count-partitions-with-max-min-difference-at-most-k/)

滑窗+前缀和优化dp

```c++
class Solution {
public:
    int bisect(vector<int>& nums, int target) {
        int i=0, j = nums.size();
        if (j == 0) {
            return 0;
        }
        if (target <= nums[0]) return 0;
        if (target == nums[nums.size() - 1]) return nums.size() - 1;
        while (i + 1 < j) {
            int mid = (i + j) / 2;
            if (nums[mid] == target) {
                return mid;
            } else if (nums[mid] > target) {
                j = mid;
            } else {
                i = mid;
            }
        }
        
        return j;
    }
    int countPartitions(vector<int>& nums, int k) {
        int n = nums.size();
        int* left = new int[n];
        int i = n - 1;
        int MOD = 1000000007;
        vector<int> tmp;
        for (int j = n - 1; j >= 0; j --) {
            while (i >= 0) {
                if (tmp.size() == 0 || tmp[tmp.size() - 1] - k <=nums[i] && nums[i] <= tmp[0] + k) {
                    int bi = bisect(tmp, nums[i]);
                    tmp.insert(tmp.begin() + bi, nums[i]);
                    i--;
                }
                else {
                    break;
                }
            }
            left[j] = i + 1;
            int bi = bisect(tmp, nums[j]);
            tmp.erase(tmp.begin() + bi);
        }

        int* dp = new int[n]{0};
        dp[0] = 1;
        int* presum = new int[n + 1]{0};
        
        for (i=1; i<n; i++) {
            presum[i] = (dp[i - 1] + presum[i - 1]) % MOD;
            if (left[i] == 0){
                dp[i] = presum[i] + 1;
            } else {
                dp[i] = (presum[i] - presum[left[i] - 1] + MOD) % MOD;
            }
        }
        
        return dp[n - 1];
    }

};
```

