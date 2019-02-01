---
title: leetCode-39:Combination Sum
date: 2018-12-09 10:39:44
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个不包含重复数字的正整数的数组和一个目标数字，要求找出所有由数组中元素构成的集合的和等于目标数字的列表，数组中的元素可以重复出现在同一个集合中，但是结果列表中不能包含重复的集合。题目链接：**[点我](https://leetcode.com/problems/combination-sum/)**

<!-- more -->

# 样例输入输出

> 输入：[2,3,6,7]     7
>
> 输出：[[2,2,3], [7]]

> 输入：[1,2,3]       7
>
> 输出：[[1,1,1,1,1,1,1],[1,1,1,1,1,2],[1,1,1,1,3],[1,1,1,2,2],[1,1,2,3],[1,2,2,2],[1,3,3],[2,2,3]]

# 问题解法

此题比较简单明了，直接采用回溯算法进行求解即可。为了保证结果中不出现重复的集合，在递归搜索过程中，需要将数组元素依次向后搜索，而不能每次都从头开始搜。代码如下

```java
class Solution 
{
    private void findSum(List<List<Integer>> result, int[] candidates, int target, int startIndex, List<Integer> nums, int sum)
    {
        for (int i = startIndex; i < candidates.length; i++)
        {
            if (sum + candidates[i] == target)
            {
                List<Integer> temp = new ArrayList<>(nums);
                temp.add(candidates[i]);
                result.add(temp);
            }
            else if (sum + candidates[i] < target)
            {
                nums.add(candidates[i]);
                findSum(result, candidates, target, i, nums, sum + candidates[i]);
                nums.remove(nums.size() - 1);
            }
        }
    }
    
    public List<List<Integer>> combinationSum(int[] candidates, int target) 
    {
        List<List<Integer>> result = new ArrayList<>();
        findSum(result, candidates, target, 0, new ArrayList<>(), 0);
        return result;
    }
}
```

