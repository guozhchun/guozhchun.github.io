---
title: leetCode-43:Multiply Strings
date: 2019-10-13 11:10:44
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定两个字符串数字（长度小于110），要求计算两个字符串数字相乘的结果。题目链接：**[点我](https://leetcode.com/problems/multiply-strings)**

<!-- more -->

# 样例输入输出

> 输入：num1 = "123"     num2 = "456"
>
> 输出："56088"

> 输入：num1 = "9999999999"  num2 = "9999999999"
>
> 输出："99999999980000000001"

# 问题解法

大数相乘的做法，直接模拟两个数字相乘即可。代码如下

```java
class Solution
{
    public String multiply(String num1, String num2) 
    {
        int[] sums = new int[num1.length() + num2.length()];
        for (int i = num1.length() - 1; i >= 0; i--)
        {
            for (int j = num2.length() - 1; j >= 0; j--)
            {
                int product = (num1.charAt(i) - '0') * (num2.charAt(j) - '0');
                int temp = sums[i + j + 1] + product;
                sums[i + j + 1] = temp % 10;
                sums[i + j] = sums[i + j] + temp / 10;
            }
        }
        
        boolean isStart = false;
        StringBuilder result = new StringBuilder();
        for (int i = 0; i < sums.length; i++)
        {
            if (sums[i] == 0 && !isStart)
            {
                continue;
            }
            
            isStart = true;
            result.append(sums[i]);
        }
        
        if (result.length() == 0)
        {
            return "0";
        }
        else 
        {
            return result.toString();
        }
    }
}
```

