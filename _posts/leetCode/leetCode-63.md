---
title: leetCode-63:Unique Paths II
date: 2018-12-02 10:38:25
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个矩阵，矩阵上的元素是 0 或 1，1 表示这是一个障碍物，机器人不能通过，0 表示机器人可以通过。一个机器人在矩阵左上角，目标在矩阵右下角，机器人每次只能向右或向下走一步，要求找出机器人从左上角走到右下角共有多少种走法。题目链接：**[点我](https://leetcode.com/problems/unique-paths-ii/)**

<!-- more -->

# 样例输入输出

> 输入：[[0, 0, 0], [0, 1, 0], [0, 0, 0]]
>
> 输出：2

> 输入：[[1], [0], [0]]
>
> 输出：0 

# 问题解法

## 暴力搜索

此题最直观的做法就是直接用深搜进行暴力搜索，虽然简单直观，但是**超时**了。代码如下

```java
class Solution 
{
    private int getPathCount(int[][] obstacleGrid, int i, int j, int count)
    {
        // 如果当前位置是障碍点，则不能达到终点
        if (obstacleGrid[i][j] == 1)
        {
            return count;
        }
        
        // 到达终点
        if (i + 1 == obstacleGrid.length && j + 1 == obstacleGrid[0].length)
        {
            return count + 1;
        }
        
        // 向下
        if (i + 1 < obstacleGrid.length && obstacleGrid[i + 1][j] == 0)
        {
            count = getPathCount(obstacleGrid, i + 1, j, count);
        }
        
        // 向右
        if (j + 1 < obstacleGrid[0].length && obstacleGrid[i][j + 1] == 0)
        {
            count = getPathCount(obstacleGrid, i, j + 1, count);
        }
        
        return count;
    }
    
    public int uniquePathsWithObstacles(int[][] obstacleGrid) 
    {
        return getPathCount(obstacleGrid, 0, 0, 0);
    }
}
```

## 动态规划

暴力搜索有很多重复的搜索过程，比如某一次搜索能达到终点，后面另一次搜索经过之前搜索过的能到达终点的路径上的某个节点时，就不必继续往下搜索了，因为这很明确是能到达终点的。而且，如果能记录某个位置达到终点的路径数量，那么后续搜索到达此位置时，就可以直接将其结果返回，不必往下继续搜索了。因此可以使用动态规划来解决。

使用二维数组 `dp[i][j]` 来表示位置 (i, j) 到达终点的路径数量，则动态方程为 `dp[i][j] = dp[i][j + 1] + dp[i + 1][j]`，最终结果就是 `dp[0][0]`

代码如下

```java
class Solution 
{
    public int uniquePathsWithObstacles(int[][] obstacleGrid) 
    {
        int rowCount = obstacleGrid.length;
        int colCount = obstacleGrid[0].length;
        int[][] dp = new int[rowCount][colCount];
        if (obstacleGrid[rowCount - 1][colCount - 1] == 0)
        {
            dp[rowCount - 1][colCount - 1] = 1;
        }
        
        for (int i = rowCount - 1; i >= 0; i--)
        {
            for (int j = colCount - 1; j >= 0; j--)
            {
                // 当前位置是障碍物时，到达终点的路径是 0
                if (obstacleGrid[i][j] == 1)
                {
                    continue;
                }
                
                if (j + 1 < colCount && obstacleGrid[i][j + 1] == 0)
                {
                    dp[i][j] += dp[i][j + 1];
                }
                
                if (i + 1 < rowCount && obstacleGrid[i + 1][j] == 0)
                {
                    dp[i][j] += dp[i + 1][j];
                }
            }
        }
        
        return dp[0][0];
    }
}
```

