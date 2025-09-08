---
layout: post
title: "LeetCode 1. Two Sum 题解"
date: 2025-09-08 12:00:00 +0800
categories: leetcode
---

> 思路

因为是n * n, 所以可以画一个图, 长度是length, n1 = length / 2, n2 = length - n1, 这片区域经过旋转4次就得到了整个图形
- [x, y]
- [y, n - 1 - x]
- [n - 1 - x, n - 1 - y]
- [n - 1 - y, x]
旋转中位置的坐标关系


> 解法

```java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;
        int n1 = n / 2;
        int n2 = n - n / 2;
        for (int i = 0; i < n1; i++) {
            for (int j = 0; j < n2; j++) {
                int x = i;
                int y = j;
                int temp = matrix[x][y];
                matrix[x][y] = matrix[n - 1 - y][x];
                matrix[n - 1 - y][x] = matrix[n - 1 - x][n - 1 - y];
                matrix[n - 1 - x][ n - 1 - y] = matrix[y][n - 1 - x];
                matrix[y][n - 1 - x] = temp;
            }
        }
    }
}
```

> 复杂度

- 时间: O(n^2)
- 空间: O(1)

