---
title: leetCode-62:Unique Paths
date: 2018-11-28 23:49:12
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个矩阵，一个机器人在矩阵左上角，目标在矩阵右下角，机器人每次只能向右或向下走一步，要求找出机器人从左上角走到右下角共有多少种走法。题目链接：**[点我](https://leetcode.com/problems/unique-paths/)**

<!-- more -->

# 样例输入输出

> 输入：3  7
>
> 输出：28

> 输入：51 9
>
> 输出：1916797311

# 问题解法

此题是数学中的组合题，直接运用组合进行求解即可。需要注意的是求解组合的方法，不能直接使用阶乘，否则大数的阶乘会超过 int 和 long 的存储范围导致错误。

还有一点需要额外注意：题目中说 m 和 n 最多只有 100，但是如果 m 和 n 是 100 的话，结果是 22750883079422934966181954039568885395604168260154104734000，这远远超出 int 的存储范围，但是题目中给出的函数返回值是 int 类型，所以**此处假设，输入的值计算得到的结果不会超过 int 类型的存储范围。**

代码如下

```java
class Solution 
{
    /**
     * 计算组合C(n,m)
     * @param n 组合的下标
     * @param m 组合的上标
     * @return n!/m!(n-m)!
     */
    private int calcCombination(int n, int m)
    {
        if (m <= 0)
        {
            return 1;
        }
        
        // 如果 m 值太大，利用 C(n, m) = C(n, n - m)的关系，将 m 值降下来
        // 既可以避免多次计算，也可以避免计算结果太大无法存储
        if (m * 2 > n)
        {
            m = n - m;
        }
        
        // 为了使计算结果能够存储，此处用 long 类型而不是 int 类型
        long a = n;
        long b = 1;
        for (int i = 1; i < m; i++)
        {
            a *= n - i;
            b *= i + 1;
            if (a % b == 0)
            {
                a = a / b;
                b = 1;
            }
        }

        return (int) (a / b);
    }
    
    public int uniquePaths(int m, int n) 
    {
        // 确保 C(n, m) 中，n 比 m 大
        if (m > n)
        {
            int temp = m;
            m = n;
            n = temp;
        }
        
        return calcCombination(m + n - 2, m - 1);
    }
}
```

