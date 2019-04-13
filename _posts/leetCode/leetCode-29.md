---
title: leetCode-29:Divide Two Integers
date: 2019-03-31 14:36:01
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个被除数和除数（都是 `int`  类型），要求在不使用除法、乘法、求余运算的前提下，算出商。如果得出的结果超过 `int` 类型，则返回 `-2147483648` （商小于 -2147483648 时）或 `2147483647` （商大于 2147483647 时）。题目链接：**[点我](<https://leetcode.com/problems/divide-two-integers/>)**

<!--more -->

# 样例输入输出

> 输入：10   3
>
> 输出：3

> 输入：-7    3
>
> 输出：-2

> 输入：-2147483648    1
>
> 输出：2147483647
>
> 解释：在数学上，-2147483648 / 1 = -2147483648，但是这个结果超过 int 类型的存储范围，所以返回结果是 int 类型的最大值 2147483647

# 问题解法

此问题要求不能使用除法、乘法和求余运算，因此最简单的做法就是暴力相减，从被除数中一直减去除数，直到被除数小于除数，记录相减的次数，即使商。但是，这种做法超时了。

根据题目的提示，可以使用二分查找的思路进行求解。当一个数 `A1` 除以 2 后，其结果 `A2` 小于被除数` B `时，说明 `A1` 除 `B` 的商是 1，如果 `A2` 大于被除数，说明 `A1 / B` 的商至少是 `A2 / B` 的商的两倍。顺着这个思路，可以一直将被除数除 2 后与除数进行比较，直到被除数小于除数时，记录相除的次数 n，此时可以根据相除的次数得到最小的商是 `2^(n - 1)`，同时将最开始的被除数减去 `除数 * 2^(n - 1)`得到新的被除数，将新的被除数与除数进行新的相除计算得到其结果，最后将这两个结果进行相加得到最终的商。由于整个过程是对 2 进行相除和相乘的操作，可以用相应的位运算进行代替，从而避免使用乘法和除法。

代码如下：

```java
class Solution 
{
    public int divide(int dividend, int divisor) 
    {
        // 先于 divisor == Integer.MIN_VALUE 的判断
        // 主要是为了避免 divisor 和 divisor 都是 Integer.MIN_VALUE 的情况
        if (dividend == divisor)
        {
            return 1;
        }
        
        // 先在此处进行判断，主要是为了后续方便对 divisor 转换成整数的操作
        if (divisor == Integer.MIN_VALUE)
        {
            return 0;
        }
        
        int count = 0;
        int sign = 1;
        // 确保除数大于 0
        if (divisor < 0)
        {
            sign = -sign;
            divisor = -divisor;
        }
      
        // 确保被除数大于 0
        if (dividend < 0)
        {
            sign = -sign;
            // 保证将被除数转为正数后不会超出整数的范围
            if (dividend == Integer.MIN_VALUE)
            {
                dividend += divisor;
                count++;
            }
            dividend = -dividend;
        }
        
        // 经过上述操作，已经将被除数和除数转成正数，此处进行正数相除操作
        int quotient = dividePositiveNum(dividend, divisor);
        // 确保结果不会超过整数的范围
        if (quotient > Integer.MAX_VALUE - count)
        {
            return sign > 0 ? Integer.MAX_VALUE : Integer.MIN_VALUE;
        }

        count = count + quotient;
        return sign > 0 ? count: -count;
    }
    
    private int dividePositiveNum(int dividend, int divisor)
    {
        if (dividend < divisor)
        {
            return 0;
        }
        
        if (dividend == divisor)
        {
            return 1;
        }
        
        // 此处初始化为 -1，避免后续对其进行减一操作
        // 程序运行到此处时， dividend 必然大于 divisor，则 powCount 必然大于等于 0
        int powCount = -1;
        int tempDividend = dividend;
        while (tempDividend > divisor)
        {
            tempDividend = tempDividend >> 1;
            powCount++;
        }

        int partQuotient = 1 << powCount;
        int partMultiNum = divisor << powCount;
        int remaindDividend = dividend - partMultiNum;
        return partQuotient + dividePositiveNum(remaindDividend, divisor);
    }
}
```

