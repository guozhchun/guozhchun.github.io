---
title: leetCode-65:Valid Number
date: 2019-02-17 14:11:01
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个字符串，判断这个字符串是否是数字（包括小数和指数）。题目没有明确说明什么情况下该字符串是合法的，需要靠自己猜测。大致的规则如下：

* 只能包含 `0~9`、`.`、`+`、`-`、`e`、`E`
* 首尾可以允许有空格，但是中间不允许有空格
* 只能出现一个小数点，且小数点在 e/E 的前面
* 只能出现一个 e/E，并且不能出现在第一位和最后一位
* +/- 只能出现在第一位或者 e/E 的后一位，后面必须有数字或小数点
* 允许小数点前没有数字，也允许小数点后没有数字，但是小数点前或后这两个位置至少要有一个位置有数字

<!-- more -->

# 样例输入输出

> 输入："   0.1   "
>
> 输出：true

> 输入："+.1"
>
> 输出：true

> 输入："3."
>
> 输出：true

> 输入："."
>
> 输出：false

> 输入："+E3"
>
> 输出：false

> 输入："-3.e-2"
>
> 输出：true

# 问题解法

## 使用正则

此题是要判断输入的字符串是否是一个数字，因此可以直接使用正则表达式进行全匹配。正则表达式为：`\\s*[-+]{0,1}((\\d*\\.){0,1}\\d+|\\d+\\.)([eE][-+]{0,1}\\d+){0,1}\\s*`。其中，前后两个 `\\s*` 是为了匹配前后的空格，`[-+]{0,1}` 用于匹配正负符号，`((\\d*\\.){0,1}\\d+|\\d+\\.)` 用于匹配整数和小数部分，`([eE][-+]{0,1}\\d+){0,1}` 用于匹配指数部分。代码如下

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;

class Solution 
{
    public boolean isNumber(String s) 
    {
        // 前后两个 \\s* 是去除前后的空格
        // ((\\d*\\.){0,1}\\d+|\\d+\\.) 匹配 3 或 3.3 或  3. 或  .3 的不带指数（e）的小数或整数
        // ([eE][-+]{0,1}\\d+){0,1} 匹配指数(e)部分的数字
        Pattern pattern = Pattern.compile("\\s*[-+]{0,1}((\\d*\\.){0,1}\\d+|\\d+\\.)([eE][-+]{0,1}\\d+){0,1}\\s*");
        Matcher matcher = pattern.matcher(s);
        return matcher.matches();
    }
}
```

## 遍历字符串

使用正则表达式进行求解虽然直观简洁，但是其速度比较慢。因此可以使用遍历字符串的方式，在遍历的过程中，对每个字符进行判断，遇到不符合规则的返回 false 结束遍历。此方式可以提供效率，但是其却比较复杂啰嗦，也很容易遗漏一些情况。其代码如下

```java
class Solution 
{
    // 判断字符是否是 0 ~ 9 的数字
    private boolean isCharNumber(char ch)
    {
        return ch >= '0' && ch <= '9';
    }
    
    public boolean isNumber(String s) 
    {
        if (s == null || s.length() == 0)
        {
            return false;
        }
        
        String str = s.trim();
        int length = str.length();
        if (length == 0 || (length == 1 && str.charAt(0) == '.'))
        {
            return false;
        }
        
        int decimalIndex = -1;     // 小数点的位置下标
        int exponentialIndex = -1; // e/E 的位置下标
        for (int i = 0; i < length; i++)
        {
            char ch = str.charAt(i);
            if (isCharNumber(ch))
            {
                continue;
            }
            
            // 小数点只能出现一次，且必须在 e 的前面
            if (ch == '.' && decimalIndex == -1 && exponentialIndex == -1)
            {
                // 小数点前或后至少要有一个数字
                if (i == 0 && !isCharNumber(str.charAt(i + 1)))
                {
                    return false;
                }
                
                if (i == length - 1 && !isCharNumber(str.charAt(i - 1)))
                {
                    return false;
                }
                
                if ((i != 0 && !isCharNumber(str.charAt(i - 1))) && (i != length - 1 && !isCharNumber(str.charAt(i + 1))))
                {
                    return false;
                }

                decimalIndex = i;
                continue;
            }
            
            // 符号只能出现在第一位或 e 的后面一位，且不能是最后一位
            // 由于初始化 exponentialIndex = -1，因此当出现在第一位时，exponentialIndex + 1 == 0 也能成立
            if ((ch == '+' || ch == '-') && exponentialIndex + 1 == i && i != length - 1)
            {
                // + - 后面必须有数字或小数点
                // 前面已经判断了 i != length - 1，此处不用再判断
                char nextChar = str.charAt(i + 1);
                if (!isCharNumber(nextChar) && nextChar != '.')
                {
                    return false;
                }
                continue;
            }
            
            // e 只能出现一次，并且不能出现在第一位和最后一位
            if ((ch == 'e' || ch == 'E') && exponentialIndex == -1 && i != 0 && i != length -1)
            {
                exponentialIndex = i;
                continue;
            }
            
            return false;
        }
        
        return true;
    }
}
```

