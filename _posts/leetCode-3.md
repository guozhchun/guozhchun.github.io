---
title: leetCode-3:Longest Substring Without Repeating Characters
date: 2018-05-29 22:42:29
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个字符串，找出其中不包含重复字符的连续最长子字符串的长度。题目链接：**[点我](https://leetcode.com/problems/longest-substring-without-repeating-characters/description/)**

<!-- more -->

# 样例输入输出

> 输入："abcabcbb"
>
> 输出：3

> 输入："pwwkew"
>
> 输出：3

> 输入："aaaaa"
>
> 输出：1

# 问题解法

## 暴力搜索

定义一个`HashSet`，用于存放已经遍历过的字符，然后用两层for循环，第一层for循环主要是遍历子字符串开始的位置，第二层for循环主要是遍历字符串结束的位置，在第二层循环中，因此判断结尾的字符是否已经出现过，如果出现过，则记录此时字符串的长度，退出循环，最后取出最长的长度返回。此算法时间复杂度为`O(n^2)`。代码实现如下

```java
class Solution 
{
    public int lengthOfLongestSubstring(String s) 
    {
        if (s == null || s.length() == 0)
        {
            return 0;
        }
        Set<Character> set = new HashSet<>();
        int result = 0;
        int count = 0;
        int length = s.length();
        for (int i = 0; i < length; i++)
        {
            set.clear();
            count = 0;
            for (int j = i; j < length; j++)
            {
                if (set.contains(s.charAt(j)))
                {
                    break;
                }
                else
                {
                    set.add(s.charAt(j));
                    count++;
                }
            }
            
            if (count > result)
            {
                result = count;
            }
        }

        return result;
    }
}
```

## 窗口移动法

此方法主要参考leetcode上文章的解法，详细参考点**[这里](https://leetcode.com/articles/longest-substring-without-repeating-characters/)**。

实现思路是定义一个可变的窗口在字符串上移动，刚开始时`startIndex = 0, endIndex = 0`，依次遍历字符串，依次增加`endIndex`的值（改变窗口的大小），在遍历的过程中如果没有遇到重复的字符，则结束返回长度，否则，此时窗口中最长的没有重复字符串的下标位置是`[startIndex ~ endIndex)`，据此可以计算出长度并与最大长度进行比较，然后改变窗口大小，此时需要改变`startIndex`的位置。如何确定`startIndex`的位置呢？取出上一个重复字符的下标位置`k`，判断这个位置是否在`[startIndex ~ endIndex)`中，如果在这个区间中，则改变`startIndex`的值为`k + 1`，否则不用改变。

至于为什么可以直接跳过那么多的字符到上一次重复字符的下一个位置呢（这里假设`startIndex <= k < endIndex`，关于`startIndex > k`的情况上面已经做出解释）？因为在`k`和`endIndex`位置的字符是相同的，所以以`[startIndex, k]`区间中的位置到`endIndex`的位置组成的字符串必然有重复的字符存在，此时可以转换为求`[startIndex, k -1]`组成的字符串的最大长度和`[k + 1, endIndex]`组成的字符串的最大长度，而`[startIndex, k -1]`组成的字符串的最大长度在前面的循环中已经求出，`[k + 1, endIndex]`组成的字符串的最大长度在下一次循环中可以求出。因此当有重复字符时，可以直接将窗口开始的位置跳到上一个重复字符出现的位置（先决条件：`startIndex <= k`）。

代码如下

```java
class Solution 
{
    public int lengthOfLongestSubstring(String s) 
	{
		if (s == null || s.length() == 0)
		{
			return 0;
		}
		
		int startIndex = 0;
		int result = 0;
		int count = 0;
		Map<Character, Integer> map = new HashMap<>();
		for (int i = 0; i < s.length(); i++)
		{
			// 当前字符已经存在时，假设存在的位置为k，则[startIndex, k]中的任何位置到i这个位置
			// 组成的字符串均有重复字符，不用比较，可以直接从k + 1的位置到i的位置组成的字符串进行比较
			// map.get(s.charAt(i)) >= startIndex 条件必须存在，否则字符串是tmmzuxt时
			// 输出是4而不是5，条件map.get(s.charAt(i)) >= startIndex是保证 
			// startIndex <= k <= i的
			if (map.containsKey(s.charAt(i)) && map.get(s.charAt(i)) >= startIndex)
			{
				count = i - startIndex;
				startIndex = map.get(s.charAt(i)) + 1;
			}
			else 
			{
				count = i - startIndex + 1;
			}
			map.put(s.charAt(i), i);
			if (count > result)
			{
				result = count;
			}
		}
		
		return result;
    }
}
```

