---
title: leetCode-934:Shortest Bridge
date: 2020-04-17 22:13:08
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个由 `0` 和 `1`  组成的二维矩阵，其中由 `1` 组成的连续区块表示一个小岛（用 `0` 隔开），题目明确有两个小岛，要求找出连接两个小岛之间的最短桥梁距离（通过将 `0` 变成 `1` 表示建桥梁）。题目链接：**[点我]( https://leetcode.com/problems/shortest-bridge/)**

<!-- more -->

# 样例输入

> 输入：[[1,1,1,1,1],[1,0,0,0,1],[1,0,1,0,1],[1,0,0,0,1],[1,1,1,1,1]]
>
> 输出：1

> 输入：[[0,1,0],[0,0,0],[0,0,1]]
>
> 输出：2

# 问题解法

先用深搜找出一个小岛，将其每个节点放入队列中，然后用广搜查找这个小岛到另一个小岛的距离。队列中最先达到另一个小岛的节点距离就是两个小岛中的最短距离。代码如下

```java
class Node
{
    int x;
    int y;
    
    public Node(int x, int y)
    {
        this.x = x;
        this.y = y;
    }
}

class Solution 
{
    private int[] dx = {0, 1, 0, -1};
    
    private int[] dy = {-1, 0, 1, 0};
    
    private void searchFirstIsland(int[][] matrix, Queue<Node> queue, int x, int y)
    {
        if (!isValid(x, y, matrix) || matrix[x][y] != 1)
        {
            return;
        }
        
        queue.offer(new Node(x, y));
        matrix[x][y] = 2;
        
        for (int i = 0; i < 4; i++)
        {
            searchFirstIsland(matrix, queue, x + dx[i], y + dy[i]);
        }      
    }
    
    private Node getFirstNode(int[][] matrix)
    {
        for (int i = 0; i < matrix.length; i++)
        {
            for (int j = 0; j < matrix[0].length; j++)
            {
                if (matrix[i][j] == 1)
                {
                    return new Node(i, j);
                }
            }
        }
        
        return new Node(0, 0);
    }
    
    private boolean isValid(int x, int y, int[][] matrix)
    {
        return x >=0 && x < matrix.length && y >= 0 && y < matrix[0].length;
    }
    
    public int shortestBridge(int[][] A)
    {
        Queue<Node> queue = new ArrayDeque<>();
        Node node = getFirstNode(A);
        searchFirstIsland(A, queue, node.x, node.y);
        
        int count = 0;
        while (!queue.isEmpty())
        {
            for (int i = queue.size(); i > 0; i--)
            {
                Node current = queue.poll();
                for (int k = 0; k < 4; k++)
                {
                    int x = current.x + dx[k];
                    int y = current.y + dy[k];
                    if (!isValid(x, y, A))
                    {
                        continue;
                    }
                    
                    if (A[x][y] == 1)
                    {
                        return count;
                    }
                    
                    if (A[x][y] == 0)
                    {
                        queue.offer(new Node(x, y));
                        A[x][y] = -1;
                    }
                }
            }
            count++;
        }
        
        return count;
    }
}
```

