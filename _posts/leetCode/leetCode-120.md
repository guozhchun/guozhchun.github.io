---
title: leetCode-120:Triangle
date: 2019-11-10 14:50:11
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个二维列表，表示一个正三角形，要求找出从三角形的顶点到底部经过的路径的最小值。每次移动只能本数字的左下相邻或者右下相邻的数字移动。题目链接：**[点我](https://leetcode.com/problems/triangle/)**

<!-- more -->

# 样例输入输出

> 输入：[[2],[3,4],[6,5,7],[4,1,8,3]]， 表示的三角形如下：
> &nbsp;&nbsp;&nbsp;&nbsp;2
> &nbsp;&nbsp;3,4
> &nbsp;6,5,7
>4,1,8,3
> 输出：11。其经过的路径为：2->3->5->1

> 输入：[[2],[3,4],[6,5,7],[4,10,8,3]]， 表示的三角形如下：
> &nbsp;&nbsp;&nbsp;&nbsp;2
> &nbsp;&nbsp;3,&nbsp;4
> &nbsp;6,&nbsp;5,&nbsp;7
> 4,10,8,3
>
> 输出：15。其经过的路径为：2->3->6->4

# 问题解法

## 二维数组动态规划

根据题目要求，可以得知，假设 `sum[i][j]` 表示三角形从顶点开始到达第 `i` 行第 `j` 列位置的路径和，则 `sum[i][j] = sum[i - 1][j - 1] + a[i][j]`  或者 `sum[i][j] = sum[i - 1][j] + a[i][j]`。因此，只要比较这两个值，就能得到从三角形顶点到达此位置的路径的和的最小值。按照此算法，依次算出当前行每个位置的最小值，然后将这些值进行比较，取出最小值就是三角形顶点到达底部的路径最小和。代码如下

```java
class Solution
{
    public int minimumTotal(List<List<Integer>> triangle)
    {
        if (triangle == null || triangle.size() == 0) {
            return 0;
        }
        
        int size = triangle.size();
        int[][] sum = new int[size][size];
        sum[0][0] = triangle.get(0).get(0);
        for (int i = 1; i < size; i++)
        {
            sum[i][0] = sum[i - 1][0] + triangle.get(i).get(0);
            sum[i][i] = sum[i - 1][i - 1] + triangle.get(i).get(i);
            for (int j = 1; j < i; j++)
            {
                sum[i][j] = Math.min(sum[i - 1][j - 1], sum[i - 1][j]) + triangle.get(i).get(j);
            }
        }
        
        int result = sum[size - 1][0];
        for (int j = 1; j < size; j++)
        {
            result = Math.min(sum[size - 1][j], result);
        }
        
        return result;
    }
}
```

## 一维数组动态规划

上面的解法虽然直观，理解起来也比较容易，但是其空间复杂度是 `O(n * n)`，可以再进一步提升优化成 `O(n)`，n 是三角形的行数。仔细观察上述的解法，每次计算 `sum[i][j]` 时，都是用的 `sum[i - 1][j - 1]` 和 `sum[i - 1][j]`，也就是说，计算本行的结果只依赖上一行本位置和上一行本位置前一个位置这两个位置的结果，再从题目中可以看出，每一行都比上一行多了一个值。所以，可以考虑，在计算本行结果时，从后面开始往前计算，并且计算结果直接放在本数组中（替换原先的值），这样，既不会因为修改数组的值导致后续的计算读到覆盖后的值，也可以复用数组减少空间的消耗。代码如下

```java
class Solution
{
    public int minimumTotal(List<List<Integer>> triangle)
    {
        if (triangle == null || triangle.size() == 0)
        {
            return 0;
        }
        
        int size = triangle.size();
        int[] sum = new int[size];
        sum[0] = triangle.get(0).get(0);
        for (int i = 1; i < size; i++)
        {
            sum[i] = sum[i - 1] + triangle.get(i).get(i);
            for (int j = i - 1; j > 0; j--)
            {
                sum[j] = Math.min(sum[j], sum[j - 1]) + triangle.get(i).get(j);
            }
            sum[0] = sum[0] + triangle.get(i).get(0);
        }
        
        int result = sum[0];
        for (int i = 1; i < size; i++)
        {
            result = Math.min(result, sum[i]);
        }
        
        return result;
    }
}
```

