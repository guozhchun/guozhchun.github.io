---
title: leetCode-38:Count and Say
date: 2019-10-27 15:11:40
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个数字 `n (1 <= n <= 30)`，要求找出 `count-and-say` 序列中第 `n` 个位置的字符串。`count-and-say` 序列就是用一个数字字符串描述上一个数字字符串。比如：第一个字符串是 `1`，第二个字符串是 `11`，表示上一个字符串有 1 个 1，第三个字符串是 `21`，表示上一个字符串 `11` 有 2 个 1，第四个字符串是 `1211`，表示上一个字符串 `21` 有 1 个 2 和 1 个 1，第五个字符串是 `111221`，表示上一个字符串 `1211` 有 1 个1、1 个 2、2 个 1。题目链接：**[点我]( https://leetcode.com/problems/count-and-say/)**

<!-- more -->

# 样例输入输出

> 输入：1
>
> 输出：1

> 输入：10
>
> 输出：13211311123113112211

# 问题解法

直接按照题目描述的构造序列的方法从 1 构造到第 n 个序列即可。代码如下

```java
class Solution 
{
    public String countAndSay(int n)
    {
        StringBuilder sb = new StringBuilder();
        sb.append(1);
        for (int i = 2; i <= n; i++)
        {
            // dummy char, the flag which represents the char array is finish
            // and it is never equal to any actual useful char in array
            sb.append('#');
            char[] prevChars = sb.toString().toCharArray();
            sb = new StringBuilder();
            int count = 1;
            for (int j = 0; j < prevChars.length - 1; j++)
            {
                if (prevChars[j] == prevChars[j + 1])
                {
                    count++;
                }
                else
                {
                    sb.append(count).append(prevChars[j]);
                    count = 1;
                }
            }
        }
        
        return sb.toString();
    }
}
```

