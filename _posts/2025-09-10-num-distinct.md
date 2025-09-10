---
layout: post
title: "LeetCode 115. 不同的子序列"
date: 2025-09-10 22:31:00 +0800
categories: leetcode
---

> 思路

dp[i][j], s串种前i个元素的字符串和t串前j个元素组成的字符串, 构成的方案数
```text
如果s.charAt(i - 1) == s.charAt(j - 1),
dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j]
可以选择用还是不用, 用这个i位置匹配的话, 转换为求解s串种前i-1个元素构成的子串和t串前j-1个元素构成的子串的方案数
可以选择不用, 转换为求解s串种前i-1个元素构成的子串和t串种前j个元素构成的子串的方案数

如果s.charAt(i - 1) != s.charAt(j - 1)
用s串种前i-1个元素构成的子串去匹配前j个元素的子串的方案数
dp[i][j] = dp[i - 1][j]
```
代码
```java
class Solution {
    public int numDistinct(String s, String t) {
        int m = s.length();
        int n = t.length();

        int[][] dp = new int[m + 1][n + 1];
        
        for (int i = 0; i <= m; i++) {
            dp[i][0] = 1;
        }

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s.charAt(i - 1) == t.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
                } else {
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }

        return dp[m][n];
    }
}
```

时间复杂度： O（n ^ 2）
空间复杂度：O（n ^ 2）