---
layout: post
title: "LeetCode 300. 最长递增子序列"
date: 2025-09-09 22:31:00 +0800
categories: leetcode
---

# 这道题两种解法都需要掌握, 面试官可能会让你做时间复杂度上的优化!!

# `做题时长: 15分钟, 方案一: 10分钟, 方案二: 15分钟`

 > # 思路

### 方案一: 动态规划

`dp[i]`含义: 以第i个位置的元素作为子序列最后一个元素时, 的最长长度

dp[i] 分2中情况,
- 情况1: 自己一个元素, dp[i] = 1, 这种情况是在j在0~i-1位置中, 均不存在nums[i] > nums[j]的情况下
- 情况2: 在j在0~i-1位置中, 最大的dp[j] + 1

- `时间: O(n ^ 2)`
- `空间: O(n)`

#### 代码

```java
public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        dp[0] = 1;

        int res = 1;
        for (int i = 1; i < n; i++) {
            dp[i] = 1;
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            res = Math.max(res, dp[i]);
        }

        return res;
    }
```

### 方案二: 贪心 + 二分

`dp[i]`含义: 以i为长度的最长子序列的最后一个元素的 `最小值`
dp[i] 是关于i单调递增的
len为长度, 如果nums[i] > dp[len], 则dp[++len] = nums[i]
否则的话, 去nums数组的0~len的范围查询第一个小于nums[i]的位置d[k], 然后更新d[k + 1]

- `时间: O(nlogn)`
- `空间: O(n)`



#### 代码
```java
public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        dp[0] = nums[0];
        int len = 0;

        for (int i = 1; i < n; i++) {
            if (nums[i] > dp[len]) {
                dp[++len] = nums[i];
            } else {
                int pos = search(dp, 0, len, nums[i]);
                dp[pos] = nums[i];
            }
        }
        return len + 1;
    }

    private int search(int[] dp, int l, int r, int target) {
        int pos = dp.length;
        while (l <= r) {
            int mid = l + (r - l) / 2;
            if (target <= dp[mid]) {
                r = mid - 1;
                pos = mid;
            } else {
                l = mid + 1;
            }
        }
        return pos;
    }
```