---
title: leetCode-115:Distinct Subsequences
date: 2019-11-17 13:50:36
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定两个字符串 `S` 和 `T`，要求找出 `S` 中的子字符串（保留`S` 字符串的顺序并在 `S` 字符串中去掉 0 个或多个字符剩下字符串）中与 `T` 字符串相等的个数。题目链接：**[点我]( https://leetcode.com/problems/distinct-subsequences/ )**

<!-- more -->

# 样例输入输出

> 输入：S = "rabbbit", T = "rabbit"
>
> 输出：3

> 输入：S = "babgbag", T = "bag"
>
> 输出：5

# 问题解法

采用动态规划的方式，用 `dp[i][j]` 表示 `S` 串前 `i` 个字符构成的字符串，其经过变换得到的子字符串等于 `T` 串前 `j` 个字符构成的子字符串的个数。动态转移方程为：如果 `S[i]` 和 `T[j]` 相等，那 `dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j]`，如果 `S[i]` 和 `T[j]` 不相等，那 `dp[i][j] = dp[i - 1][j]`。代码如下

```java
class Solution
{
    public int numDistinct(String s, String t)
    {
        if (s == null || t == null || s.length() == 0 || t.length() == 0)
        {
            return 0;
        }
        
        int sLength = s.length();
        int tLength = t.length();
        int[][] dp = new int[sLength][tLength];
        if (s.charAt(0) == t.charAt(0))
        {
            dp[0][0] = 1;
        }
        
        for (int i = 1; i < sLength; i++)
        {
            if (s.charAt(i) == t.charAt(0))
            {
                dp[i][0] = dp[i - 1][0] + 1;
            }
            else
            {
                dp[i][0] = dp[i - 1][0];
            }
        }
        
        for (int i = 1; i < sLength; i++)
        {
            for (int j = 1; j < tLength; j++)
            {
                if (s.charAt(i) == t.charAt(j))
                {
                    dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
                }
                else
                {
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }
        
        return dp[sLength - 1][tLength - 1];
    }
}
```

