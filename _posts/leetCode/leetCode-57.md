---
title: leetCode-57:Insert Interval
date: 2019-06-29 20:31:48
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个二维数组，表示区间的集合。数组中已经根据区间的起始下标从小到大排序，而且各个区间没有重叠部分。另外给定一个区间，要求将这个区间合并到原先的区间集合中。题目链接：**[点我](https://leetcode.com/problems/insert-interval/)**

<!-- more -->

# 样例输入输出

> 输入：[[1,3],[6,9]]          [2,5]
>
> 输出：[[1,5],[6,9]]

> 输入：[[1,2],[3,5],[6,7],[8,10],[12,16]]                 [4,8]
>
> 输出：[[1,2],[3,10],[12,16]]

# 问题解法

循环遍历原有的区间集合，对每个区间做如下判断

* 如果区间在合入的区间左侧，则保留此区间
* 如果区间与合入的区间左相交，则更新合入的区间的起始下标为当前区间的起始下标
* 如果区间包含要合入的区间，则返回原先的区间集合
* 如果区间在合入的区间内部，则跳过此区间，不做任何操作
* 如果区间与合入的区间右相交，则更新合入的区间的结束下标为当前区间的结束下标
* 如果区间在合入区间的右侧，则将合入的区间加入区间集合中，同时保留此区间及后续的区间，并结束循环

代码如下

```java
class Solution 
{
    public int[][] insert(int[][] intervals, int[] newInterval) 
    {
        int length = intervals.length;
        if (length == 0)
        {
            return new int[][]{newInterval};
        }
        
        if (newInterval.length == 0)
        {
            return intervals;
        }
        
        int[][] tempIntervals = new int[length + 1][2];
        int index = 0;
        int i = 0;
        for (i = 0; i < length; i++)
        {
            int[] interval = intervals[i];
            
            // the interval before the newInterval
            if (interval[1] < newInterval[0])
            {
                tempIntervals[index] = interval;
                index++;
                continue;
            }
            
            if (interval[0] <= newInterval[0])
            {
                // interval outer newInterval
                if (interval[1] >= newInterval[1])
                {
                    return intervals;
                }
                
                // interval left cross newInterval
                newInterval[0] = interval[0];
                continue;
            }
            
            // interval inner newInterval
            if (interval[0] > newInterval[0] && interval[1] <= newInterval[1])
            {
                continue;
            }
            
            if (interval[0] <= newInterval[1])
            {
                newInterval[1] = interval[1];
                continue;
            }
            
            tempIntervals[index] = new int[]{newInterval[0], newInterval[1]};
            index++;
            break;
        }
        
        if (i == length)
        {
            tempIntervals[index] = new int[]{newInterval[0], newInterval[1]};
            index++;
        }
        
        int[][] result = new int[length - i + index][2];
        System.arraycopy(tempIntervals, 0, result, 0, index);
        System.arraycopy(intervals, i, result, index, length - i);
        
        return result;
    }
}
```

