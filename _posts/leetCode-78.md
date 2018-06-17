---
title: leetCode-78:Subsets
date: 2018-06-11 23:02:48
tags: [leetCode, java]
categories: leetCode
---

 # 问题描述

  给出一个不包含重复数字的整形数组，要求找出这个数组的所有子集合（幂集）。题目链接：**[点我](https://leetcode.com/problems/subsets/description/)**

<!-- more -->

# 样例输入输出

> 输入：[1,2,3]
>
> 输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]

# 问题解法

此问题可以转换成以下的解法：求出[1,2,3,...n-1]的所有子集合，对这每个子集合都加上[n]这个数得到新的包含[n]的集合，将这些集合合并就是[1,2,3,...n]的所有子集合。从这个过程来看，是一个递归的过程，但是可以将其转成递推，主要做法是先放入一个空集合，然后对数组中的每个数，都取出数组前面的数的所有子集合，然后加上这个数得到新的集合，再将这些集合合并得到截至数组目前这个数的所有子集合。代码如下

```java
class Solution 
{
    public List<List<Integer>> subsets(int[] nums) 
    {
        List<List<Integer>> result = new ArrayList<>();
        result.add(new LinkedList<>());
        if (nums == null || nums.length == 0)
        {
            return result;
        }
        
        for (int i = 0; i < nums.length; i++)
        {
            int size = result.size();
            for (int j = 0; j < size; j++)
            {
                List<Integer> multiNumList = new LinkedList<>(result.get(j));
                multiNumList.add(nums[i]);
                result.add(multiNumList);
            }
        }
        
        return result;
    }
}
```

