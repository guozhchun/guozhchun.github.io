---
title: leetCode-73:Set Matrix Zeroes
date: 2019-09-06 23:52:16
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个整数矩阵，要求用 `O(1)` 的空间复杂度将矩阵中数字 0 所在的行和列的数字都变成 0。题目链接：**[点我](https://leetcode.com/problems/set-matrix-zeroes/)**

<!-- more -->

# 样例输入输出

> 输入：[[1,1,1], [1,0,1], [1,1,1]]
>
> 输出：[[1,0,1], [0,0,0], [1,0,1]]

> 输入：[[0,1,2,0], [3,4,5,2], [1,3,1,5]]
>
> 输出：[[0,0,0,0], [0,4,5,0], [0,3,1,0]]

# 问题解法

将第一行和第一列单独出来，先遍历第一行和第一列，看是否存在 0 的元素，如果存在，则说明该行或该列必为 0，此操作放在最后操作。从第二行第二列遍历矩阵，如果遇到元素为 0，则将该元素所在行的首元素和所在列的首元素标记为 0。待循环结束后，再次遍历矩阵，如果遇到元素所在行首元素或所在列首元素为 0，则将当前元素变为 0。最后，再根据首行和首列是否为 0 的标志位决定将首行或首列的元素变为 0。代码如下

```java
class Solution 
{
    public void setZeroes(int[][] matrix)
    {
        int rowNum = matrix.length;
        int colNum = matrix[0].length;
        
        // 判断矩阵首行是否应该设置为 0
        boolean isFirstRowZero = false;
        for (int j = 0; j < colNum; j++)
        {
            if (matrix[0][j] == 0)
            {
                isFirstRowZero = true;
                break;
            }
        }
        
        // 判断矩阵首列是否应该设置为 0
        boolean isFirstColZero = false;
        for (int i = 0; i < rowNum; i++)
        {
            if (matrix[i][0] == 0)
            {
                isFirstColZero = true;
                break;
            }
        }
        
        // 遍历矩阵，将值为 0 的元素所在的行首和所在的列首的元素设置为 0，方便后续操作
        for (int i = 1; i < rowNum; i++)
        {
            for (int j = 1; j < colNum; j++)
            {
                if (matrix[i][j] == 0)
                {
                    matrix[i][0] = 0;
                    matrix[0][j] = 0;
                    
                }
            }
        }
        
        // 从第二行开始遍历矩阵的每一行，如果当前行的首元素是 0，则将当前行的所有元素值设置为 0
        for (int i = 1; i < rowNum; i++)
        {
            if (matrix[i][0] == 0)
            {
                for (int j = 0; j < colNum; j++)
                {
                    matrix[i][j] = 0;
                }
            }
        }
        
        // 从第二列开始遍历矩阵的每一列，如果当前列的首元素是 0，则将当前列的所有元素值设置为 0
        for (int j = 1; j < colNum; j++)
        {
            if (matrix[0][j] == 0)
            {
                for (int i = 0; i < rowNum; i++)
                {
                    matrix[i][j] = 0;
                }
            }
        }
        
        // 如果矩阵首列应该设置为 0，则将当前列的所有元素设置为 0
        if (isFirstColZero)
        {
            for (int i = 0; i < rowNum; i++)
            {
                matrix[i][0] = 0;
            }
        }
        
        // 如果矩阵首行应该设置为 0，则将当前行的所有元素设置为 0
        if (isFirstRowZero)
        {
            for (int j = 0; j < colNum; j++)
            {
                matrix[0][j] = 0;
            }
        }
    }
}
```

