---
title: leetCode-26:Remove Duplicates from Sorted Array
date: 2020-03-29 12:13:53
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个已经排序的数组，要求使用 `O(1)` 的空间复杂度剔除数组中的重复数字。题目链接：**[点我]( https://leetcode.com/problems/remove-duplicates-from-sorted-array)**

<!-- more -->

# 样例输入输出

> 输入：[1, 1, 2]
>
> 输出：[1, 2]

> 输入：[0, 0, 1, 1, 1, 2, 2, 3, 3, 4]
>
> 输出：[0, 1, 2, 3, 4]

# 问题解法

用一个下标记录变化后的数组的当前位置，遍历原有数组，如果数组当前元素和下标元素相同，则说明是重复元素，此时下标不用动，继续遍历下一个数组元素，如果两者不同，则移动下标到下一个位置，并将当前数组元素填充到下标的位置。代码如下

```java
class Solution 
{
    public int removeDuplicates(int[] nums)
    {
        if (nums == null || nums.length == 0)
        {
            return 0;
        }
        
        int index = 0;
        for (int i = 1; i < nums.length; i++)
        {
            if (nums[i] != nums[index])
            {
                index++;
                nums[index] = nums[i];
            }
        }
        
        return index + 1;
    }
}
```

