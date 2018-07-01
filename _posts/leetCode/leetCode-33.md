---
title: leetCode-33:Search in Rotated Sorted Array
date: 2018-06-28 23:30:05
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个无重复整数的升序的循环数组（例如：[1,2,3,4,5] 或 [4,5,6,1,2,3]），要求以O(log n)的时间复杂度查找一个数字是否在这个数组中，如果存在，则返回该数字的下标，否则返回-1。题目链接：**[点我](https://leetcode.com/problems/search-in-rotated-sorted-array/description/)**

<!-- more -->

# 样例输入输出

> 输入：nums = [4,5,6,7,0,1,2], target = 0
>
> 输出：4

> 输入：nums = [1,2,3,4,5], target = 0
>
> 输出：-1

# 问题解法

此题要求以O(log n)的时间复杂度进行求解，只有二分查找才能满足要求。但是二分查找要求的前提条件是数组有序，而此题数组的“有序”是可循环的，即数组从某个位置开始向右遍历回到此位置，这之间经过的数字是有序的。因此，只需要求出数组最小值的位置，记录此偏移量，然后在二分查找的比较中，用计算得到的中间下标位置加上偏移量得到真正的数组的中间下标位置，然后跟目标值进行比较，其他步骤跟二分查找一样，即可得到问题的答案。代码如下

```java
class Solution 
{
    /**
     * 获取逆序位置的偏移量，当a[i] > a[i + 1]时，返回 i+1，否则返回0
     * 使用二分查找，时间复杂度为O(log n)
     * @param nums 数组
     * @return 逆序位置的偏移量
     */
    private int getOffsetNum(int[] nums)
    {
        int result = 0;
        int length = nums.length;
        int i = 0;
        int j = length - 1;
        
        while (i <= j)
        {
            int middle = (i + j) / 2;
            if (nums[middle] > nums[(middle + 1) % length])
            {
                result = middle + 1;
                break;
            }
            
            if (nums[middle] > nums[length - 1])
            {
                i = middle + 1;
            }
            else 
            {
                j = middle - 1;
            }
        }
        
        return result;
    }
    
    public int search(int[] nums, int target) 
    {
        int offsetNum = getOffsetNum(nums);
        int i = 0;
        int j = nums.length - 1;
        int result = -1;
        while (i <= j)
        {
            int middle = (i + j) / 2;
            // 在循环的数组中，middle对应的真正下标
            int realMiddleIndex = (middle + offsetNum) % nums.length;
            if (nums[realMiddleIndex] == target)
            {
                result = realMiddleIndex;
                break;
            }
            
            if (nums[realMiddleIndex] < target)
            {
                i = middle + 1;
            }
            else 
            {
                j = middle - 1;
            }
        }
        return result;
    }
}
```

