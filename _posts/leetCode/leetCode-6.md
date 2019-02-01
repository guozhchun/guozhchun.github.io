---
title: leetCode-6:ZigZag Conversion
date: 2018-12-30 13:55:35
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个字符串和数字 n ，要求将字符串按照 n 竖行 “Z”  字行分布后，将分布后的字符按照从上到下，从左到右的顺序构成的新字符串输出。题目链接：**[点我](https://leetcode.com/problems/zigzag-conversion/)**

<!-- more -->

# 样例输入输出

> 输入：PAYPALISHIRING      3
> 输出：PAHNAPLSIIGYIR
> 解释： 3 竖行 Z 字形排序后的图形如下
> ```
> P   A   H   N
> A P L S I I G
> Y   I   R
> ```

> 输入：PAYPALISHIRING    4
> 输出：PINALSIGYAHRPI
> 解释：4 竖行 Z 字形排序后的图形如下
> ```
> P     I    N
> A   L S  I G
> Y A   H R
> P     I
> ```

# 问题解法

此题就是模拟题，要找题目要求模拟输出即可。最直观的做法就是定义二维数组，然后按照 Z 形走法将原字符串填充到数组中，然后再遍历输出进行输出。但是这样稍微麻烦了点，所以寻找另一种做法：通过观察转换后的图形，可以得出以下规律：假设行数为 n

* 对于第一行，每两个字符之间的间隔是 2 * (n - 1)
* 对于第二行，令 k 为奇数，则 k + 1 为其相邻的偶数，则第 k 个字符与第 k + 1 个字符之间的间隔是 2 * (n - 2)，第 k + 1 个字符和第 k + 2 个字符之间的间隔是 2 * 1
* 对于第三行，令 k 为奇数，则 k + 1 为其相邻的偶数，则第 k 个字符与第 k + 1 个字符之间的间隔是 2 * (n - 3)，第 k + 1 个字符和第 k + 2 个字符之间的间隔是 2 * 2
* .......
* 对于最后一行，每两个字符之间的间隔是 2 * (n - 1)

根据以上规律，可以写出代码如下

```java
class Solution 
{
    public String convert(String s, int numRows) 
    {
        int length = s.length();
        if (length == 0 || length <= numRows || numRows < 2)
        {
            return s;
        }
        
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < numRows; i++)
        {
            sb.append(s.charAt(i));
            int index = i + 2 * (numRows - i - 1);
            // 最后一行，计算下一个字符，否则进入循环后会重复添加
            if (i + 1 == numRows)
            {
                index = i + 2 * i;
            }
            
            while (index < length)
            {
                sb.append(s.charAt(index));
                // 首行和尾行每次循环只有一个重复的字符，单独处理
                if (i == 0 || i + 1 == numRows)
                {
                    index = index + 2 * (numRows - 1);
                    continue;
                }

                // 除去首尾两行的其他中间行，每个循环，两个字符为一组
                int nextIndex = index + 2 * i;
                if (nextIndex < length)
                {
                    sb.append(s.charAt(nextIndex));
                }

                index = nextIndex + 2 * (numRows - i - 1);
            }
        }
        
        return sb.toString();
    }
}
```