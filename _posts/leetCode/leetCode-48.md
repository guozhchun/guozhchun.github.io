---
title: leetCode-48:Rotate Image
date: 2018-10-10 23:27:54
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个矩阵，要求在不额外新增矩阵空间的情况下在原矩阵上将矩阵顺时针旋转 90 度。题目链接：**[点我](https://leetcode.com/problems/rotate-image/description/)**

<!-- more -->

# 样例输入输出

> 输入：
> [
>   [1,2,3],
>   [4,5,6],
>   [7,8,9]
> ]
>
> 输出：
> [
>   [7,4,1],
>   [8,5,2],
>   [9,6,3]
> ]

> 输入：
> [
>   [ 5, 1, 9,11],
>   [ 2, 4, 8,10],
>   [13, 3, 6, 7],
>   [15,14,12,16]
> ]
>
> 输出：
> [
>   [15,13, 2, 5],
>   [14, 3, 4, 1],
>   [12, 6, 8, 9],
>   [16, 7,10,11]
> ]

# 问题解法

此题要求在原有矩阵上进行变换，即不能新 new 一个矩阵然后将数字进行填充。因此，可以先将矩阵进行转置，然后再将矩阵向左翻转（以列为单位，首尾互换）。例如，对于以下的矩阵

```
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
]
```

求转置后矩阵如下

```
[
  [1,4,7],
  [2,5,8],
  [3,6,9]
]
```

再向左翻转后，得到的矩阵如下，此时的矩阵就是最开始的矩阵顺时针旋转 90 度的结果。

```
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
]
```

代码如下

```java
class Solution 
{
    public void rotate(int[][] matrix) 
    {
        if (matrix == null || matrix[0] == null || matrix.length == 0 || matrix.length != matrix[0].length)
        {
            return;
        }
        
        int n = matrix.length;
        // 矩阵转置（行列变换）
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < i; j++)
            {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }
        
        // 向左翻转矩阵（以列为单位，首尾互换）
        int len = n / 2;
        for (int j = 0; j < len; j++)
        {
            for (int i = 0; i < n; i++)
            {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[i][n - j - 1];
                matrix[i][n - j - 1] = temp;
            }
        }
    }
}
```

