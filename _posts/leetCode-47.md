---
title: leetCode-47:Permutations II
date: 2018-06-08 19:23:19
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一个整形数组，包含重复的数字，要求输出该数组数字的全排列，不能有全部相同的项出现。题目链接：**[点我](https://leetcode.com/problems/permutations-ii/description/)**

<!-- more -->

# 样例输入输出

> 输入：[2,2,1,1]
>
> 输出：[[2,2,1,1],[2,1,2,1],[2,1,1,2],[1,2,2,1],[1,2,1,2],[1,1,2,2]]

> 输入：[1,2,1]
>
> 输出：[[1,2,1],[1,1,2],[2,1,1]]

# 问题解法

## 暴力查找

此题跟查找非重复数字的全排列类似，如果使用暴力查找的方式，可以先按照非重复数字的全排列的方式找出所有的组合，在找到每个组合后，判断是否已经有重复的存在，存在则不添加到列表中，否则添加到列表中。代码如下

```java
class Solution 
{
    private List<Integer> getList(int[] nums)
    {
        List<Integer> result = new LinkedList<>();
        for (int i = 0; i < nums.length; i++)
        {
            result.add(nums[i]);
        }
        return result;
    }
    
    private void swap(int[] nums, int i, int j)
    {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    
    private void permuteUnique(int[] nums, List<List<Integer>> result, int startIndex, HashSet<String> set)
    {
        if (startIndex == nums.length - 1)
        {
            if (!set.contains(Arrays.toString(nums)))
            {
                set.add(Arrays.toString(nums));
                result.add(getList(nums));
            }
            return;
        }
        
        permuteUnique(nums, result, startIndex + 1, set);
        for (int i = startIndex + 1; i < nums.length; i++)
        {
            swap(nums, startIndex, i);
            permuteUnique(nums, result, startIndex + 1, set);
            swap(nums, startIndex, i);
        }
    }
    
    public List<List<Integer>> permuteUnique(int[] nums) 
    {
        List<List<Integer>> result = new LinkedList<List<Integer>>();
        if (nums == null || nums.length == 0)
        {
            result.add(new LinkedList<Integer>());
            return result;
        }
        HashSet<String> set = new HashSet<>();
        permuteUnique(nums, result, 0, set);
        return result;
    }
}
```

## 剪枝查找

暴力查找虽然能解法问题，但是效率太低了。问题在于对于重复数字，在第一次跟 i 元素交换后，其产生了一系列的组合，但是在后续遇到重复数字时，其仍会跟 i 进行交换，导致产生了一系列的重复组合。例如：对于[2,1,1]，在第一位数字2和第二位数字1交换后产生[1,2,1]，然后递归产生[1,1,2]，回溯后变回原先的组合[2,1,1]，此时如果将第一位数字2和第三为数字1交换，则会产生重复的组合[1,1,2]。因此在每次交换前先判断要交换的数字是否相同或之前已经交换过，如果交换的两个数字相同或者要交换的数字之前已经交换过，则不进行交换（因为这样会产生重复的组合），否则进行交换然后递归回溯。代码如下

```java
class Solution 
{
    private List<Integer> getList(int[] nums)
    {
        List<Integer> result = new LinkedList<>();
        for (int i = 0; i < nums.length; i++)
        {
            result.add(nums[i]);
        }
        return result;
    }
    
    private void swap(int[] nums, int i, int j)
    {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    
    private void permuteUnique(int[] nums, List<List<Integer>> result, int startIndex)
    {
        if (startIndex == nums.length - 1)
        {
            result.add(getList(nums));
            return;
        }
        
        permuteUnique(nums, result, startIndex + 1);
        
        // 存放交换过的数字，避免重复交换产生重复的结果
        // 只能放在每层函数调用里面，不能作为全局变量，否则影响外层函数的交换，导致最终少数据
        Set<Integer> usedNums = new HashSet<>();
        for (int i = startIndex + 1; i < nums.length; i++)
        {
            // 如果要交换的数字与当前数字相同，则不交换
            // 如果要交换的数字之前已经被交换过了，则不交换
            if (nums[i] != nums[startIndex] && !usedNums.contains(nums[i]))
            {
                swap(nums, startIndex, i);
                permuteUnique(nums, result, startIndex + 1);
                swap(nums, startIndex, i);
                
                usedNums.add(nums[i]);
            }
        }
    }
    
    public List<List<Integer>> permuteUnique(int[] nums) 
    {
        List<List<Integer>> result = new LinkedList<List<Integer>>();
        if (nums == null || nums.length == 0)
        {
            result.add(new LinkedList<Integer>());
            return result;
        }
        HashSet<String> set = new HashSet<>();
        permuteUnique(nums, result, 0);
        return result;
    }
}
```

