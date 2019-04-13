---
title: leetCode-13:Integer to Roman
date: 2019-03-10 18:56:12
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个罗马数字，要求将其转换成整数（阿拉伯数字）。这些数字的对应规则如下表所示，题目链接：**[点我](https://leetcode.com/problems/roman-to-integer/)**

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

> 输入：IX
>
> 输出：9

> 输入：XXVII
>
> 输出：27
>
> 解释：X = 10, V = 5, I = 1 。XXVII = 10 + 10 + 5 + 1 + 1 = 27

> 输入：MCMXCIV
>
> 输出：1994
>
> 解释： M = 1000, CM = 900, XC = 90, IV = 4。MCMXCIV = 1000 + 900 + 90 + 4 = 1994

# 问题解法

## 解法一

根据上述表格，从左到右依次遍历字符串，对字符进行匹配，遇到 `I、X、C` 这三个字符时，将其后面的字符也一起进行连接进行匹配。将每次匹配得到的数字进行相加即可得到最终结果。代码如下

```java
class Solution 
{
    public int romanToInt(String s) 
    {
        String symbols = s + "$";
        int length = symbols.length();
        int ans = 0;
        for (int i = 0; i < length - 1; i++)
        {
            char current = symbols.charAt(i);
            char next = symbols.charAt(i + 1);
            if (current == 'M')
            {
                ans += 1000;
                continue;
            }
            
            if (current == 'C')
            {
                if (next == 'M')
                {
                    ans += 900;
                    i++;
                    continue;
                }
                
                if (next == 'D')
                {
                    ans += 400;
                    i++;
                    continue;
                }
                
                ans += 100;
                continue;
            }
            
            if (current == 'D')
            {
                ans += 500;
                continue;
            }
            
            if (current == 'X')
            {
                if (next == 'C')
                {
                    ans += 90;
                    i++;
                    continue;
                }
                
                if (next == 'L')
                {
                    ans += 40;
                    i++;
                    continue;
                }
                
                ans += 10;
                continue;
            }
            
            if (current == 'L')
            {
                ans += 50;
                continue;
            }
            
            if (current == 'I')
            {
                if (next == 'X')
                {
                    ans += 9;
                    i++;
                    continue;
                }
                
                if (next == 'V')
                {
                    ans += 4;
                    i++;
                    continue;
                }
                
                ans += 1;
                continue;
            }
            
            if (current == 'V')
            {
                ans += 5;
                continue;
            }
        }
        
        return ans;
    }
}
```

## 解法二

此解法参考 [https://leetcode.com/problems/roman-to-integer/discuss/252172/Java-Easy-to-understand-(35ms)](https://leetcode.com/problems/roman-to-integer/discuss/252172/Java-Easy-to-understand-(35ms))。主要是利用了「单个罗马数字从右到左值依次递增，两个罗马数字右边数字值大于左边数字值，并且右边数字值减左边数字值的结果就是这两个罗马数字转换的结果」的特点进行求解。因此可以从右到左遍历字符串，对每个字符，将其对应的数字与上个字符对应的数字进行比较，如果当前数字大于或者等于右边的数字，说明这是一个罗马数字，直接将对应的数字加到结果中。如果当前数字小，说明这是由当前数字和右边的数字构成一个整体的罗马数字，由于上一个循环中已经将右边的数字加到结果中，因此，此时需要从结果中减去当前数字，才能保证由这两个罗马数字构成的整体的正确转换。代码如下

```java
class Solution 
{
    public int romanToInt(String s) 
    {
        Map<Character, Integer> map = new HashMap<>();
        map.put('I', 1);
        map.put('V', 5);
        map.put('X', 10);
        map.put('L', 50);
        map.put('C', 100);
        map.put('D', 500);
        map.put('M', 1000);
        
        int ans = 0;
        int postNum = 0;
        for (int i = s.length() - 1; i >= 0; i--)
        {
            int num = map.get(s.charAt(i));
            if (num >= postNum)
            {
                ans += num;
            }
            else
            {
                ans -= num;
            }
            
            postNum = num;
        }
        
        return ans;
    }
}
```



# 参考资料

1. [https://leetcode.com/problems/roman-to-integer/discuss/252172/Java-Easy-to-understand-(35ms)](https://leetcode.com/problems/roman-to-integer/discuss/252172/Java-Easy-to-understand-(35ms))