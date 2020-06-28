---
title: leetCode-74:Search a 2D Matrix
date: 2019-09-21 21:05:20
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个矩阵，其中每一行的数字从左到右是递增的，每一行第一个数比前一行最后一个数大，要求判断给定的数字是否在矩阵中。题目链接：**[点我](https://leetcode.com/problems/search-a-2d-matrix)**

<!-- more -->

# 样例输入输出

> 输入：[[1, 3, 5, 7], [10, 11, 16, 20], [23, 30, 34, 50]]     3
>
> 输出：true

> 输入：[[1, 3, 5, 7], [10, 11, 16, 20], [23, 30, 34, 50]]     13
>
> 输出：false

# 问题解法

此问题最简单是遍历，不过既然题目告诉矩阵的特征，说明有更好的算法。如果将组件每一行连接起来，就是一个递增的数组，就可以用二分搜索来算了。基于此思路，可以先对矩阵最后一列用二分搜索，找到特定的行，再对那行用二分搜索，以此可以找到目标数字是否在矩阵中。此算法的时间复杂度为 `O(lg(m * n))`，与 `m * n` 长度的数组二分查找时间复杂度相同。代码如下

```java
class Solution 
{
    public boolean searchMatrix(int[][] matrix, int target) 
    {
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0)
        {
            return false;
        }
        
        int m = matrix.length;
        int n = matrix[0].length;
        if (matrix[0][0] > target || matrix[m - 1][n - 1] < target)
        {
            return false;
        }
        
        int start = 0;
        int end = m - 1;
        int middle = 0;
        while (start <= end)
        {
            middle = (start + end) / 2;
            if (matrix[middle][n - 1] == target)
            {
                return true;
            }
            
            if (matrix[middle][n - 1] > target)
            {
                end = middle - 1;
            }
            else
            {
                start = middle + 1;
            }
        }
        
        int row = middle;
        if (matrix[middle][n - 1] < target)
        {
            row++;
        }
        
        start = 0;
        end = n - 1;
        while (start <= end)
        {
            middle = (start + end) / 2;
            if (matrix[row][middle] == target)
            {
                return true;
            }
            
            if (matrix[row][middle] > target)
            {
                end = middle - 1;
            }
            else
            {
                start = middle + 1;
            }
        }
        
        return false;
    }
}
```

