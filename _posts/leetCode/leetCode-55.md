---
title: leetCode-55:Jump Game
date: 2019-01-13 00:04:42
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一个非负整数的数组，每个元素代表当前位置向右移动的最大距离，要求判断给出的数组中是否能从第一个元素走到最后一个元素，如果可以，返回 true，否则返回 false。题目链接：**[点我](https://leetcode.com/problems/jump-game/)**

<!-- more -->

# 样例输入输出

> 输入：[2,3,1,1,4]
>
> 输出：true

> 输入：[3,2,1,0,4]
>
> 输出：false

# 问题解法

此题可以用一个变量来表示从第一位到数组中当前某个元素中能达到的最大距离，然后遍历数组，实时更新这个最大距离，当发现这个距离小于数组下标（即不能从第一个元素到达当前元素）时，返回 false，否则继续循环直到结束。最后判断其最大长度是否超过数组长度，如果是，返回 true，否则返回 false。

代码如下：

```java
class Solution 
{
    public boolean canJump(int[] nums) 
    {
        if (nums == null || nums.length < 2)
        {
            return true;
        }
            
        int length = nums.length;
        int maxJumpLength = nums[0];
        for (int i = 1; i < length - 1; i++)
        {
            if (maxJumpLength < i)
            {
                return false;
            }
            
            if (i + nums[i] > maxJumpLength)
            {
                maxJumpLength = i + nums[i];
            }
        }
        
        return maxJumpLength + 1 >= length;
    }
}
```

