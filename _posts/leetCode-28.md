---
title: leetCode-28:Implement strStr()
date: 2018-05-27 23:07:41
tags: [leetCode, java, kmp]
categories: leetCode
---

# 问题描述

给出一个待匹配的字符串A和匹配的字符串B，要求判断B是否是A的子字符串。如果是则输出匹配的A字符串的起始下标，如果不是子字符串则输出-1。

*注意：如果B字符串是空的，返回 0 ，这与`String.indexOf`函数效果相同*

<!-- more -->

# 问题解法

## 使用java的String类函数

java库中的String类中已经有`indexOf`函数可以提供查找子字符串的功能，且其查询效果高效，一般情况下，都是使用此函数直接进行查找，无需自己重复造轮子，而且，即使自己造轮子也不一定比其高效。此实现方式代码如下：

```java
class Solution 
{
    public int strStr(String haystack, String needle) 
    {
        if (haystack == null) 
        {
            return -1;
        }
        
        return haystack.indexOf(needle);
    }
}
```

## 暴力查询

当没想到使用库函数时，第一反应是用暴力查询。即对遍历循环A字符串，在每个循环中依次遍历B字符串进行匹配，当全部匹配时，返回A字符串进行匹配的起始下标。循环结束后仍无匹配时返回-1。此种解法的时间复杂度是`O(n * m)`，其中 n 是A字符串长度，m 是B字符串长度。代码如下

```java
class Solution 
{
    public int strStr(String haystack, String needle) 
    {
        if (haystack == null || needle == null || needle.length() > haystack.length())
        {
            return -1;
        }
        
        if (needle.length() == 0)
        {
            return 0;
        }
        
        for (int i = 0; i < haystack.length(); i++)
        {
            int j = 0;
            for (; j < needle.length(); j++)
            {
                if (i + j >= haystack.length() || haystack.charAt(i + j) != needle.charAt(j))
                {
                    break;
                }
            }
            
            if (j == needle.length())
            {
                return i;
            }
        }
        
        return -1;
    }
}
```

 ## kmp算法

暴力查询的时间复杂度相对较高，在字符串比较长时效率低，因此可以使用kmp算法进行求解，此算法的时间复杂度为`O(m + n)`，其中 n 是A字符串长度，m 是B字符串长度。关于kmp算法可以参考**[这里](https://www.bilibili.com/video/av3246487/?from=search&seid=17173603269940723925)**。代码实现如下

```java
class Solution 
{
    public int[] getPartialMatchTable(String str)
    {
        if (str == null || str.length() == 0)
        {
            return null;
        }
        
        int[] partialMatchTable = new int[str.length()];
        int pointer = 0;
        partialMatchTable[0] = 0;
        for (int i = 1; i < str.length(); i++)
        {
            while (str.charAt(i) != str.charAt(pointer) && pointer > 0)
            {
                pointer = partialMatchTable[pointer - 1];
            }
            
            if (str.charAt(i) == str.charAt(pointer))
            {
                partialMatchTable[i] = pointer + 1;
                pointer++;
            }
            else
            {
                partialMatchTable[i] = 0;
            }
        }
        return partialMatchTable;
    }
    
    public int strStr(String haystack, String needle) 
    {
        if (haystack == null || needle == null)
        {
            return -1;
        }
        
        if (needle.length() == 0)
        {
            return 0;
        }
        
        int[] partialMatchTable = getPartialMatchTable(needle);
        int index = 0;
        int i = 0;
        while (i < haystack.length())
        {
            if (haystack.charAt(i) == needle.charAt(index))
            {
                if (index == needle.length() - 1)
                {
                    return i - index;
                }
                
                index++;
                i++;
            }
            else
            {
                if (index > 0)
                {
                    index = partialMatchTable[index - 1];
                }
                else
                {
                    i++;
                }
            }
        }
        
        return -1;
    }
}
```

