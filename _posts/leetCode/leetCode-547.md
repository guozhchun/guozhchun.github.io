---
title: leetCode-547:Friend Circles
date: 2020-04-04 19:37:31
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个 n * n 的二维数组，用来表示班级成员的朋友关系。如果 `M[i][j]` 值为 1，表示第 `i` 人和第 `j` 人是朋友关系，如果值为 0，表示第 `i` 人和第 `j` 人不是朋友关系。朋友关系具有传递性，如 a 和 b 是朋友，b 和 c 是朋友，则 a 和 c 也是朋友。要求找出有班级中多少个朋友圈。题目链接：**[点我]( https://leetcode.com/problems/friend-circles)**

<!-- more -->

# 样例输入输出

> 输入： [[1,1,0],[1,1,1],[0,1,1]] 
>
> 输出：1

> 输入： [[1,1,0],[1,1,0],[0,0,1]] 
>
> 输出：2

# 问题解法

用并查集算法进行解答。假设每个人都有一个上层的朋友，如果该成员没有任何朋友，则其上层的朋友就是自己。首先，先初始化每个人的上层朋友，然后遍历二维数组，对每一对朋友关系，都分别找出这两个成员的最上层的朋友，如果值不相等，则将其合并，最后再遍历定义的上层朋友的数组，统计上层朋友是自己的成员个数，就是班级朋友圈的个数。代码如下

```java
class Solution
{
    private int findRoot(int[] friend, int i)
    {
        while (friend[i] != i)
        {
            i = friend[i];
        }
        
        return i;
    }
    
    public int findCircleNum(int[][] M)
    {
        int n = M.length;
        int[] friend = new int[n];
        for (int i = 0; i < M.length; i++)
        {
            friend[i] = i;
        }
        
        for (int i = 0; i < n; i++)
        {
            for (int j = i + 1; j < n; j++)
            {
                if (M[i][j] == 1)
                {
                    int iRoot = findRoot(friend, i);
                    int jRoot = findRoot(friend, j);
                    if (iRoot != jRoot)
                    {
                        friend[iRoot] = jRoot;
                    }
                }
            }
        }
        
        int count = 0;
        for (int i = 0; i < n; i++)
        {
            if (friend[i] == i)
            {
                count++;
            }
        }
        
        return count;
    }
}
```

