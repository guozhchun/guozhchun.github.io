---
title: leetCode-34:Find First and Last Position of Element in Sorted Array
date: 2018-09-28 22:11:12
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个升序数组和一个目标数字，要求找出目标数字在数组中的最先出现的位置和最后出现的位置。找不到则输出 [-1, -1]。要求时间复杂度近似于 `Olog(n)`。 题目链接：**[点我](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/description/)**

<!-- more -->

# 样例输入输出

> 输入：nums = [5,7,7,8,8,10]，target = 8
>
> 输出：[3,4]

> 输入：nums = [5,7,7,8,8,10]，target = 6
>
> 输出：[-1,-1]

# 问题解法

在一个有序数组中寻找一个数出现的起始位置和结束位置，最简单的做法就是循环遍历数组，记录最先出现的问题和最后出现的位置，但是这种做法时间复杂度为 `O(n)`，不满足题目的 `Olog(n)` 的要求，既然时间复杂度是 `Olog(n)`，那很容易想到用二分法来解决，再加上数组是升序排序的，因此，可以用类似二分搜索的方式来解决。

主要思路为：先用二分查找，找到目标数，然后在左侧数组范围内，继续用二分查找找到最先出现的目标数的下标，同样在右侧数组范围内使用二分查找找到最后出现的目标数的下标，这两个下标就是目标数在数组中出现的开始位置和结束位置。

代码如下：

```java
class Solution 
{
    // 寻找升序数组中 nums[end] 元素最先出现的位置，返回其下标
    private int findLeftIndex(int[] nums, int start, int end)
    {
        if (nums[start] == nums[end])
        {
            return start;
        }
        
        int target = nums[end];
        int mid = (start + end) / 2;
        if (nums[mid] < target)
        {
            return findLeftIndex(nums, mid + 1, end);
        }
        
        return findLeftIndex(nums, start, mid);
    }
    
    // 寻找升序数组中 nums[start] 元素最后出现的位置，返回其下标
    private int findRightIndex(int[] nums, int start, int end)
    {
        if (nums[start] == nums[end])
        {
            return end;
        }
        
        int target = nums[start];
        int mid = (start + end + 1) / 2;
        if (nums[mid] > target)
        {
            return findRightIndex(nums, start, mid - 1);
        }
        
        return findRightIndex(nums, mid, end);
    }
    
    public int[] searchRange(int[] nums, int target) 
    {
        int[] result = {-1, -1};
        if (nums == null || nums.length == 0 || nums[0] > target)
        {
            return result;
        }

        int i = 0;
        int j = nums.length - 1;
        int mid = (i + j) / 2;
        while (mid >= i && mid <= j)
        {
            if (nums[mid] > target)
            {
                j = mid - 1;
                mid = (i + j) / 2;
            }
            else if (nums[mid] < target)
            {
                i = mid + 1;
                mid = (i + j) / 2;
            }
            else 
            {
                result[0] = findLeftIndex(nums, i, mid);
                result[1] = findRightIndex(nums, mid, j);
                break;
            }
        }
        
        return result;
    }
}
```



