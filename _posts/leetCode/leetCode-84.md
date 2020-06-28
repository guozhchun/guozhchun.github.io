---
title: leetCode-84:Largest Rectangle in Histogram
date: 2019-09-28 16:58:57
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个非负整数的数组，每个数字代表直方图的高度，每个直方图的宽度都是 1，要求找出又这些直方图所能构成的矩形的最大面积。题目链接：**[点我](https://leetcode.com/problems/largest-rectangle-in-histogram/)**

<!-- more -->

# 样例输入输出

> 输入：[2, 1, 5, 6, 2, 3]
>
> 输出：10

> 输入：[5, 4, 0, 1]
>
> 输出：8

# 问题解法

## 解法一：动态规划（空间复杂度不满足）

主要是遍历数组，找出所有任意两个直方图围起来的矩形面积，对其进行比较找出最大面积。观察矩形可知，矩形的宽度可以由左右两个边界值计算得出，矩形的高度是构成矩形的直方图中的最小高度。因此可以用 `dp[i][j]` 表示 `[i, j]` 区间内的直方图的最小高度，转移方程为：`dp[i][j] = min{dp[i + 1][j - 1], a[i], a[j]}`，通过 `(j - i + 1) * dp[i][j]` 可以计算出 `[i, j]` 区间的直方图构成的矩形面积。在计算 `dp` 数组的过程中更新最大矩形的面积。代码如下

```java
class Solution 
{
    public int largestRectangleArea(int[] heights) 
    {
        if (heights == null || heights.length == 0)
        {
            return 0;
        }
        
        int length = heights.length;
        int[][] minHeights = new int[length][length];
        for (int i = 0; i < length; i++)
        {
            minHeights[i][i] = heights[i];
        }
        
        int maxArea = heights[0];
        for (int right = 0; right < length; right++)
        {
            for (int left = 0; left <= right; left++)
            {
                if (left + 1 > right - 1)
                {
                    minHeights[left][right] = Math.min(heights[left], heights[right]);
                }
                else
                {
                    minHeights[left][right] = Math.min(Math.min(heights[left], heights[right]), minHeights[left + 1][right - 1]);
                }
                maxArea = Math.max(maxArea, minHeights[left][right] * (right - left + 1));
            }
        }
        
        return maxArea;
    }
}
```

## 解法二：递推

由于解法一使用动态规划，需要用 `n*n` 的数组来保存状态，消耗空间比较多，也不满足要求。仔细分析上述思路，可以优化如下：用外层 for 循环遍历数组，再用里层 for 循环从外层的指针出开始向左遍历，因此求出直方图最小高度和矩形宽度，计算并更新最大的矩形面积。代码如下

```java
class Solution 
{
    public int largestRectangleArea(int[] heights) 
    {
        if (heights == null || heights.length == 0)
        {
            return 0;
        }
        
        int length = heights.length;
        int maxArea = heights[0];
        for (int right = 0; right < length; right++)
        {
            int minHeight = heights[right];
            for (int left = right; left >= 0; left--)
            {
                minHeight = Math.min(minHeight, heights[left]);
                maxArea = Math.max(maxArea, minHeight * (right - left + 1));
            }
        }
        
        return maxArea;
    }
}
```

## 解法三：栈

方法二虽然解决了空间复杂度的问题，但是其时间复杂度是 `O(n*n)`，[网上](https://www.youtube.com/watch?v=KkJrGxuQtYo)有更好的解法，是借助栈来完成的。其思想是用栈来存储递增的直方图，遇到比当前直方图高度小的，则可以确定以栈顶元素对应的直方图的高度构成的矩形，其右边界是当前数组的元素直方图，左边界是栈顶元素下一个元素，据此可以算出矩形的宽度，如果栈顶元素下一个元素为空，则矩形宽度为当前数组下标。其具体做法是：遍历数组，如果栈为空或当前栈顶元素对应的数字小于等于当前数组元素，则将当前数组下标压如栈中；如果栈顶元素对应的数字大于当前数组元素，则弹出栈顶的元素，获取其对应的高度，然后计算矩形的宽度，宽度等于当前数组下标减去栈顶元素下标再减去一（如果栈为空，则宽度为当前数组下标）。此种做法的时间复杂度为 `O(n)`

代码如下

```java
class Solution 
{
    public int largestRectangleArea(int[] heights) 
    {
        if (heights == null || heights.length == 0)
        {
            return 0;
        }
        
        int length = heights.length;
        int maxArea = heights[0];
        Stack<Integer> stack = new Stack<>();
        // 让 i <= length，是为了避免出现 [1, 2, 3] 这样纯递增的数组导致最后没有计算矩形面积的情况
        // 当 i == length 时，让获取的数组元素为 0，即可保证所有的元素均参与了计算
        for (int i = 0; i <= length; i++)
        {
            int currentHeight = i == length ? 0 : heights[i];
            while (!stack.empty() && heights[stack.peek()] > currentHeight)
            {
                int index = stack.pop();
                int width = stack.isEmpty() ? i : i - stack.peek() - 1;
                maxArea = Math.max(maxArea, width * heights[index]);
            }
            stack.push(i);
        }
        
        return maxArea;
    }
}
```

# 参考资料

https://www.youtube.com/watch?v=KkJrGxuQtYo