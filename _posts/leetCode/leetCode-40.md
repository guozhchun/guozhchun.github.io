---
title: leetCode-40:Combination Sum II
date: 2018-12-09 14:12:31
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个包含重复数字的正整数的数组和一个目标数字，要求找出所有由数组中元素构成的集合的和等于目标数字的列表，数组中的每个元素在同一个集合中最多只能出现一次，在结果列表中不能包含重复的集合。题目链接：**[点我](https://leetcode.com/problems/combination-sum-ii/)**

<!-- more -->

# 样例输入输出

> 输入：[1,1,1,2,2,3,3,3,4,4,4]   7
>
> 输出：[[1,1,1,2,2],[1,1,1,4],[1,1,2,3],[1,2,4],[1,3,3],[2,2,3],[3,4]]

> 输入：[10,1,2,7,6,1,5]             8
>
> 输出：[[1,1,6],[1,2,5],[1,7],[2,6]]

# 问题解法

此题跟[leetcode-39:Combination Sum](https://guozhchun.github.io/2018/12/09/leetCode/leetCode-39/)类似，不同的是原先数组中有重复元素，且数组中的元素在同一个构成集合中只能出现一次。此题仍然可以用[leetcode-39:Combination Sum](https://guozhchun.github.io/2018/12/09/leetCode/leetCode-39/)中的算法来求解。只是需要剔除重复的结果集。最简单直接的方法就是暴力搜索出所有的结果，然后去重，但是这样耗时太长。可以使用剪枝的做法：在每次递归中循环遍历数组元素时，如果当前数组元素跟前一个数组元素相同，则不用再进行遍历查找，因为前一个数组元素已经将所有情况都遍历过了。代码如下：

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
                return;
            }
            
            if (sum + candidates[i] > target)
            {
                break;
            }
            
            if (i > startIndex && candidates[i] == candidates[i - 1])
            {
                continue;
            }
            
            nums.add(candidates[i]);
            findSum(result, candidates, target, i + 1, nums, sum + candidates[i]);
            nums.remove(nums.size() - 1);
        }
    }
    
    public List<List<Integer>> combinationSum2(int[] candidates, int target) 
    {
        Arrays.sort(candidates);
        List<List<Integer>> result = new ArrayList<>();
        findSum(result, candidates, target, 0, new ArrayList<>(), 0);
        return result;
    }
}
```

