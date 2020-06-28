---
title: leetCode-76:Minimum Window Substring
date: 2020-03-22 15:45:06
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定两个字符串 `s` 和 `t`，要求在 `s` 的连续子字符串列表中找出包含 `t` 字符串中的所有字符的最小连续子字符串。题目链接：**[点我]( https://leetcode.com/problems/minimum-window-substring)**

<!-- more -->

# 样例输入输出

> 输入： S = "ADOBECODEBANC", T = "ABC"
>
> 输出："BANC"

> 输入：S = "AA", T = "AA"
>
> 输出："AA"

# 问题解法

使用一个 map 存储 `t` 的字符以及字符出现的次数。再用两个指针分别指向 `s` 字符串，右边指针遍历 `s` 字符串，直到这两个指针围成的字符串包含了 `t` 中所有的字符，此时计算并更新字符串的长度及开始位置。然后移动左边的指针，依次缩小范围，看是否有更小的满足要求的字符串，如果有则更新结果。待左边指针紧靠右边指针时，继续移动右边指针，重复上述查找过程。代码如下

```java
class Solution 
{
    public String minWindow(String s, String t)
    {
        Map<Character, Integer> tmap = new HashMap<>();
        for (char ch : t.toCharArray())
        {
            putCharacter(tmap, ch);
        }
        
        Map<Character, Integer> map = new HashMap<>();
        int length = s.length();
        int minLength = -1;
        int windowStartIndex = 0;
        int begin = 0;
        int end = 0;
        
        while (end < length)
        {
            Character character = s.charAt(end);
            if (tmap.containsKey(character))
            {
                putCharacter(map, character);
                
                // 判断 map 中是否包含 tmap
                // 判断依据：map 包含 tmap 中所有的 key，并且每个key对应的value值，map不小于tmap
                if (isContaions(map, tmap))
                {
                    while (begin < end && needMove(map, tmap, s.charAt(begin)))
                    {
                        map.computeIfPresent(s.charAt(begin), (k, v) -> v - 1);
                        begin++;
                    }
                    
                    if (minLength == -1 || (end - begin + 1) < minLength)
                    {
                        minLength = end - begin + 1;
                        windowStartIndex = begin;
                    }
                }
            }
            else
            {
                if (minLength > 0 && end - begin + 1 > minLength)
                {
                    reduceCharacter(map, s.charAt(begin));
                    begin++;
                    while (begin < end && needMove(map, tmap, s.charAt(begin)))
                    {
                        map.computeIfPresent(s.charAt(begin), (k, v) -> v - 1);
                        begin++;
                    }
                }  
            }
            
            end++;
        }
        
        if (minLength == -1)
        {
            return "";
        }
        else
        {
            return s.substring(windowStartIndex, windowStartIndex + minLength);
        }
    }
    
    // 判断 sMap 是否包含 tMap
    // 判断依据： sMap中包含tMap中所有key，并且对于每个key，sMap的value值大于等于tMap的value值 
    private boolean isContaions(Map<Character, Integer> sMap, Map<Character, Integer> tMap)
    {
        for (Map.Entry<Character, Integer> entry : tMap.entrySet())
        {
            if (!sMap.containsKey(entry.getKey()))
            {
                return false;
            }
            
            if (sMap.get(entry.getKey()) < tMap.get(entry.getKey()))
            {
                return false;
            }
        }
        
        return true;
    }
    
    // 判断是否要移动当前指针，如果 sMap 或者 tMap 中不包含当前字符，或者 sMap 中对应 value 值大于 tMap，则需要移动当前指针
    private boolean needMove(Map<Character, Integer> sMap, Map<Character, Integer> tMap, Character character)
    {
        if (!tMap.containsKey(character))
        {
            return true;
        }
        
        if (!sMap.containsKey(character))
        {
            return true;
        }
        
        return sMap.get(character) > tMap.get(character);
    }

    // 增加当前字符计数，如果 map 中存在字符，则计数加一，否则当字符放入 map 中并设置 value 值为 1
    private void putCharacter(Map<Character, Integer> map, Character character)
    {
        if (map.containsKey(character))
        {
            map.compute(character, (k, v) -> v + 1);
        }
        else
        {
            map.put(character, 1);
        }
    }
    
    // 减少当前字符计数，如果 map 中存在当前字符并且对应的 value 值为 1，则移除当前字符
    private void reduceCharacter(Map<Character, Integer> map, Character character)
    {
        if (map.containsKey(character))
        {
            int num = map.get(character);
            if (num == 1)
            {
                map.remove(character);
            }
            else
            {
                map.put(character, num - 1);
            }
        }
    }
}
```

上述代码可以通过所有用例，但是耗时比较长，达到 200+ ms，我想主要是判断当前子字符串是否包含 `t` 中所有字符的操作（`isContaions` 循环比较 map，并且多次进行调用这个函数）比较耗时。针对此，可以优化判断条件，增加一个参数记录当前已经满足要求的字符的个数，用这个参数与 `t` 中不同的字符个数进行比较，可以避免多次循环。另一个优化点，是针对 `sMap`，不再判断当前字符在 `t` 中才放入 `map`，而是对所有的字符都直接放入 `map` 中，这样可以减少判断。此中做法主要参考： https://leetcode.com/articles/minimum-window-substring/ ，耗时 20+ ms。代码如下，

```java
class Solution 
{
    public String minWindow(String s, String t)
    {
        Map<Character, Integer> tMap = new HashMap<>();
        for (int i = 0; i < t.length(); i++)
        {
            Character character = t.charAt(i);
            int num = tMap.getOrDefault(character, 0);
            tMap.put(character, num + 1);
        }
        
        Map<Character, Integer> sMap = new HashMap<>();
        int begin = 0; 
        int end = 0;
        int windowBeginIndex = 0;
        int minLength = -1;
        int count = 0;   // 记录 [begin, end] 围起来的字符串中满足 t 中的非重复字符的个数
        while (end < s.length())
        {
            Character currentChar = s.charAt(end);
            int num = sMap.getOrDefault(currentChar, 0);
            sMap.put(currentChar, num + 1);
            // 注意：map.get() 获取的是 Integer，必须手动拆装成 int 进行数值的比较
            // 否则两个 Integer 进行 == 比较，可能会出现错误
            if (tMap.containsKey(currentChar) && tMap.get(currentChar).intValue() == sMap.get(currentChar).intValue())
            {
                count++;
            }
            
            while (begin <= end && count == tMap.size())
            {
                if (minLength == -1 || (end - begin + 1) < minLength)
                {
                    minLength = end - begin + 1;
                    windowBeginIndex = begin;
                }
                
                Character beginChar = s.charAt(begin);
                int temp = sMap.getOrDefault(beginChar, 0);
                sMap.put(beginChar, temp - 1);
                if (tMap.containsKey(beginChar) && tMap.get(beginChar).intValue() > sMap.get(beginChar).intValue())
                {
                    count--;
                }
                
                begin++;
            }
            
            end++;
        }
        
        if (minLength == -1)
        {
            return "";
        }
        else
        {
            return s.substring(windowBeginIndex, windowBeginIndex + minLength);
        }
    }
}
```

# 参考资料

 https://leetcode.com/articles/minimum-window-substring/ 