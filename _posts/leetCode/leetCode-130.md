---
title: leetCode-130:Surrounded Regions
date: 2020-06-14 15:10:35
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个二维数组，数组中的元素为 `X` 和 `O`，要求将数组被 `X` 包围的 `O` 转成 `X`，如果 `O` 的区域中有任何一个元素在边界上，则这个区域不进行转换。题目链接：**[点我](https://leetcode.com/problems/surrounded-regions)**

<!-- more -->

# 样例输入输出

> 输入： [["X","X","X","X"],["X","O","O","X"],["X","X","O","X"],["X","O","X","X"]] 
>
> 输出： [["X","X","X","X"],["X","X","X","X"],["X","X","X","X"],["X","O","X","X"]] 

> 输入：[["X","X","X","X"],["X","O","O","X"],["X","X","O","X"],["X","O","O","X"]] 
>
> 输出：[["X","X","X","X"],["X","O","O","X"],["X","X","O","X"],["X","O","O","X"]] 

# 问题解法

使用深搜，对边界上的 `O` 区域先设置一个临时的值 `T`，然后再对二维数组中的 `O` 元素进行搜索遍历，将其变成 `X`，最后遍历二维数据，将 `T` 还原成 `O`。代码如下

```java
class Solution
{
    public void solve(char[][] board)
    {
        if (board == null || board.length == 0)
        {
            return;
        }
        
        changeBoundaryValue(board);
        changeCellValue(board);
        recoverBoundaryValue(board);
    }

    private void recoverBoundaryValue(char[][] board)
    {
        int rows = board.length;
        int cols = board[0].length;
        for (int i = 0; i < rows; i++)
        {
            for (int j = 0; j < cols; j++)
            {
                if (board[i][j] == 'T')
                {
                    board[i][j] = 'O';
                }
            }
        }
    }

    private void changeCellValue(char[][] board)
    {
        int rows = board.length;
        int cols = board[0].length;

        for (int i = 1; i < rows - 1; i++)
        {
            for (int j = 1; j < cols - 1; j++)
            {
                if (board[i][j] == 'O')
                {
                    change(board, 'O', 'X', i, j);
                }
            }
        }
    }

    private void changeBoundaryValue(char[][] board)
    {
        int rows = board.length;
        int cols = board[0].length;
        for (int i = 0; i < rows; i++)
        {
            if (board[i][0] == 'O')
            {
                change(board, 'O', 'T', i, 0);
            }

            if (board[i][cols - 1] == 'O')
            {
                change(board, 'O', 'T', i, cols - 1);
            }
        }

        for (int j = 0; j < cols; j++)
        {
            if (board[0][j] == 'O')
            {
                change(board, 'O', 'T', 0, j);
            }

            if (board[rows - 1][j] == 'O')
            {
                change(board, 'O', 'T', rows - 1, j);
            }
        }
    }

    private void change(char[][] board, char targetValue, char toValue, int i, int j)
    {
        if (!isInBoard(board, i, j) || board[i][j] != targetValue)
        {
            return;
        }

        board[i][j] = toValue;
        change(board, targetValue, toValue, i - 1, j);
        change(board, targetValue, toValue, i + 1, j);
        change(board, targetValue, toValue, i, j - 1);
        change(board, targetValue, toValue, i, j + 1);
    }

    private boolean isInBoard(char[][] board, int i, int j)
    {
        return i >=0 && i < board.length && j >= 0 && j < board[0].length;
    }
}
```

## 改进版

上个版本中，在对边界的 `O` 区域设值 `T` 后，仔细观察二维数组中的值，可以发现剩下的 `O` 区域上的值必然会被变成  `X`，而 `T` 也必然会变成 `O`，所以，在对边界区域设值后，没有必要在搜索 `O` 的区域设值，只需要直接遍历二维数组，将每个 `O` 元素变成 `X`，将每个 `T` 元素变成 `O`。代码如下

```java
class Solution
{
    public void solve(char[][] board)
    {
        if (board == null || board.length == 0)
        {
            return;
        }

        changeBoundaryValue(board);
        
        for (int i = 0; i < board.length; i++)
        {
            for (int j = 0; j < board[0].length; j++)
            {
                if (board[i][j] == 'T')
                {
                    board[i][j] = 'O';
                }
                else if (board[i][j] == 'O')
                {
                    board[i][j] = 'X';
                }
            }
        }
    }

    private void changeBoundaryValue(char[][] board)
    {
        int rows = board.length;
        int cols = board[0].length;
        for (int i = 0; i < rows; i++)
        {
            if (board[i][0] == 'O')
            {
                change(board, i, 0);
            }

            if (board[i][cols - 1] == 'O')
            {
                change(board, i, cols - 1);
            }
        }

        for (int j = 0; j < cols; j++)
        {
            if (board[0][j] == 'O')
            {
                change(board, 0, j);
            }

            if (board[rows - 1][j] == 'O')
            {
                change(board, rows - 1, j);
            }
        }
    }

    private void change(char[][] board, int i, int j)
    {
        if (!isInBoard(board, i, j) || board[i][j] != 'O')
        {
            return;
        }

        board[i][j] = 'T';
        change(board, i - 1, j);
        change(board, i + 1, j);
        change(board, i, j - 1);
        change(board, i, j + 1);
    }

    private boolean isInBoard(char[][] board, int i, int j)
    {
        return i >=0 && i < board.length && j >= 0 && j < board[0].length;
    }
}
```

