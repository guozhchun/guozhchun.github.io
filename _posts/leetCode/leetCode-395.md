---
title: leetCode-395:Longest Substring with At Least K Repeating Characters
date: 2018-06-05 22:55:31
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一个只包含小写字母的字符串和一个整数 k，要求找出字符串中的一个最长子字符串T，其中要求T中的每个字符出现的次数都必须大于等于 k。题目链接：**[点我](https://leetcode.com/problems/longest-substring-with-at-least-k-repeating-characters/description/)**

<!-- more -->

# 样例输入输出

> 输入：s = "aaabb", k = 3
>
> 输出：3
>
> 解释：返回的是`aaa`字符串的长度

> 输入：s = "ababbc", k = 2
>
> 输出：5
>
> 解释：返回的是`ababb`字符串的长度

# 问题解法

## 暴力查询-1（超时）

采用暴力的做法，需要将字符串的所有子字符串找出，这个过程需要`O(n^2)`时间复杂度，然后对每个字符串判断是否满足要求，这个过程需要`O(n)`，由于判断子字符串是否满足要求这个过程在查找全部子字符串的过程中，因此整个过程需要`O(n^3)`时间复杂度。这虽然简单，但是超时了。代码如下

```java
class Solution 
{
    private boolean isCharsNoLessThanK(String s, int k, int startIndex, int endIndex)
    {
        int[] charNums = new int[26];
        for (int i = startIndex; i <= endIndex; i++)
        {
            charNums[s.charAt(i) - 'a']++;
        }
        
        for (int i = startIndex; i <= endIndex; i++)
        {
            if (charNums[s.charAt(i) - 'a'] < k)
            {
                return false;
            }
        }
        
        return true;
    }
    
    public int longestSubstring(String s, int k)
    {
        if (s == null || k > s.length())
        {
            return 0;
        }
        
        if (k <= 1)
        {
            return s.length();
        }
        
        int length = s.length();
        int maxLength = 0;
        for (int i = 0; i < length; i++)
        {
            for (int j = i + 1; j < length; j++)
            {
                if (isCharsNoLessThanK(s, k, i, j))
                {
                    if (j - i + 1 > maxLength)
                    {
                        maxLength = j - i + 1;
                    }
                }
            }
        }
        
        return maxLength;
    }
}
```

## 暴力查询-2（超时）

上一种暴力查询的方法，由于把判断子字符串是否满足要求的循环放在了查找子字符串的循环中，导致多了一层循环，如果先找出所有的子字符串，再依次判断子字符串是否满足要求并找出满足要求的最长的子字符串的长度。这样可以将时间复杂度降为`O(n^2)`，可惜这样还是超时了。代码如下

```java
class Solution 
{
    private boolean isCharsNoLessThanK(String s, int k)
    {
        int[] charNums = new int[26];
        for (int i = 0; i < s.length(); i++)
        {
            charNums[s.charAt(i) - 'a']++;
        }
        
        for (int i = 0; i < s.length(); i++)
        {
            if (charNums[s.charAt(i) - 'a'] < k)
            {
                return false;
            }
        }
        
        return true;
    }
    
    public int longestSubstring(String s, int k)
    {
        if (s == null || k > s.length())
        {
            return 0;
        }
        
        if (k <= 1)
        {
            return s.length();
        }
        
        int length = s.length();
        List<String> subStrings = new ArrayList<>();
        for (int i = 0; i < length; i++)
        {
            for (int j = i + 1; j < length; j++)
            {
                subStrings.add(s.substring(i, j + 1));
            }
        }
        
        int maxLength = 0;
        for (String subString : subStrings)
        {
            if (isCharsNoLessThanK(subString, k) && maxLength < subString.length())
            {
                maxLength = subString.length();
            }
        }
        
        return maxLength;
    }
}
```

## 分治算法（4ms通过）

对于某个不满足要求的字符，暴力查询并不会对其进行过滤，仍然会产生许多包含此字符的子字符串并进行判断。这就导致了时间的耗费。而采用分治算法时，当发现某个字符不满足要求，则以此字符为分割点，对左边的字符串进行类似的递归查找，对右边的字符串也进行类似的递归查找。然后将这两个结果进行比较，将最大结果返回即可。此解法的时间复杂度为`O(nlogn)`。代码如下

```java
class Solution 
{
    private int longestSubstringLength(String string, int k, int startIndex, int endIndex)
    {
        // 由于此函数参数 k >= 2，因此当startIndex == endIndex时，也是不满足要求的
        if (startIndex >= endIndex)
        {
            return 0;
        }
        
        // 找出范围内的字母的数量
        // 如果取出全是小写字母的限制，此处可以改为Map
        int[] charNums = new int[26];
        for (int i = startIndex; i <= endIndex; i++)
        {
            int charIndex = string.charAt(i) - 'a';
            charNums[charIndex] += 1;
        }
        
        // 找到满足要求的起始下标
        while (startIndex <= endIndex && charNums[string.charAt(startIndex) - 'a'] < k)
        {
            startIndex++;
        }
        
        // 找到满足要求的结束下标
        while (startIndex <= endIndex && charNums[string.charAt(endIndex) - 'a'] < k)
        {
            endIndex--;
        }
        
        // 在起始下标和结束下标的范围内没有满足要求的字符串
        if (startIndex >= endIndex)
        {
            return 0;
        }
        
        // 查找在起始下标和结束下标的范围内是否有不满足要求的字符
        // 有则以此为界限，分别对左边的字符串和右边的字符串查找满足要求的子字符的长度
        // 然后比较获取最大的长度值作为返回结果
        // 没有则说明在起始下标和结束下标的范围内的字符都满足要求，直接返回结果
        int index = endIndex - 1;
        for (; index >= startIndex; index--)
        {
            if (charNums[string.charAt(index) - 'a'] < k)
            {
                break;
            }
        }
        
        // 在起始下标和结束下标之间的字符都满足要求，直接返回
        if (index == startIndex - 1)
        {
            return endIndex - startIndex + 1;
        }
        
        // 在起始下标和结束下标之间startIndex位置的字符不满足要求
        // 则从这里划分，分别找出左边子字符串和右边子字符串的满足要求的最长字符串的长度
        // 然后进行比较得到最终结果
        int leftResult = longestSubstringLength(string, k, startIndex, index - 1);
        int rightResult = longestSubstringLength(string, k, index + 1, endIndex);
        
        if (leftResult > rightResult)
        {
            return leftResult;
        }
        else 
        {
            return rightResult;
        }
    }
    
    public int longestSubstring(String s, int k) 
    {
        if (s == null || k > s.length())
        {
            return 0;
        }
        
        if (k <= 1)
        {
            return s.length();
        }
        
        return longestSubstringLength(s, k, 0, s.length() - 1);
    }
}
```

