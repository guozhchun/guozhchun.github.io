---
title: leetCode-44:Wildcard Matching
date: 2019-09-01 11:23:24
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个只保护小写字母的字符串和一个只包含小写字母和 `?`、`*` 的模式匹配串，要求判断该模式匹配串是否能匹配给出的字符串。题目链接：**[点我](https://leetcode.com/problems/wildcard-matching)**

<!-- more -->

# 样例输入输出

> 输入：s = "aabbccdd"  p = "a*d"
>
> 输出：true

> 输入：s = "acdcb" p = "a*c?b"
>
> 输出：false

# 问题解法

## 解法一：递归（超时）

直接模拟匹配，暴力搜索，不过搜索过程中有大量重复计算的，导致超时。

```java
class Solution 
{
    private boolean isMatch(char[] s, char[] p, int sIndex, int pIndex)
    {
        if (sIndex == s.length)
        {
            if (pIndex == p.length)
            {
                return true;
            }
            
            for (int i = pIndex; i < p.length; i++)
            {
                if (p[i] != '*')
                {
                    return false;
                }
            }
            
            return true;
        }
        
        if (sIndex > s.length || pIndex >= p.length)
        {
            return false;
        }

        if (Character.isLowerCase(p[pIndex]))
        {
            if (s[sIndex] != p[pIndex])
            {
                return false;
            }
            
            return isMatch(s, p, sIndex + 1, pIndex + 1);
        }
        
        if (p[pIndex] == '?')
        {
            return isMatch(s, p, sIndex + 1, pIndex + 1);
        }
        
        // p[pIndex] == '*'
        while (pIndex < p.length && p[pIndex] == '*')
        {
            pIndex++;
        }
        
        if (pIndex == p.length)
        {
            return true;
        }
        
        for (int i = sIndex; i < s.length; i++)
        {     
            if (isMatch(s, p, i, pIndex))
            {
                return true;
            }
        }
        
        return false;
    }
    
    public boolean isMatch(String s, String p) 
    {
        return isMatch(s.toCharArray(), p.toCharArray(), 0, 0);
    }
}
```

## 解法二：动态规划

由于递归存在大量重复计算超时了，所以考虑用动态规划实现。用 `dp[i][j]` 表示 `s` 串的前 `i` 个字符和 `p` 串的前 `j` 个字符是否匹配，匹配保存 `true`，不匹配保存 `false`。此时，`dp[i][j]` 可以从`dp[i - 1][j]`、`dp[i][j - 1]`、`dp[i - 1][j - 1]` 三个地方演变而来。如果从 `dp[i - 1][j]` 演变而来，则 ``p`` 串当前字符是 `*`，如果从 `dp[i][j - 1]` 演变而来，则 `p` 串当前字符是 `*`（ 也可以是 `p` 的前一个字符是 `*` 当前字符是任意字符，不过这种情况可以归到 `dp[i - 1][j - 1]` 演变而来的情况，此处就不做复杂化），如果是从 `dp[i - 1][j - 1]` 演变而来，则 `p` 的当前字符串是 `?` 或 `*`  或与 `s` 的当前字符相等。

动态规划方程为：`dp[i][j] = (((dp[i - 1][j] == true || dp[i][j - 1] == true) && p[j - 1] == '*') || (dp[i - 1][j - 1] == true && (s[i - 1] == p[j - 1] || p[j - 1] == '?' || p[j - 1] == '*')))`

代码如下

```java
class Solution 
{
    public boolean isMatch(String s, String p) 
    {
        int sLength = s.length();
        int pLength = p.length();
        boolean[][] dp = new boolean[sLength + 1][pLength + 1];
        
        dp[0][0] = true;
        for (int j = 1; j <= pLength; j++)
        {
            if (dp[0][j - 1] && p.charAt(j - 1) == '*')
            {
                dp[0][j] = true;
            }
            else
            {
                break;
            }
        }
        
        for (int i = 1; i <= sLength; i++)
        {
            for (int j = 1; j <= pLength; j++)
            {
                char pChar = p.charAt(j - 1);
                if ((dp[i][j - 1] || dp[i - 1][j]) && pChar == '*')
                {
                    dp[i][j] = true;
                    continue;
                }
                
                if (dp[i - 1][j - 1] && (pChar == '*' || pChar == '?' || pChar == s.charAt(i - 1)))
                {
                    dp[i][j] = true;
                    continue;
                }
                
                dp[i][j] = false;
            }
        }
        
        return dp[sLength][pLength];
    }
}
```

## 解法三：双指针

此解法参考 [https://www.iteye.com/blog/shmilyaw-hotmail-com-2154716](https://www.iteye.com/blog/shmilyaw-hotmail-com-2154716) 。主要是用两个指针分别指向两个字符串。遍历 `s` 字符串，遍历过程中与 `p` 字符串进行比较，如果遇到相同的字符串或者 `?` ，则分别将指针向后移动一位，如果遇到 `*`，则记录当前 `*` 的位置和当前匹配到 `s` 的位置（方便后续匹配失败可以从当前位置下一个位置继续开始匹配查找），继续向后匹配。如果发生不匹配的情况，则看之前是否存在 `*` ，如果存在，则从上一个记录的位置的下一个位置开始重新匹配，否则匹配失败。

代码如下 

```java
class Solution 
{
    public boolean isMatch(String s, String p) 
    {
        int sIndex = 0;
        int pIndex = 0;
        int startIndex = -1;
        int match = 0;
        while (sIndex < s.length())
        {
            if (pIndex < p.length() && (p.charAt(pIndex) == '?' || p.charAt(pIndex) == s.charAt(sIndex)))
            {
                sIndex++;
                pIndex++;
                continue;
            }
            
            if (pIndex < p.length() && p.charAt(pIndex) == '*')
            {
                startIndex = pIndex;
                match = sIndex;
                pIndex = startIndex + 1;
                continue;
            }
            
            if (startIndex != -1)
            {
                sIndex = match + 1;
                match++;
                pIndex = startIndex + 1;
                continue;
            }
            
            return false;
        }
        
        while (pIndex < p.length() && p.charAt(pIndex) == '*')
        {
            pIndex++;
        }
        
        return pIndex == p.length();
    }
}
```

# 参考资料

[1] frank-liu.leetcode: Wildcard Matching [J/OJ].https://www.iteye.com/blog/shmilyaw-hotmail-com-2154716 ,2014-11-12