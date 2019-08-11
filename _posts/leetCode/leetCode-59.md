---
title: leetCode-59:Spiral Matrix II
date: 2019-05-24 22:12:41
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个正整数 n，要求将数字 1 到 n*n 按顺时针方向螺旋排列成 n * n 的矩阵。题目链接：**[点我](https://leetcode.com/problems/spiral-matrix-ii/)**

<!-- more -->

# 样例输入输出

> 输入：3
>
> 输出：[[1,2,3],[8,9,4],[7,6,5]]

> 输入：4
>
> 输出：[[1,2,3,4],[12,13,14,5],[11,16,15,6],[10,9,8,7]]

# 问题解法

此题跟 [leetCode-54](https://leetcode.com/problems/spiral-matrix) 有点类似，直接模拟规则填充数组即可。

为了简化代码，此处分别用 `rowSteps` 和 `colSteps` 表示由当前位置移动到下一个位置的行下标和列下标应该变化的值的列表，两个数组中的元素一一对应，由 `nextStepIndex` 控制应该取数组中的哪个元素。比如 `rowSteps[0] = 0`、`colSteps[0] = 1` 表示由当前位置移动到下一个位置，是行坐标保持不变，列坐标加`1`，再比如 `rowSteps[2] = -1`、`colSteps[2] = 0` ，表示由当前位置移动到下一个位置，是行坐标减 `1`，列坐标保持不变。通过这种方式，可以避免用四个循环来模拟「从左到右」、「从上到下」、「从右到左」、「从下到上」这一个圈，从而减少代码。

至于判断是否要改变方向（即 `nextStepIndex` 的值是否要变动），则根据行列的下标是否达到数组的边界以及是否会覆盖之前填充的数字来判断。如果达到数组的边界或者会覆盖之前填充的数字，则需要改变方向（ `nextStepIndex` 的值加一）。由于数组初始化后所有元素的值都是 0，并且填充的数字永远大于 0，所以可以根据数组的元素值是否为 0 判断该位置是否已经被填充过。

代码如下

```java
class Solution 
{
    // 判断是否需要改变方向获取下一个元素，比如由“从左到右”变成“从上到下”
    private boolean needChangeDirection(int[][] matrix, int nextRowIndex, int nextColIndex)
    {
        // 下标越界，需要改变方向
        if (nextRowIndex < 0 || nextRowIndex >= matrix.length)
        {
            return true;
        }
        
        // 下标越界，需要改变方向
        if (nextColIndex < 0 || nextColIndex >= matrix.length)
        {
            return true;
        }
        
        // 下一个元素已经被填充过，需要改变当前指针移动方向
        if (matrix[nextRowIndex][nextColIndex] != 0)
        {
            return true;
        }
        
        return false;
    }
    
    public int[][] generateMatrix(int n) 
    {
        int[][] matrix = new int[n][n];
        int totalNum = n * n;
        // 由当前元素移动到下一个元素需要的列下标的移动距离，与 rowSteps 的元素一一对应
        int[] colSteps = {1, 0, -1, 0}; 
        // 由当前元素移动到下一个元素需要的行下标的移动距离，与 colSteps 的元素一一对应
        int[] rowSteps = {0, 1, 0, -1}; 
        int currentRowIndex = 0;
        int currentColIndex = 0;
        int nextStepIndex = 0;
        for (int i = 1; i <= totalNum; i++)
        {
            matrix[currentRowIndex][currentColIndex] = i;
            int tempNextRowIndex = currentRowIndex + rowSteps[nextStepIndex];
            int tempNextColIndex = currentColIndex + colSteps[nextStepIndex];
            if (needChangeDirection(matrix, tempNextRowIndex, tempNextColIndex))
            {
                nextStepIndex = (nextStepIndex + 1) % 4;
                tempNextRowIndex = currentRowIndex + rowSteps[nextStepIndex];
                tempNextColIndex = currentColIndex + colSteps[nextStepIndex];
            }
            
            currentRowIndex = tempNextRowIndex;
            currentColIndex = tempNextColIndex;
        }
        
        return matrix;
    }
}
```

