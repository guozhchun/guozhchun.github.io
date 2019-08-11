---
title: leetCode-30:Substring with Concatenation of All Words
date: 2019-04-17 21:26:39
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个字符串和有相同长度的字符串组成的字符串数组（数组元素允许重复），要求找出字符串中所有由该数组的字符串（允许乱序）拼接而成的子字符串的起始下标。题目链接：**[点我](<https://leetcode.com/problems/substring-with-concatenation-of-all-words/>)**

<!-- more -->

# 样例输入输出

> 输入："barfoothefoobarman"    ["foo","bar"]
>
> 输出：[0, 9]
>
> 解释： ["foo","bar"] 可以构成两个子字符串，"foobar" 和 "barfoo"，其中 "foobar" 出现在 "barfoothefoobarman" 的 9 ~ 14 （从 0 开始计算），"barfoo" 出现在 "barfoothefoobarman" 的 0 ~ 5 （从 0 开始计算）的位置，因此返回列表 [0, 9]

> 输入："wordgoodgoodgoodbestword"   ["word","good","best","word"]
>
> 输出：[]
>
> 解释： ["word","good","best","word"] 构成的所有的字符串均不是 "wordgoodgoodgoodbestword" 的子字符串，所以返回空的列表

# 问题解法

先用一个 map 将字符串数组中的字符串保存起来，然后遍历原始的字符串，截取固定长度（字符串数组中所有字符串的长度总和）的字符串，再从这个子字符串中依次取出每个 word 串，与 map 中保存的元素进行比较，如果不存在 map 中，说明截取的这个子字符串不满足条件，直接进入下一轮循环。当每个 word 串都存在 map 中并且每个 word 数量刚好与 map 中对应元素的数量相等时，说明截取的子字符串满足题目要求，此时记录截取的字符串的开始位置，继续下一轮循环。待循环结束时，返回每个下标组成的列表即可。代码如下

```java
class Solution 
{
    public List<Integer> findSubstring(String s, String[] words) 
    {
        List<Integer> result = new LinkedList<>();
        if (s == null || s.length() == 0 || words == null || words.length == 0)
        {
            return result;
        }
        
        Map<String, Integer> wordMap = new HashMap<>();
        for (String word : words)
        {
            Integer count = wordMap.getOrDefault(word, 0) + 1;
            wordMap.put(word, count);
        }
        
        int wordLength = words[0].length();
        for (int i = 0; i < s.length(); i++)
        {
            if (i + wordLength * words.length > s.length())
            {
                return result;
            }
            
            Map<String, Integer> workMapCopy = new HashMap<>(wordMap);
            for (int j = 0; j < words.length; j++)
            {
                String word = s.substring(i + j * wordLength, i + (j + 1) * wordLength);
                Integer count = workMapCopy.get(word);
                if (count == null)
                {
                    break;
                }
                
                if (count == 1)
                {
                    workMapCopy.remove(word);
                }
                else
                {
                    workMapCopy.put(word, count - 1);
                }
            }
            
            if (workMapCopy.isEmpty())
            {
                result.add(i);
            }
        }
        
        return result;
    }
}
```

