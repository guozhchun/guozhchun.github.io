---
title: leetCode-51:N-Queens
date: 2020-04-25 17:17:49
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个数字 `n`，要求在 `n * n` 的棋盘上摆放 `n` 个皇后，使得各个皇后间不会互相攻击。输出所有可能的结果。题目链接：**[点我](https://leetcode.com/problems/n-queens/)**

<!-- more -->

# 样例输入输出

> 输入：1
>
> 输出：[[Q]]

> 输入：4
>
> 输出：[
> [".Q..",  // Solution 1
> "...Q",
> "Q...",
> "..Q."],
>
> ["..Q.",  // Solution 2
> "Q...",
> "...Q",
> ".Q.."]
> ]
>
> 
>
> Q 表示皇后，. 表示空白

# 问题解法

N 皇后问题，用回溯算法求解

```java
class Solution 
{
    private int[] dx = {-1, -1, 0, 1, 1, 1, 0, -1};

    private int[] dy = {0, 1, 1, 1, 0, -1, -1, -1};

    private int[][] occupy;

    private char[][] grid;

    private int cellNum;

    public List<List<String>> solveNQueens(int n)
    {
        init(n);
        List<List<String>> result = new LinkedList<>();
        searchQueen(0, result);
        return result;
    }

    private void searchQueen(int num, List<List<String>> result)
    {
        if (cellNum == num)
        {
            result.add(getGridValueAsList());
            return;
        }

        for (int i = 0; i < cellNum; i++)
        {
            if (occupy[num][i] == 0)
            {
                grid[num][i] = 'Q';
                handleOccupy(num, i, 1);
                searchQueen(num + 1, result);
                grid[num][i] = '.';
                handleOccupy(num, i, -1);
            }
        }
    }

    private List<String> getGridValueAsList()
    {
        List<String> ans = new LinkedList<>();
        for (int i = 0; i < cellNum; i++)
        {
            StringBuilder sb = new StringBuilder();
            for (int j = 0; j < cellNum; j++)
            {
                sb.append(grid[i][j]);
            }
            ans.add(sb.toString());
        }

        return ans;
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

