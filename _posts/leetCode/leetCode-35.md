---
title: leetCode-35:Search Insert Position
date: 2020-05-01 00:47:30
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个排好序的升序整形数组和一个目标数字，要求找出这个数字在数组中的位置。如果数字没有出现在数组中，则返回数字应该插入数组的位置。题目链接：**[点我](https://leetcode.com/problems/search-insert-position)**

<!-- more -->

# 样例输入输出

> 输入：[1,3,5,6]   5
>
> 输出：2

> 输入：[1,3,5,6]   2
>
> 输出：1

# 问题解法

题目简单，直接遍历判断即可。代码如下

```java
class Solution
{
    public int searchInsert(int[] nums, int target) 
    {
        for (int i = 0; i < nums.length; i++)
        {
            if (nums[i] >= target)
            {
                return i;
            }
        }
        
        return nums.length;
    }
}
```

