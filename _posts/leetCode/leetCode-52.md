---
title: leetCode-52:N-Queens II
date: 2020-04-25 18:21:11
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个数字 `n`，要求在 `n * n` 的棋盘上摆放 `n` 个皇后，使得各个皇后间不会互相攻击，输出所有可能的结果的总数。题目链接：**[点我](https://leetcode.com/problems/n-queens-ii/)**

<!-- more -->

# 样例输入输出

> 输入：4
>
> 输出：2

> 输入：12
>
> 输出： 14200 

# 问题解法

此题跟 [LeetCode-51](https://guozhchun.github.io/leetCode/leetCode-51/) 题目类似，只不过是把结果集合换成了结果数量，因此可以在其基础上进行修改，将结果集变成数量。代码如下

```java
class Solution
{
    private int[] dx = {-1, -1, 0, 1, 1, 1, 0, -1};

    private int[] dy = {0, 1, 1, 1, 0, -1, -1, -1};

    private int[][] occupy;

    private char[][] grid;

    private int cellNum;
    
    public int totalNQueens(int n)
    {
        init(n);
        return searchQueen(0, 0);
    }

    private int searchQueen(int num, int total)
    {
        if (cellNum == num)
        {
            return total + 1;
        }
        
        for (int i = 0; i < cellNum; i++)
        {
            if (occupy[num][i] == 0)
            {
                grid[num][i] = 'Q';
                handleOccupy(num, i, 1);
                total = searchQueen(num + 1, total);
                grid[num][i] = '.';
                handleOccupy(num, i, -1);
            }
        }
        
        return total;
    }

    private void handleOccupy(int row, int col, int step)
    {
        occupy[row][col] += step;
        for (int i = 0; i < dx.length; i++)
        {
            int nextX = row + dx[i];
            int nextY = col + dy[i];
            while (isInBoundary(nextX, nextY))
            {
                occupy[nextX][nextY] += step;
                nextX = nextX + dx[i];
                nextY = nextY + dy[i];
            }
        }
    }

    private boolean isInBoundary(int row, int col)
    {
        return row >= 0 && row < cellNum && col >=0 && col < cellNum;
    }

    private void init(int n)
    {
        cellNum = n;
        grid = new char[n][n];
        occupy = new int[n][n];
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < n; j++)
            {
                grid[i][j] = '.';
            }
        }
    }
}
```

## 另一种解法

上面的解法是在每次放置皇后完成后，立即处理棋盘，设置棋盘上不能放置皇后的位置。另一种做法是放置皇后完成后不做处理，将判断是否能放置皇后的计算步骤延后（提前）处理。由于不用棋盘的放置结果，因此可以不用存储棋盘的信息。代码如下

```java
class Solution
{
    public int totalNQueens(int n)
    {
        return searchQueen(n, 0, new LinkedList<>());
    }

    private int searchQueen(int n, int total, List<Integer> queenCols)
    {
        int row = queenCols.size();
        if (n == row)
        {
            return total + 1;
        }

        for (int i = 0; i < n; i++)
        {
            if (canPlace(queenCols, row, i))
            {
                queenCols.add(i);
                total = searchQueen(n, total, queenCols);
                queenCols.remove(queenCols.size() - 1);
            }
        }

        return total;
    }

    private boolean canPlace(List<Integer> queenCols, int row, int col)
    {
        int size = queenCols.size();
        for (int i = 0; i < size; i++)
        {
            if (queenCols.get(i) == col)
            {
                return false;
            }

            int dx = Math.abs(i - row);
            int dy = Math.abs(queenCols.get(i) - col);
            if (dx == dy)
            {
                return false;
            }
        }

        return true;
    }
}
```

