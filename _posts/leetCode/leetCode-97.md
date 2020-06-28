---
title: leetCode-97:Interleaving String
date: 2020-05-24 17:53:03
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定三个字符串 `s1`、`s2`、`s3`，要求判断 `s3` 是否能由 `s1` 和 `s2` 中的字符按顺序组成。题目链接：**[点我](https://leetcode.com/problems/interleaving-string)**

<!-- more -->

# 样例输入输出

> 输入：s1 = "aabcc", s2 = "dbbca", s3 = "aadbbcbcac"
>
> 输出：true

> 输入：s1 = "aabcc", s2 = "dbbca", s3 = "aadbbbaccc"
>
> 输出：false

# 问题解法

用深搜算法对 `s1`、`s2`、`s3` 的子字符串进行判断，为了避免中间大量的重复计算，用一个 `map` 来保存中间计算的结果。代码如下

```java
class Solution
{
    public boolean isInterleave(String s1, String s2, String s3)
    {
        return isInterleave(s1, s2, s3, new HashMap<>());
    }

    private boolean isInterleave(String s1, String s2, String s3, Map<String, Boolean> calcMap)
    {
        String temp = s1 + "#" + s2;
        if (calcMap.containsKey(temp))
        {
            return calcMap.get(temp);
        }

        boolean result = false;
        if (s1.length() == 0)
        {
            result = s2.equals(s3);
        }
        else if (s2.length() == 0)
        {
            result = s1.equals(s3);
        }
        else if (s1.charAt(0) == s2.charAt(0) && s1.charAt(0) == s3.charAt(0))
        {
            result = isInterleave(s1.substring(1), s2, s3.substring(1), calcMap) || isInterleave(s1, s2.substring(1), s3.substring(1), calcMap);
        }
        else if (s1.charAt(0) == s3.charAt(0))
        {
            result = isInterleave(s1.substring(1), s2, s3.substring(1), calcMap);
        }
        else if (s2.charAt(0) == s3.charAt(0))
        {
            result = isInterleave(s1, s2.substring(1), s3.substring(1), calcMap);
        }

        calcMap.put(temp, result);
        return result;
    }
}
```

