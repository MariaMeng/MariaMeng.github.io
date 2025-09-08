---
layout: post
title: "LeetCode 241. 为运算表达式设计优先级 题解"
date: 2025-09-08 16:31:00 +0800
categories: leetcode
---

> 思路

方法一: 分治 + 记忆化搜索


就是遍历判断如果当前字符是操作符, 就以当前字符为分割, 拆成两部分
```text
left op right
```
left 为 操作符左侧的求解集合
right 为 操作符右侧的求解集合

原问题: 从第一个位置到最后一个位置的求解集合, 转换为两个子问题的求解集合

然后根据 op 具体的操作符类型进行运算

子问题可能会被多次调用, 所以需要存起来, 防止重复计算, 就是记忆化搜索
递归求解

方案二: 动态规划

dp[i][j] 表示从第i个位置到第j个位置组成的表达式的求解集合
所以，当 i < k < j 时, 如果k 为操作符,则dp[i][j] = dp[i][k - 1]集合 与 dp[k + 1][j]集合的运算结果, 就是原问题的解

遍历方式, 是"自下而上"
初始化, 
先计算长度为1的dp[i][j], 就是dp[i][i], 数值是数值本身
再计算长度为3的dp[i][j]
再计算长度为5的dp[i][j]
再计算长度为7的dp[i][j]
...
再计算长度为n的dp[i][j]

迭代求解

> 代码

```java
class Solution {

    int PLUS = -1;
    int SUB = -2;
    int MUL = -3;
    
    public List<Integer> diffWaysToCompute(String expression) {
        List<Integer> op = new ArrayList<>();
        int len = expression.length();
        int num = 0;
        
        // 数字和操作数元素构成的集合
        for (int i = 0; i < len;) {
            if (expression.charAt(i) == '+') {
                op.add(PLUS);
                i++;
            } else if (expression.charAt(i) == '-') {
                op.add(SUB);
                i++;
            } else if (expression.charAt(i) == '*') {
                op.add(MUL);
                i++;
            } else {
                num = 0;
                while (i < len && Character.isDigit(expression.charAt(i))) {
                    num = num * 10 + (expression.charAt(i) - '0');
                    i++;
                }
                op.add(num);
            }
        }

        // 遍历集合中的操作数, 分治 + 记忆化搜索
        List<Integer>[][] dp = new List[op.size()][op.size()];
        
        // 初始化
        for (int i = 0; i < op.size(); i++) {
            for (int j = 0; j < op.size(); j++) {
                dp[i][j] = new ArrayList<>();
                if (i == j && op.get(i) >= 0) {
                    dp[i][j].add(op.get(i));
                }
            }
        }

        for (int j = 2; j < op.size(); j = j + 2) {
            for (int i = 0; i + j < op.size(); i = i + 2) {
                int l = i, r = i + j;
                for (int k = l + 1; k < r; k = k + 2) {
                    List<Integer> left = dp[l][k - 1];
                    List<Integer> right = dp[k + 1][r];
                    for (int num1 : left) {
                        for (int num2 : right) {
                            if (op.get(k) == PLUS) {
                                dp[l][r].add(num1 + num2);
                            } else if (op.get(k) == SUB) {
                                dp[l][r].add(num1 - num2);
                            } else if (op.get(k) == MUL) {
                                dp[l][r].add(num1 * num2);
                            }
                        }
                    }
                }
            }
        }
        
        return dp[0][op.size() - 1];

    }

}

```

> 复杂度

- 时间: O(n ^ 3), i, j, k 三个维度了
- 空间: O(n ^ 2), 二维数组