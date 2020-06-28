---
title: leetCode-72:Edit Distance
date: 2020-03-14 21:31:14
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定两个字符串，要求计算出从第一个字符串到第二个字符串所需要的最小的操作次数。每次操作允许：在任何一个位置增加一个字符、删除任何一个字符、替换任何一个字符。题目链接：**[点我]( https://leetcode.com/problems/edit-distance)**

<!-- more -->

# 样例输入输出

> 输入：word1 = "horse", word2 = "ros"
>
> 输出：3

> 输入：word1 = "intention", word2 = "execution"
>
> 输出：5

# 问题解法

使用动态规划。用 `dp[i][j]` 表示第一个字符串的前 `i` 个字符组成的子字符串变换到第二个字符串前 `j` 个字符组成的子字符串所需要经过的最小操作次数。则动态转移方程为 `dp[i][j] = min(dp[i - 1][j] + 1, dp[i][j - 1] + 1, dp[i - 1][j - 1] + (word1[i] == word2[j] ? 0 : 1))`。代码如下

```java
class Solution 
{
    public int minDistance(String word1, String word2)
    {
        int len1 = word1.length();
        int len2 = word2.length();
        if (len1 == 0)
        {
            return len2;
        }
        
        if (len2 == 0)
        {
            return len1;
        }
        
        int[][] dp = new int[len1][len2];
        if (word1.charAt(0) != word2.charAt(0))
        {
            dp[0][0] = 1;
        }
        for (int i = 1; i < len1; i++)
        {
            if (word1.charAt(i) == word2.charAt(0))
            {
                dp[i][0] = i;
            }
            else
            {
                dp[i][0] = dp[i - 1][0] + 1;
            }
        }
        
        for (int j = 1; j < len2; j++)
        {
            if (word2.charAt(j) == word1.charAt(0))
            {
                dp[0][j] = j;
            }
            else
            {
                dp[0][j] = dp[0][j - 1] + 1;
            }
        }
        
        for (int i = 1; i < len1; i++)
        {
            for (int j = 1; j < len2; j++)
            {
                int temp = Math.min(dp[i - 1][j] + 1, dp[i][j - 1] + 1);
                if (word1.charAt(i) == word2.charAt(j))
                {
                    dp[i][j] = Math.min(temp, dp[i - 1][j - 1]);
                }
                else
                {
                    dp[i][j] = Math.min(temp, dp[i - 1][j - 1] + 1);
                }
            }
        }
        
        return dp[len1 - 1][len2 - 1];
    }
}
```

# 参考资料

 https://blog.csdn.net/chichoxian/article/details/53944188 