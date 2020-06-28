---
title: leetCode-96:Unique Binary Search Trees
date: 2020-05-10 11:01:13
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个正整数 `n`，要求找出以数字 `1 ~ n` 构成的不同结构的二叉搜索树的数量。题目链接：**[点我]( https://leetcode.com/problems/unique-binary-search-trees/)**

<!-- more -->

# 样例输入输出

> 输入：3
>
> 输出：5

> 输入：4
>
> 输出：14

# 问题解法

对数字 `1 ~ n` 中的每个数字，分别以其作为树的根节点，可以将剩下的数字分成两堆，小于根节点数字的和大于根节点数字的，只要将左子树的数量和右子树的数量相乘，就能得到以当前数字作为根节点时其二叉搜索树的数量，将每个数字作为根节点时的二叉搜索树的数量相加就是最终的结果。对于左子树和右子树，其构成的二叉搜索树的数量的算法与计算总体二叉搜索树的数量算法相同，因此可以用递归来解决，其计算公式为：`numTrees(n) = numTrees(0) * numTrees(n - 1) + numTrees(1) * numTress(n - 2) + numTrees(2) * numTrees(n - 3) + ... + numTrees(n - 1) * numTrees(0)`。由于这中间存在大量的重复计算，因此可以用一个数组来保存已经计算过的数值，提高计算效率。代码如下

```java
class Solution
{
    public int numTrees(int n)
    {
        int[] dp = new int[n + 1];
        dp[0] = 1;
        dp[1] = 1;
        return numTrees(n, dp);
    }
    
    private int numTrees(int n, int[] dp)
    {
        if (dp[n] != 0)
        {
            return dp[n];
        }
        
        for (int i = 1; i <= n; i++)
        {
            dp[n] += numTrees(i - 1, dp) * numTrees(n - i, dp);
        }
        
        return dp[n];
    }
}
```

