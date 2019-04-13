---
title: leetCode-11:Container With Most Water
date: 2019-03-03 15:10:45
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个非负整数的数组，数组中每个元素的位置代表其在双平面坐标系下横坐标的值，元素的值代表其在双平面坐标系下纵坐标的值。这些元素在坐标系中组成了不同的柱子，要求找出由其中两个柱子所组成的长方形（长方形的宽度是两根柱子的距离，长度是两个柱子中最短柱子的值）的面积最大。题目链接：**[点我](https://leetcode.com/problems/container-with-most-water/)**

<!-- more -->

# 样例输入输出

> 输入：[1,8,6,2,5,4,8,3,7]
>
> 输出：49
>
> 解释：由 8 和 7 这两根柱子组成的范围，宽度是 7 （数组最后一个元素 7 和第二个元素 8 之间的距离），长度是 7 （7 和 8 中最小的数字），所以面积是 7 * 7 = 49
>
> ![示例图](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/07/17/question_11.jpg)
>
> 说明：此图来自于 leetCode 官网

# 问题解法

## 暴力遍历

既然是求任意两个柱子围成的面积的最大值，那么就可以将任意两个柱子围成的面积的所有值都求出，然后取其最大值。代码如下

```java
class Solution 
{
    public int maxArea(int[] height) 
    {
        int result = 0;
        int length = height.length;
        for (int i = 0; i < length; i++)
        {
            for (int j = i + 1; j < length; j++)
            {
                int minNum = Math.min(height[i], height[j]);
                int width = j - i;
                int area = minNum * width;
                if (result < area)
                {
                    result = area;
                }
            }
        }
        
        return result;
    }
}
```

## 暴力遍历改进版

直接暴力破解，耗时较长，有 200ms，所以需要进行改进。仔细观察柱子的分布情况，如果当前两个柱子构成了一个长方形，在将左侧柱子向右移动时，如果下一个柱子的高度比当前柱子的高度低，那么其构成的长方形的面积肯定不会比之前计算的要大。基于此，可以在每次移动左侧柱子的同时记录并比较柱子的高度，避免一些不必要的计算。代码如下

```java
class Solution 
{
    public int maxArea(int[] height) 
    {
        int result = 0;
        int currentMaxHeight = -1;
        int length = height.length;
        for (int i = 0; i < length; i++)
        {
            if (currentMaxHeight >= height[i])
            {
                continue;
            }
            
            currentMaxHeight = height[i];
            for (int j = i + 1; j < length; j++)
            {
                int minNum = Math.min(height[i], height[j]);
                int width = j - i;
                int area = minNum * width;
                if (result < area)
                {
                    result = area;
                }
            }
        }
        
        return result;
    }
}
```

## 双头指针解法

虽然第二版的暴力破解相比第一版进行了改进，使时间从 200ms 降到了 100 ms，但是仍然是比较耗时的，毕竟两者的时间复杂度都是 `O(n*n)`。因此可以使用双头指针的做法将时间复杂度降到 `O(n)`。具体做法是：定义一个指针指向数组头部，定义另一个指针指向数组尾部，然后计算这两根柱子构成的区域面积，每次移动指针时，将两个柱子中最短的那根柱子向中间进行移动（移动柱子时，长方形宽度变小，因此只能使其长度变大才有可能使面积变大，而只能使短柱子移动才有可能使其长度变大）。此方法参考[https://leetcode.com/articles/container-with-most-water/](https://leetcode.com/articles/container-with-most-water/)

代码如下

```java
class Solution 
{
    public int maxArea(int[] height)
    {
        int left = 0;
        int right = height.length - 1;
        int result = -1;
        while (left < right)
        {
            int area = Math.min(height[left], height[right]) * (right - left);
            if (area > result)
            {
                result = area;
            }
            
            if (height[left] < height[right])
            {
                left++;
            }
            else
            {
                right--;
            }
        }
        
        return result;
    }
}
```

# 参考来源

1. [https://leetcode.com/articles/container-with-most-water/](https://leetcode.com/articles/container-with-most-water/)

