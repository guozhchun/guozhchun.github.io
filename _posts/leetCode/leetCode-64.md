---
title: leetCode-64:Minimum Path Sum
date: 2019-02-23 21:58:52
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个矩阵，要求找出一个从左上角到右下角的路径，使其和最小并输出该值。其中，每次只能向下或向右移动一个单位。题目链接：**[点我](https://leetcode.com/problems/minimum-path-sum/)**

<!-- more -->

# 样例输入输出

> 输入：[[1,3,1],[1,5,1],[4,2,1]]
>
> 输出：7

> 输入：[[1,2,3]]
>
> 输出：6

# 问题解法

此题同 [LeetCode-63](https://guozhchun.github.io/2018/12/02/leetCode/leetCode-63/) 类似，都可以用动态规划来解决。假设用 `dp[i][j]` 来表示从左上角到 `[i][j]` 位置经过的路径中的最短路径的和，根据题意可知，其上一个位置要么在左侧要么在上方，因此只需比较这两个位置，取其最小值，加上本位置的数字就能得到最终结果。动态方程为：`dp[i][j] = min(dp[i][j - 1], dp[i - 1][j]) + grid[i][j]`。代码如下：

```java
class Solution 
{
    public int minPathSum(int[][] grid) 
    {
        if (grid == null)
        {
            return 0;
        }
        
        int row = grid.length;
        int col = grid[0].length;
        
        int[][] dp = new int[row][col];
        dp[0][0] = grid[0][0];
        for (int i = 1; i < row; i++)
        {
            dp[i][0] = dp[i - 1][0] + grid[i][0];
        }
        
        for (int j = 1; j < col; j++)
        {
            dp[0][j] = dp[0][j - 1] + grid[0][j];
        }
        
        if (row == 1 || col == 1)
        {
            return dp[row - 1][col - 1];
        }
        
        for (int i = 1; i < row; i++)
        {
            for (int j = 1; j < col; j++)
            {
                dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
            }
            
        }
        
        return dp[row - 1][col - 1];
    }
}
```

