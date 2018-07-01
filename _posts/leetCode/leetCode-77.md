---
title: leetCode-77:Combinations
date: 2018-06-29 23:21:14
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出数字 n 和数字 k，要求在数字1...n中找出 k 个数字的组合。题目链接：**[点我](https://leetcode.com/problems/combinations/description/)**

<!-- more -->

# 样例输入输出

> 输入：n = 4, k = 2
>
> 输出：[[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]

# 问题解法

此题主要使用回溯算法进行求解。需要用 k 轮迭代来获取 k 个数字的值，在每轮迭代中，依次遍历数字 i~n （第一轮：遍历1~n，第二轮遍历2~n，第三轮遍历3~n，...），对每个数字加进列表中，如果列表中的个数刚好是k，则将这个列表放如结果集中。代码如下

```java
class Solution 
{
    public void helper(int n, int k, int start, List<Integer> items, List<List<Integer>> result)
    {
        if (items.size() == k)
        {
            List<Integer> tempItems = new ArrayList<>(items);
            result.add(tempItems);
            return;
        }
        
        for (int i = start; i <= n; i++)
        {
            items.add(i);
            helper(n, k, i + 1, items, result);
            items.remove(items.size() - 1);
        }
    }
    
    public List<List<Integer>> combine(int n, int k) 
    {
        List<List<Integer>> result = new ArrayList<>();
        if (n < k)
        {
            return result;
        }
        
        helper(n, k, 1, new ArrayList<>(), result);
        return result;
    }
}
```

