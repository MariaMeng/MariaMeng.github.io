---
layout: post
title: "LeetCode 84. 柱状图中最大的矩形 题解"
date: 2025-09-10 18:47:00 +0800
categories: leetcode
---

> 思路

left[i][j] 标识第i行中的从左往右, 到j位置为止的1的连续的个数

for 循环遍历每个位置, 如果是1的位置, area初始值为left[i][j], 然后从下往上迭代, 计算left[k][j] * (i - k + 1)的面积, 更新最大值

比如

0 1 1
0 1 1
0 1 1
0 1 1
0 1 1
1 1 1
1 1 1
1 1 1

最右下方的位置, 计算的最大值, 是16

代码
```java
class Solution {
    public int maximalRectangle(char[][] matrix) {
        int m = matrix.length;
        int n = matrix[0].length;

        int[][] left = new int[m][n];
        
        for (int i = 0; i < m;i++) {
            for (int j = 0; j < n; j++) {
                if (matrix[i][j] == '0') {
                    continue;
                }

                if (j == 0) {
                    left[i][j] = 1; 
                } else {
                    left[i][j] = left[i][j - 1] + 1;
                }
            }
        }

        int res = 0;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (matrix[i][j] == '0') {
                    continue;
                }
                int width = left[i][j];
                int area = width;
                for (int k = i - 1; k >= 0; k--) {
                     width = Math.min(width, left[k][j]);
                     area = Math.max(area, width * (i - k + 1));
                }
                res = Math.max(res, area);
            }
        }
        
        return res;
    }
}
```

时间复杂度O(m ^ 2 * n)
空间复杂度O(n ^ 2)
