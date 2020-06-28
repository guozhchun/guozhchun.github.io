---
title: leetCode-75:Sort Colors
date: 2019-10-20 15:59:44
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个只包括 `0、1、2` 三种数字的的数组，要求用一次循环迭代，使用常量空间对数组进行升序排序。题目链接：**[点我](https://leetcode.com/problems/sort-colors/)**

<!-- more -->

# 样例输入输出

> 输入：[2,0,2,1,1,0]
>
> 输出：[0,0,1,1,2,2]

> 输入：[2,0,1]
>
> 输出：[0,1,2]

# 问题解法

对于排序，如果采用常规做法，要么需要 `O(nlgn)` 的时间复杂度配合 `O(1)`的空间复杂度，要么需要 `O(1)` 的时间复杂度配合 `O(n)` 的空间复杂度，这都不满足题目的要求。鉴于排序中只有 `0、1、2` 三中数字，因此可以采用取巧的做法：用一个左指针指向数组从左到右第一个非 0 元素，用一个右指针指向数组从右到左第一个非 2 元素，从左到右遍历数组，如果当前元素是 0，则与左指针的元素交换值，并向右移动左指针，如果当前位置比右指针的值大，则交换这两个值，并移动右指针。代码如下

```java
class Solution
{
    public void sortColors(int[] nums)
    {
        int left = 0;
        int right = nums.length - 1;
        int i = 0;
        while (i <= right && left <= right)
        {
            if (nums[left] == 0)
            {
                left++;
                if (i < left)
                {
                    i = left;
                }
                continue;
            }
            
            if (nums[right] == 2)
            {
                right--;
                continue;
            }
            
            if (nums[i] == 0)
            {
                int temp = nums[i];
                nums[i] = nums[left];
                nums[left] = temp;
                left++;
            }
            else if (nums[i] > nums[right])
            {
                int temp = nums[i];
                nums[i] = nums[right];
                nums[right] = temp;
            }
            else
            {
                i++;
            }
        }
    }
}
```

