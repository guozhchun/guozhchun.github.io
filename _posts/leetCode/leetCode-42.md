---
title: leetCode-42:Trapping Rain Water
date: 2019-07-31 22:31:46
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个非负整数的数组，每个数字代表柱子的长度，要求计算出这些柱子组成的容器可以存储的雨水量。题目链接：**[点我](https://leetcode.com/problems/trapping-rain-water/)**

<!-- more -->

# 样例输入输出

> 输入：[0,1,0,2,1,0,1,3,2,1,2,1]
>
> 输出：6

> 输入：[5,4,3,2,1,0,1,2,3,4,5]
>
> 输出：25

# 问题解法

循环遍历数组，分别计算每个位置能够存储的雨水量，将其相加得到最终答案。为了能得到每个位置能够存储的雨水量，可以用双指针的方式逐渐缩小范围。

* 开始时，两个指针分别在数组首尾处。

* 如果左边的柱子小于右边的柱子，则用另一个指针在两根柱子之间从左到右进行移动，如果当前位置柱子长度小于左边的柱子，则计算当前位置能够存储的雨水量，将其加入总的蓄水量中，否则将左边柱子的位置移动到当前位置。
* 如果左边的柱子大于右边的柱子，则用另一个指针在两根柱子之间从右到左进行移动，如果当前位置柱子长度小于右边的柱子，则计算当前位置能够存储的雨水量，将其加入总的蓄水量中，否则将右边柱子的位置移动到当前位置。
* 重复上述过程直到左右两个柱子相邻，此时的蓄水总量就是要求的答案。

代码如下

```java
class Solution 
{
    public int trap(int[] height) 
    {
        if (height == null || height.length <= 1)
        {
            return 0;
        }
        
        int left = 0;
        while (height[left] == 0)
        {
            left++;
        }
        
        int right = height.length - 1;
        while (height[right] == 0)
        {
            right--;
        }

        int sum = 0;
        while (left < right)
        {
            if (height[left] < height[right])
            {
                int index = left + 1;
                while (height[index] < height[left])
                {
                    sum += height[left] - height[index];
                    index++;
                }
                
                left = index;
            }
            else
            {
                int index = right - 1;
                while (height[index] < height[right])
                {
                    sum += height[right] - height[index];
                    index--;
                }
                
                right = index;
            }
        }
        
        return sum;
    }
}
```

