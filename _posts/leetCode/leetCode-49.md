---
title: leetCode-49:Group Anagrams
date: 2018-12-15 21:57:04
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个字符串数组，要求将其进行分组，分组规则：同一个字符串所组成的字母相同则分为一组。题目链接：**[点我](https://leetcode.com/problems/group-anagrams/)**

<!-- more -->

# 样例输入输出

> 输入：["eat", "tea", "tan", "ate", "nat", "bat"]
>
> 输出：
>
> [
>   ["ate","eat","tea"],
>   ["nat","tan"],
>   ["bat"]
> ]

# 问题解法

首先用一个 map 来存储各个组别，其中 key 是组中的某个元素的字符排序得到的字符串，value 是组中的所有元素。然后遍历字符串数组，对每个字符串进行排序，将得到的结果放入 map 中对应的位置。最后将 map 中的所有 value 值取出构造成一个 list 对象返回。代码如下：

```java
class Solution 
{
    public List<List<String>> groupAnagrams(String[] strs) 
    {
        Map<String, List<String>> map = new HashMap<>();
        for (int i = 0; i < strs.length; i++)
        {
            char[] chars = strs[i].toCharArray();
            Arrays.sort(chars);
            String newStr = String.valueOf(chars);
            if (map.containsKey(newStr))
            {
                List<String> anagrams = map.get(newStr);
                anagrams.add(strs[i]);
            }
            else
            {
                List<String> anagrams = new LinkedList<>();
                anagrams.add(strs[i]);
                map.put(newStr, anagrams);
            }
        }
        
        return new LinkedList<>(map.values()) ;
    }
}
```

