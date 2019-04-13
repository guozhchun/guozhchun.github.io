---
title: leetCode-8:String to Integer (atoi)
date: 2019-02-08 11:30:49
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个字符串，将其转换成整数。转换规则如下：

* 整数第一个字符必须是 `-` 或 `+` 或 `0-9` 的数字
* 整数前面允许有空格，但是不允许有其他的字符出现，如果出现其他字符，表明这个字符串无法转成整数，返回数字 0
* 在由一连串数字构成的整数后面，如果出现其他字符，则不用继续转换，只取第一个转换的整数返回即可
* 如果在转换的过程中，整数的范围超过 `Integer.MIN_VALUE ~ Integer.MAX_VALUE` ，则负数返回 `Integer_MIN_VALUE`，正数返回 `Integer_MAX_VALUE`

<!-- more -->

# 样例输入输出

> 输入："     -42abc"
>
> 输出：-42

> 输入："-123456789012345"
>
> 输出：-2147483648

> 输入："abc 1123"
>
> 输出：0

> 输入："123"
>
> 输出：123

# 问题解法

## 使用正则

使用正则表达式先将数字查找出来，然后使用 `Integer.parseInt` 函数将字符串转成数字，在转换的过程中，如果超过范围，则会抛出`NumberFormatException`，因此，只需要捕获 `NumberFormatException` 异常，并在异常处理中判断是返回最小值还是最大值即可。

此种方法相对而言比较简单，看起来也很清晰，不过在将字符串转成整形的过程中，使用了异常代码块进行正常业务逻辑的处理，这不太好。

代码如下

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;

class Solution 
{
    public int myAtoi(String str) 
    {
        if (str == null || str.length() == 0)
        {
            return 0;
        }
        
        String nums = str.trim();
        int length = nums.length();
        if (length < 1)
        {
            return 0;
        }
        
        Pattern pattern = Pattern.compile("[-+]{0,1}\\d+");
        Matcher matcher = pattern.matcher(nums);
        if (!matcher.find())
        {
            return 0;
        }
        
        String num = matcher.group();
        int startIndex = matcher.start();
        if (startIndex != 0)
        {
            return 0;
        }
        
        int result = 0;
        try
        {
            result = Integer.parseInt(num);
        }
        catch (NumberFormatException e)
        {
            if (num.charAt(0) == '-')
            {
                result = Integer.MIN_VALUE;
            }
            else
            {
                result = Integer.MAX_VALUE;
            }
        }
        
        return result;
    }
}
```

## 遍历字符串

使用正则虽然简单明了，但是却比较耗时，因此可以直接使用最原始的方法：遍历字符串。在遍历字符串的过程中同时将其转成整形。此种方法相对而言比较繁琐一点，但是耗时更少。代码如下

```java
class Solution 
{
    public int myAtoi(String str) 
    {
        if (str == null || str.length() == 0)
        {
            return 0;
        }
        
        int length = str.length();
        int beginIndex = 0;
        int sign = 1;
        for (int i = 0; i < length; i++)
        {
            char ch = str.charAt(i);
            if (ch == ' ')
            {
                continue;
            }
            
            if (ch == '-')
            {
                sign = -1;
                beginIndex = i + 1;
                break;
            }
            
            if (ch == '+')
            {
                beginIndex = i + 1;
                break;
            }
            
            if (ch >= '0' && ch <= '9')
            {
                beginIndex = i;
                break;
            }
            
            return 0;
        }
        
        int result = 0;
        for (int i = beginIndex; i < length; i++)
        {
            char ch = str.charAt(i);
            if (ch < '0' || ch > '9')
            {
                break;
            }
            
            if (sign == -1)
            {
                if ((Integer.MIN_VALUE + ch - '0') / 10 > result)
                {
                    result = Integer.MIN_VALUE;
                    break;
                }
            }
            else 
            {
                if ((Integer.MAX_VALUE - (ch - '0')) / 10 < result)
                {
                    result = Integer.MAX_VALUE;
                    break;
                }
            }
            result = result * 10 + sign * (ch - '0');
        }
        
        return result;
    }
}
```

