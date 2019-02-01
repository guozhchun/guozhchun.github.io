---
title: leetCode-36:Valid Sudoku
date: 2018-11-25 17:52:07
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个 9 * 9 的数独棋盘（二维数组），其中只有数字 1~9 或字符 `.`，用字符 `.` 代表棋盘位置没有填充数字，要求判断给出的部分填充数字的数独棋盘是否是合法的。其合法性只针对填充的数字而言，未填充的区域不用判断，其要求同时满足以下几个条件：

* 每一行、每一列的数字没有重复
* 从左到右、从上到下，依次每个 3 * 3 的小矩阵内数字没有重复

题目链接：**[点我](https://leetcode.com/problems/valid-sudoku/)**

<!-- more -->

# 样例输入输出

> 输入：
>
> ```
> [
>   ["5","3",".",".","7",".",".",".","."],
>   ["6",".",".","1","9","5",".",".","."],
>   [".","9","8",".",".",".",".","6","."],
>   ["8",".",".",".","6",".",".",".","3"],
>   ["4",".",".","8",".","3",".",".","1"],
>   ["7",".",".",".","2",".",".",".","6"],
>   [".","6",".",".",".",".","2","8","."],
>   [".",".",".","4","1","9",".",".","5"],
>   [".",".",".",".","8",".",".","7","9"]
> ]
> ```
>
> 输出：true

> 输入：
>
> ```
> [
>   ["8","3",".",".","7",".",".",".","."],
>   ["6",".",".","1","9","5",".",".","."],
>   [".","9","8",".",".",".",".","6","."],
>   ["8",".",".",".","6",".",".",".","3"],
>   ["4",".",".","8",".","3",".",".","1"],
>   ["7",".",".",".","2",".",".",".","6"],
>   [".","6",".",".",".",".","2","8","."],
>   [".",".",".","4","1","9",".",".","5"],
>   [".",".",".",".","8",".",".","7","9"]
> ]
> ```
>
> 输出：false

# 问题解法

此题比较简单，直接按题目要求依次进行判断即可。代码如下

```java
class Solution 
{
    private void resetArray(boolean[] isUsed)
    {
        int length = isUsed.length;
        for (int i = 0; i < length; i++)
        {
            isUsed[i] = false;
        }
    }
    
    public boolean isValidSudoku(char[][] board) 
    {
        boolean[] isUsed = new boolean[9];
        
        // 校验每一行每一列
        for (int i = 0; i < 9; i++)
        {
            // 校验每一行
            resetArray(isUsed);
            for (int j = 0; j < 9; j++)
            {
                if (board[i][j] != '.')
                {
                    int num = board[i][j] - '0';
                    if (isUsed[num - 1])
                    {
                        return false;
                    }
                    
                    isUsed[num - 1] = true;
                }
            }
            
            // 校验每一列
            resetArray(isUsed);
            for (int j = 0; j < 9; j++)
            {
                if (board[j][i] != '.')
                {
                    int num = board[j][i] - '0';
                    if (isUsed[num - 1])
                    {
                        return false;
                    }
                    
                    isUsed[num - 1] = true;
                }
            }
        }
        
        // 校验每个 3 * 3 的小矩阵
        // row、col 表示横纵小矩阵的下标
        // i、j 表示小矩阵中元素的横纵下标
        for (int row = 0; row < 3; row++)
        {
            for (int col = 0; col < 3; col++)
            {
                resetArray(isUsed);
                for (int i = 0; i < 3; i++)
                {
                    for (int j = 0; j < 3; j++)
                    {
                        if (board[i + row * 3][j + col * 3] != '.')
                        {
                            int num = board[i + row * 3][j + col * 3] - '0';
                            if (isUsed[num - 1])
                            {
                                return false;
                            }
                            
                            isUsed[num - 1] = true;
                        }
                    }
                }
            }
        }
        
        return true;
    }
}
```

