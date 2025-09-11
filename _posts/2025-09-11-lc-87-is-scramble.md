---
layout: post
title: "LeetCode 87. 扰乱字符串题解"
date: 2025-09-11 12:00:00 +0800
categories: leetcode
---



> 思路

状态转移的前提是清晰的状态定义，这是动态规划的核心。我们的三维 DP 数组定义如下：
```text
状态变量
i：字符串 s1 的起始索引（表示我们关注 s1 中从 i 开始的子串）。
j：字符串 s2 的起始索引（表示我们关注 s2 中从 j 开始的子串）。
k：子串的长度（表示我们关注的 s1 和 s2 子串的长度均为 k）。
状态含义
dp[i][j][k] = 布尔值，表示：
s1 中从 i 开始、长度为 k 的子串（记为 s1_sub = s1[i..i+k-1]），是否是 s2 中从 j 开始、长度为 k 的子串（记为 s2_sub = s2[j..j+k-1]）的扰乱字符串。
```

要判断 dp[i][j][k] 是否为 true，需要基于「扰乱字符串」的定义：递归分割为两部分，可选择交换这两部分。
具体来说，对于长度为 k 的子串，我们可以在任意位置 m（1 ≤ m < k）将其分割为两部分：
第一部分长度为 m，第二部分长度为 k - m。
此时有两种可能的情况，只要满足其中一种，dp[i][j][k] 就为 true。
- 情况 1：分割后「不交换」两部分
即 s1_sub 分割为 s1_left = s1[i..i+m-1] 和 s1_right = s1[i+m..i+k-1]，s2_sub 分割为 s2_left = s2[j..j+m-1] 和 s2_right = s2[j+m..j+k-1]。
要求：s1_left 是 s2_left 的扰乱串，且 s1_right 是 s2_right 的扰乱串。
对应状态：dp[i][j][m] && dp[i+m][j+m][k-m]。
- 情况 2：分割后「交换」两部分
即 s1_sub 分割为 s1_left 和 s1_right 后，交换得到 s1_right + s1_left；此时 s2_sub 分割为 s2_left' = s2[j..j+(k-m)-1] 和 s2_right' = s2[j+(k-m)..j+k-1]（因为交换后 s1_left 对应 s2_right'，s1_right 对应 s2_left'）。
要求：s1_left 是 s2_right' 的扰乱串，且 s1_right 是 s2_left' 的扰乱串。
对应状态：dp[i][j + (k - m)][m] && dp[i + m][j][k - m]。
最终转移方程
综合两种情况，对于所有可能的分割点 m（1 ≤ m < k），只要有一个 m 满足上述两种情况之一，dp[i][j][k] 就为 true。
用公式表示为：
```plaintext
dp[i][j][k] = OR (  dp[i][j][m] && dp[i+m][j+m][k-m]   )  对于所有 1 ≤ m < k
OR (  dp[i][j + k - m][m] && dp[i+m][j][k-m]  )
```

当子串长度 k = 1 时（即单个字符），只有当两个字符完全相等时，才是扰乱串。因此边界条件为：

```plaintext
dp[i][j][1] = (s1.charAt(i) == s2.charAt(j))
```

> 代码

```java
class Solution {
    public boolean isScramble(String s1, String s2) {
        int n = s1.length();
        if (n != s2.length()) return false;
        // dp[i][j][k]：s1从i开始、长度k的子串，是否是s2从j开始、长度k的子串的扰乱串
        boolean[][][] dp = new boolean[n][n][n + 1];

        // 边界条件：长度为1的子串
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                dp[i][j][1] = (s1.charAt(i) == s2.charAt(j));
            }
        }

        // 枚举子串长度k（从2到n）
        for (int k = 2; k <= n; k++) {
            // 枚举s1的起始位置i（确保i+k-1 < n）
            for (int i = 0; i <= n - k; i++) {
                // 枚举s2的起始位置j（确保j+k-1 < n）
                for (int j = 0; j <= n - k; j++) {
                    // 枚举分割点m（将长度k分为m和k-m两部分）
                    for (int m = 1; m < k; m++) {
                        // 情况1：不交换分割后的两部分
                        boolean noSwap = dp[i][j][m] && dp[i + m][j + m][k - m];
                        // 情况2：交换分割后的两部分
                        boolean swap = dp[i][j + k - m][m] && dp[i + m][j][k - m];
                        // 两种情况满足一种即可
                        if (noSwap || swap) {
                            dp[i][j][k] = true;
                            break; // 找到一种分割方式即可，无需继续枚举m
                        }
                    }
                }
            }
        }

        // 最终答案：s1从0开始、长度n的子串，是否是s2从0开始、长度n的子串的扰乱串
        return dp[0][0][n];
    }
}
```
- 时间复杂度: O(n^3)
- 空间复杂度: O(n^3)