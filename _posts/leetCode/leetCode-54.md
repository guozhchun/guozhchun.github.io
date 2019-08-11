---
title: leetCode-54:Spiral Matrix
date: 2019-05-18 21:13:50
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个整数的二维数组（矩阵），要求将其数字按照螺旋式顺时针方向放入 List 列表中并返回该结果列表。题目链接：**[点我](https://leetcode.com/problems/spiral-matrix/)**

<!-- more -->

# 样例输入输出

> 输入：[[1, 2, 3], [4, 5, 6]]
>
> 输出：[1, 2, 3, 6, 5, 4]

> 输入：[[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12]]
>
> 输出：[1, 2, 3, 4, 8, 12, 11, 10, 9, 5, 6, 7]

# 问题解法

直接按照题目要求模拟元素的走法，每次将遍历的元素放入列表中，最后返回列表即可。

分别用一个变量表示上、下、左、右的边界，当元素移动时，用元素的下标和这个边界值进行比较，避免将同一位置的元素重复放入结果列表中。每次移动元素到达相应的边界时（遍历完一行或者一列），更新对应的边界值。

代码如下

```java
class Solution 
{
    public List<Integer> spiralOrder(int[][] matrix) 
    {
        if (matrix == null || matrix.length == 0)
        {
            return new ArrayList<Integer>();
        }
        
        int total = matrix.length * matrix[0].length;
        int topBound = 0;
        int bottomBound = matrix.length;
        int leftBound = 0;
        int rightBound = matrix[0].length;
        int i = 0;
        int j = 0;
        List<Integer> result = new ArrayList<>(total);
        while (result.size() < total)
        {
            while (result.size() < total && j < rightBound)
            {
                result.add(matrix[i][j]);
                if (j + 1 == rightBound)
                {
                    topBound++;
                    i++;
                    break;
                }
                else
                {
                    j++;
                }
            }
            
            while (result.size() < total && i < bottomBound)
            {
                result.add(matrix[i][j]);
                if (i + 1 == bottomBound)
                {
                    rightBound--;
                    j--;
                    break;
                }
                else
                {
                    i++;
                }
            }
            
            while (result.size() < total && j >= leftBound)
            {
                result.add(matrix[i][j]);
                if (j == leftBound)
                {
                    bottomBound--;
                    i--;
                    break;
                }
                else
                {
                    j--;
                }
            }
            
            while (result.size() < total && i >= topBound)
            {
                result.add(matrix[i][j]);
                if (i == topBound)
                {
                    leftBound++;
                    j++;
                    break;
                }
                else 
                {                    
                    i--;
                }
            }
        }
        
        return result;
    }
}
```

