---
title: leetCode-7:Reverse Integer
date: 2018-05-28 22:07:09
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个整数，范围是`-2^31 ~ 2^32-1`，要求返回这个整数的逆序整数。如果得到的逆序整数超过整形的范围`-2^31 ~ 2^32-1`，则返回0。题目链接：**[点我](https://leetcode.com/problems/reverse-integer/description/)**

<!-- more -->

# 样例输入输出

> 输入：123
>
> 输出：321

> 输入：-123
>
> 输出：-321

> 输入：-1200
>
> 输出：-21

> 输入：1234567899
>
> 输出：0

# 问题解法

## 使用StringBuilder.reverse函数

java类库中给我们提供了很多的函数方便我们使用，其中`StringBuilder.reverse`函数是将字符串按顺序倒置，与这道题类似。因此可以将输入的整数转成StringBuilder，再使用reverse函数完成逆序，再转成int类型即可。

*注意：在转成 int 类型时，可能会超出 int 类型的范围而抛出异常，此时需要捕获异常并将值置为 0*

代码如下

```java
class Solution 
{
    public int reverse(int x) 
    {
        if (x < Integer.MIN_VALUE || x > Integer.MAX_VALUE) 
        {
            return 0;
        }
        
        int num = x > 0 ? x : -x;
        StringBuilder s = new StringBuilder(String.valueOf(num));
        int result;
        try
        {
            result = Integer.valueOf(s.reverse().toString());
        }
        catch (NumberFormatException e) 
        {
            result = 0;
        }
        result = x > 0 ? result : -result;
        
        return result;
    }
}
```

## 分拆数字逆序

上面使用try-catch来处理转换后的数字超过整形范围的情况，这类似于使用try-catch来代替 if 语句处理业务逻辑，这是不建议的。而且，上面使用库函数虽然比较方便，但是效率上却有点低，我们可以使用另外一种方式来提高一下效率。首先，将整数的每个数字取出放在数组中，然后将数字逆序拼接成新的整数，在拼接的过程中判断是否会超出整形范围，超出则返回0。代码如下

```java
class Solution 
{
    /**
     * 获取非负整数的逆序数组，返回10位数组（2 ^ 31 - 1 = 2147483647，10位数组足够），不足10位时补0
     * 前置条件：输入的数为非负整数
     * 例如：输入：0
     *     输出：[0,0,0,0,0,0,0,0,0,0]
     *     输入：123
     *     输出：[3,2,1,0,0,0,0,0,0,0]
     * @param x 非负整数
     * @return 10位长度的数组，表示x的逆序，不足10位时补0
     */
    private int[] getReverseNums(int x)
    {
        int[] nums = new int[10];
        int index = 0;
        while (x > 0)
        {
            nums[index] = x % 10;
            x /= 10;
            index++;
        }
        return nums;
    }
    
    public int reverse(int x) 
    {
        if (x < Integer.MIN_VALUE || x > Integer.MAX_VALUE) 
        {
            return 0;
        }
        
        int num = x > 0 ? x : -x;
        int[] nums = getReverseNums(num);
        int endIndex = 0;
        // find the end index
        for (int i = nums.length - 1; i >= 0; i--)
        {
            if (nums[i] != 0)
            {
                endIndex = i;
                break;
            }
        }
        
        int result = 0;
        for (int i = 0; i <= endIndex; i++)
        {
            // 先判断转成整数是否超出范围，否则溢出时数据发生改变
            // 虽然 -Integer.MIN_VALUE 比  Integer.MAX_VALUE 大 1，
            // 但是无法构造出result * 10 + nums[i] = 2147483648的值
            // 因为这样要求输入的值是-8463847412，超出了整形范围，在函数刚开始时就会进行判断返回0
            // 从另一个角度讲，逆序数无法达到整形的边界，因此此处可以直接用整形最大值进行判断是否越界
            if (result > (Integer.MAX_VALUE - nums[i]) / 10)
            {
                result = 0;
                break;
            }
            
            result = result * 10 + nums[i];
        }
        
        result = x > 0 ? result : -result;
        
        return result;
    }
}
```

