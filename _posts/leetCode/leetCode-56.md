---
title: leetCode-56:Merge Intervals
date: 2019-02-01 22:32:18
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个包含多个区间的列表，要求将将重叠的区间进行合并，返回合并后的区间列表。题目链接：**[点我](https://leetcode.com/problems/merge-intervals/)**

<!-- more -->

# 样例输入输出

> 输入：[[1,3],[2,6],[8,10],[15,18]]
>
> 输出：[[1,6],[8,10],[15,18]]
>
> 解释：[1,3] 这个区间，结束的位置 3 在区间 [2, 6] 之间，因此可以把 [1, 3] 和 [2, 6] 两个区间合并成一个区间 [1, 6]

> 输入：[[1,4],[4,5]]
>
> 输出：[[1,5]]
>
> 解释：[1,4] 这个区间，结束的位置 4 在区间 [4, 5] 开始处，因此可以把 [1, 4] 和 [4, 5] 两个区间合并成一个区间 [1, 5]

# 问题解法

如果将所有的区间在直线上画出来进行表示，那么进行区间合并就会很直观，也很简单。按照此思路，可以先对列表中的所有区间进行排序，区间起始数小的排在前面。然后依次遍历排序后的区间列表，对相邻的区间进行合并，合并的原则是前一个区间的结束数字在后一个区间之间（包含区间的起始位置），则更新区间的结束位置。具体实现代码如下

```java
/**
 * Definition for an interval.
 * public class Interval {
 *     int start;
 *     int end;
 *     Interval() { start = 0; end = 0; }
 *     Interval(int s, int e) { start = s; end = e; }
 * }
 */
class Solution 
{
    private void sort(List<Interval> intervals)
    {
        intervals.sort((Interval a, Interval b) -> {
            if (a.start < b.start)
            {
                return -1;
            }
            
            if (a.start > b.start)
            {
                return 1;
            }
            
            return a.end - b.end;
        });
    }
    
    public List<Interval> merge(List<Interval> intervals) 
    {
        if (intervals == null || intervals.size() < 1)
        {
            return new ArrayList<>();
        }
        
        sort(intervals);
        List<Interval> result = new LinkedList<>();
        Interval current = intervals.get(0);
        result.add(current);
        for (Interval interval : intervals)
        {
            if (current.end < interval.start)
            {
                current = interval;
                result.add(current);
                continue;
            }
            
            if (current.end > interval.end)
            {
                continue;
            }
            
            current.end = interval.end;
        }
        return result;
    }
}
```

