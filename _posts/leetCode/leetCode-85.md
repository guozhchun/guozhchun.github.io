---
title: leetCode-85:Maximal Rectangle
date: 2019-09-30 23:20:17
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个只包含数字 0 和 1 的矩阵，要求找出其中只包含 1 构成的矩形的最大面积。题目链接：**[点我](https://leetcode.com/problems/maximal-rectangle/)**

<!-- more -->

# 样例输入输出

> 输入：[["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
>
> 输出：6

> 输入：[["1"], ["0"], ["1"]]
>
> 输出：1

# 问题解法

## 解法一：栈

将此题转换成[leetCode84:Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram)直方图来求解。分别以矩阵的每一行为基准，以该行上方的数据构造成 leetCode84 题中的直方图，求解出该直方图的最大矩形。当遍历完矩阵每一行求出每个直方图的最大面积后，将这些最大面积进行比较返回其中的最大值即可。代码如下：

```java
class Solution
{
    public int maximalRectangle(char[][] matrix)
    {
        if (matrix == null || matrix.length == 0)
        {
            return 0;
        }
        
        int[][] dp = new int[matrix.length][matrix[0].length];
        for (int i = 0; i < matrix.length; i++)
        {
            for (int j = 0; j < matrix[0].length; j++)
            {
                if (i == 0 || matrix[i][j] == '0')
                {
                    dp[i][j] = matrix[i][j] - '0';
                }
                else
                {
                    dp[i][j] = dp[i - 1][j] + (matrix[i][j] - '0');
                }
            }
        }
        
        int maxArea = 0;
        for (int i = 0; i < dp.length; i++)
        {
            maxArea = Math.max(maxArea, maxRectangleArea(dp[i]));
        }
        
        return maxArea;
    }
    
    private int maxRectangleArea(int[] nums)
    {
        int maxArea = nums[0];
        Stack<Integer> stack = new Stack<>();
        for (int i = 0; i <= nums.length; i++)
        {
            int h = i == nums.length ? 0 : nums[i];
            while (!stack.empty() && nums[stack.peek()] > h)
            {
                int index = stack.pop();
                int width = stack.empty() ? i : i - stack.peek() - 1;
                maxArea = Math.max(maxArea, nums[index] * width);
            }
            stack.push(i);
        }
        return maxArea;
    }
}
```

## 解法二：动态规划

此题解法主要是参考网上的做法。同样是遍历矩阵每一行，以该行为基准，求出该行上方构成的图形中的最大矩形面积。具体做法是：定义 `height[j]` 表示该行 `j` 列所能到达的高度，`left[j]` 表示该行 `j` 位置的高度所能延伸到左边的最左下标，`right[j]` 表示该行 `j` 位置的高度所能延伸到右边的最右下标。考虑到后续计算的方便，如果当前位置是 `0`，则 `left[j] = 0, right[j] = colNum - 1`，此时 `height[j] = 0`，对于计算以该位置的柱子向左右延伸构成的矩形面积并不会发生改变，都是 `0`，这是一步取巧的做法。代码如下

```java
class Solution
{
    public int maximalRectangle(char[][] matrix)
    {
        if (matrix == null || matrix.length == 0)
        {
            return 0;
        }
        
        int colNum = matrix[0].length;
        int[] left = new int[colNum];
        int[] right = new int[colNum];
        int[] height = new int[colNum];
        Arrays.fill(right, colNum - 1);
        int maxArea = 0;
        for (int i = 0; i < matrix.length; i++)
        {
            for (int j = 0; j < colNum; j++)
            {
                if (matrix[i][j] == '1')
                {
                    height[j] = height[j] + 1;
                }
                else
                {
                    height[j] = 0;
                }
            }
            
            int currentLeft = 0;
            for (int j = 0; j < colNum; j++)
            {
                if (matrix[i][j] == '1')
                {
                    left[j] = Math.max(left[j], currentLeft);
                }
                else
                {
                    currentLeft = j + 1;
                    left[j] = 0;
                }
            }
            
            int currentRight = colNum - 1;
            for (int j = colNum - 1; j >=0; j--)
            {
                if (matrix[i][j] == '1')
                {
                    right[j] = Math.min(right[j], currentRight);
                }
                else
                {
                    currentRight = j - 1;
                    right[j] = colNum - 1;
                }
            }
            
            for (int j = 0; j < colNum; j++)
            {
                maxArea = Math.max(maxArea, height[j] * (right[j] - left[j] + 1));
            }
        }
        
        return maxArea;
    }
}
```

