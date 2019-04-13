---
title: leetCode-12:Integer to Roman
date: 2019-03-10 18:22:47
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个整数（阿拉伯数字），要求将其转换罗马数字（字符串）。一般情况下，罗马数字是从左到右是按照从大到小的顺序进行分布的，除了少数几种特殊情况（IV、IX、XL、XC、CD、CM），这些数字的对应规则如下表所示，题目链接：**[点我](https://leetcode.com/problems/integer-to-roman/)**

<!-- more -->

| 罗马数字 | 阿拉伯数字 |
| -------- | ---------- |
| I        | 1          |
| V        | 5          |
| X        | 10         |
| L        | 50         |
| C        | 100        |
| D        | 500        |
| M        | 1000       |
| IV       | 4          |
| IX       | 9          |
| XL       | 40         |
| XC       | 90         |
| CD       | 400        |
| CM       | 900        |

# 样例输入输出

>输入：9
>
>输出：IX

> 输入：27
>
> 输出：XXVII
>
> 解释：X = 10, V = 5, I = 1 。XXVII = 10 + 10 + 5 + 1 + 1 = 27

> 输入：1994
>
> 输出：MCMXCIV
>
> 解释： M = 1000, CM = 900, XC = 90, IV = 4。MCMXCIV = 1000 + 900 + 90 + 4 = 1994

# 问题解法

从题目中分析可以得知，对于一个阿拉伯数字，罗马数字会先选择最大的数字字符（`M`）进行表示，不匹配后再选择第二大的数字字符进行匹配，依次类推，直到最后选择 `I` 为止。因此，题目的解法就按照这种规则进行模拟拼接每个匹配的字符串。

## if 分支判断法

```java
class Solution 
{
    public String intToRoman(int num) 
    {
        StringBuilder sb = new StringBuilder();
        
        // 1000
        int mCount = num / 1000;
        for (int i = 0; i < mCount; i++)
        {
            sb.append("M");
        }
        num = num % 1000;
        
        // 900
        if (num + 100 >= 1000)
        {
            sb.append("CM");
            num = num % 900;
        }
        
        // 500
        if (num >= 500)
        {
            sb.append("D");
            num = num % 500;
        }
        
        // 400
        if (num + 100 >= 500)
        {
            sb.append("CD");
            num = num % 400;
        }
        
        // 100
        int cCount = num / 100;
        for (int i = 0; i < cCount; i++)
        {
            sb.append("C");
        }
        num = num % 100;
        
        // 90
        if (num + 10 >= 100)
        {
            sb.append("XC");
            num = num % 90;
        }
        
        // 50
        if (num >= 50)
        {
            sb.append("L");
            num = num % 50;
        }
        
        // 40
        if (num + 10 >= 50)
        {
            sb.append("XL");
            num = num % 40;
        }
        
        // 10
        int xCount = num / 10;
        for (int i = 0; i < xCount; i++)
        {
            sb.append("X");
        }
        num = num % 10;
        
        // 9
        if (num == 9)
        {
            sb.append("IX");
            return sb.toString();
        }
        
        // 5
        if (num >= 5)
        {
            sb.append("V");
            num = num % 5;
        }
        
        // 4
        if (num == 4)
        {
            sb.append("IV");
            return sb.toString();
        }
        
        // 1
        for (int i = 0; i < num; i++)
        {
            sb.append("I");
        }
        
        return sb.toString();
    }
}
```

## 数组优化法

上述解决虽然可以解决问题，但是有太多 if 分支，代码不够简洁，可以用数组进行优化，此方法参考：[https://leetcode.com/problems/integer-to-roman/discuss/6310/My-java-solution-easy-to-understand](https://leetcode.com/problems/integer-to-roman/discuss/6310/My-java-solution-easy-to-understand)，优化后的代码如下

```java
class Solution 
{
    public String intToRoman(int num) 
    {
        int[] numValues = {1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1};
        String[] symbols = {"M", "CM", "D", "CD", "C", "XC", "L", "XL", "X", "IX", "V", "IV", "I"};
        
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < numValues.length; i++)
        {
            while (num >= numValues[i])
            {
                sb.append(symbols[i]);
                num -= numValues[i];
            }
        }
        
        return sb.toString();
    }
}
```

# 参考资料

1. [https://leetcode.com/problems/integer-to-roman/discuss/6310/My-java-solution-easy-to-understand](https://leetcode.com/problems/integer-to-roman/discuss/6310/My-java-solution-easy-to-understand)

