---
title: leetCode-41:First Missing Positive
date: 2019-06-16 21:24:55
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个整形数组，要求用`O(n)`的时间复杂度和`O(1)`的空间复杂度，找出这些数字中缺少的第一个正整数。题目链接：**[点我](https://leetcode.com/problems/first-missing-positive)**

<!-- more -->

# 样例输入输出

> 输入：[1, 3, 2, 5]
>
> 输出：4

> 输入：[-1, 3, 2, 7, 3]
>
> 输出：1

# 问题解法

刚开始拿到这道题目，第一反映是对数组进行排序，然后遍历数组找出缺少的正整数。但是这样时间复杂度是`O(nlogn)`，并不满足题目要求。顺着这个思路，想到了计数排序的方法，可以把时间复杂度控制在`O(n)`，但是这样空间复杂度也需要`O(n)`，还是不满足要求。还是顺着计数排序的思路，只不过这次不新用数组存储，而是直接在原数组上进行操作。

首先遍历数组，对每个元素进行如下判断：按照计数排序的方法，此位置的元素是否准确，如果不准确，则将这个位置的元素与对应下标的元素交换，然后重复上述判断，直到遇到条件不满足（要交换的元素与当前元素值相同，当前元素值不在数组下标范围内）时结束。当循环结束时，判断当前位置的元素是否是下标值加一，如果不是，说明当前位置值不对，将其值设置为 0。待上述循环结束后，此时数组元素只有两种情况，位置正确的正整数和缺少对应正整数的 0。此时再用一个循环遍历，找到第一个 0 的位置，就是缺失的第一个正整数。

代码如下

```java
class Solution 
{
    public int firstMissingPositive(int[] nums) 
    {
        int length = nums.length;
        for (int i = 0; i < length; i++)
        {
            while (nums[i] > 0 && nums[i] <= length && nums[i] != nums[nums[i] - 1])
            {
                // 交换 i 和 nums[i] - 1 两个下标对应的元素
                int t = nums[i];
                nums[i] = nums[nums[i] - 1];
                nums[t - 1] = t;
            }
            
            if (nums[i] != i + 1)
            {
                nums[i] = 0;
            }
        }
        
        for (int i = 0; i < length; i++)
        {
            if (nums[i] == 0)
            {
                return i + 1;
            }
        }
        
        return length + 1;
    }
}
```

