---
title: leetCode-37:Sudoku Solver
date: 2019-04-27 15:56:30
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个 9 * 9 的数独棋盘（二维数组），其中只有数字 1~9 或字符 `.`，用字符 `.` 代表棋盘位置没有填充数字，要求找出数独的解（题目保证每个输入只有唯一的解）。题目链接：**[点我](https://leetcode.com/problems/sudoku-solver/)**

<!-- more -->

# 样例输入输出

> 输入：
>
> 5 3 . . 7 . . . . 
> 6 . . 1 9 5 . . . 
> . 9 8 . . . . 6 . 
> 8 . . . 6 . . . 3 
> 4 . . 8 . 3 . . 1 
> 7 . . . 2 . . . 6 
> . 6 . . . . 2 8 . 
> . . . 4 1 9 . . 5 
> . . . . 8 . . 7 9
>
> 输出：
>
> 5 3 4 6 7 8 9 1 2 
> 6 7 2 1 9 5 3 4 8 
> 1 9 8 3 4 2 5 6 7 
> 8 5 9 7 6 1 4 2 3 
> 4 2 6 8 5 3 7 9 1 
> 7 1 3 9 2 4 8 5 6 
> 9 6 1 5 3 7 2 8 4 
> 2 8 7 4 1 9 6 3 5 
> 3 4 5 2 8 6 1 7 9 

# 问题解法

直接用递归回溯的方法进行求解，在递归过程中，每次填充数字前先判断要填充的数字是否合法，可以根据此进行剪枝，从而减少不必要的递归。代码如下

```java
class Solution 
{
    private static final int BOARD_SIZE = 9;
    
    private boolean isSudokuValid(char[][] board)
    {
        // 校验横、纵列数字是否合法
        for (int i = 0; i < BOARD_SIZE; i++)
        {
            boolean[] isRowCellUsed = new boolean[BOARD_SIZE];
            boolean[] isColCellUsed = new boolean[BOARD_SIZE];
            for (int j = 0; j < BOARD_SIZE; j++)
            {
                if (isRowCellUsed[board[i][j] - '1'])
                {
                    return false;
                }
                else
                {
                    isRowCellUsed[board[i][j] - '1'] = true;
                }
                
                if (isColCellUsed[board[j][i] - '1'])
                {
                    return false;
                }
                else
                {
                    isColCellUsed[board[j][i] - '1'] = true;
                }
            }
        }
        
        // 校验 3 * 3 矩阵内数字是否合法
        int blockSize = 3;
        for (int i = 0; i < blockSize; i++)
        {
            for (int j = 0; j < blockSize; j++)
            {
                boolean[] isBlockCellUsed = new boolean[BOARD_SIZE];
                for (int row = 0; row < blockSize; row++)
                {
                    for (int col = 0; col < blockSize; col++)
                    {
                        int cellNum = board[i * blockSize + row][j * blockSize + col] - '1';
                        if (isBlockCellUsed[cellNum])
                        {
                            return false;
                        }
                        else
                        {
                            isBlockCellUsed[cellNum] = true;
                        }
                    }
                }
            }
        }
        
        return true;
    }
    
    private boolean isCellNumValid(char[][] board, int rowIndex, int colIndex, int num)
    {
        char numChar = (char) (num + '0');
        for (int i = 0; i < BOARD_SIZE; i++)
        {
            // 横列中数字是否有重复
            if (board[rowIndex][i] == numChar)
            {
                return false;
            }
            
            // 纵列数字是否有重复
            if (board[i][colIndex] == numChar)
            {
                return false;
            }
        }
        
        // 3 * 3 矩阵内数字是否重复
        int rowBlockIndex = rowIndex / 3;
        int colBlockIndex = colIndex / 3;
        for (int i = 0; i < 3; i++)
        {
            for (int j = 0; j < 3; j++)
            {
                if (board[rowBlockIndex * 3 + i][colBlockIndex * 3 + j] == numChar)
                {
                    return false;
                }
            }
        }
        
        return true;
    }
    
    private boolean findSolveSudoku(char[][] board, int leftCell)
    {
        if (leftCell == 0)
        {
            return isSudokuValid(board);
        }
        
        for (int i = 0; i < BOARD_SIZE; i++)
        {
            for (int j = 0; j < BOARD_SIZE; j++)
            {
                if (board[i][j] == '.')
                {
                    for (int k = 1; k <= 9; k++)
                    {
                        if (isCellNumValid(board, i, j, k))
                        {
                            board[i][j] = (char) (k + '0');
                            boolean isFind = findSolveSudoku(board, leftCell - 1);
                            if (isFind)
                            {
                                return true;
                            }
                            board[i][j] = '.';
                        }
                    }
                    // 9 个数字都无法匹配时，说明此种情况下无解，不用再向后续进行匹配
                    return false;
                }
            }
        }
        
        return false;
    }

    public void solveSudoku(char[][] board) 
    {
        int leftCell = 0;
        for (int i = 0; i < BOARD_SIZE; i++)
        {
            for (int j = 0; j < BOARD_SIZE; j++)
            {
                if (board[i][j] == '.')
                {
                    leftCell++;
                }
            }
        }
        
        findSolveSudoku(board, leftCell);
    }
}
```

