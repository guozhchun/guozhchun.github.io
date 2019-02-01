---
title: leetCode-17:Letter Combinations of a Phone Number
date: 2018-10-24 22:46:13
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个字符串，字符串中只包含数字 2 ~ 9，每个数字对应的字母如下所示，要求将给定的数字的字符串转成对应的字母字符串组合。题目链接：**[点我](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)**

> 2：a、b、c
>
> 3：d、e、f
>
> 4：g、h、i
>
> 5：j、k、l
>
> 6：m、n、o
>
> 7：p、q、r、s
>
> 8：t、u、v
>
> 9：w、x、y、z

<!-- more -->

# 样例输入输出

*注意：输出结果顺序可以不一致*

> 输入："23"
>
> 输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]

> 输入："32"
>
> 输出：["da","db","dc","ea","eb","ec","fa","fb","fc"]

# 问题解法

此题就是将各个数字对应的字母进行组合，输出所有可能的组合，直接进行模拟输出即可。代码如下

```java
class Solution 
{
    public List<String> letterCombinations(String digits) 
    {
        if (digits == null || digits.length() < 1)
        {
            return new ArrayList<>();
        }
        
        char[][] digitLetters = {{'a','b','c'},
                                 {'d','e','f'},
                                 {'g','h','i'},
                                 {'j','k','l'},
                                 {'m','n','o'},
                                 {'p','q','r','s'},
                                 {'t','u','v'},
                                 {'w','x','y','z'}};
        
        List<String> combinations = new ArrayList<>();
        int digit = digits.charAt(0) - '0';
        char[] letters = digitLetters[digit - 2];
        for (int j = 0; j < letters.length; j++)
        {
            combinations.add(String.valueOf(letters[j]));
        }
        
        int length = digits.length();
        for (int i = 1; i < length; i++)
        {
            digit = digits.charAt(i) - '0';
            letters = digitLetters[digit - 2];
            List<String> newCombinations = new ArrayList<>();
            for (String combination : combinations)
            {
                for (int j = 0; j < letters.length; j++)
                {
                    newCombinations.add(combination + letters[j]);
                }
            }
            combinations = newCombinations;
        }
        
        return combinations;
    }
}
```

