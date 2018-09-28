---
title: leetCode-4:Median of Two Sorted Arrays
date: 2018-09-24 18:32:36
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定两个有序的数组，要求找出合并后的数组的中位数，要求时间复杂度为 `O(log (m + n))`。题目链接：**[点我](https://leetcode.com/problems/median-of-two-sorted-arrays/description/)**

<!-- more -->

# 样例输入输出

> 输入：[1, 3], [2]
>
> 输出：2.0

> 输入：[1, 2], [3, 4]
>
> 输出：2.5

# 问题解法

刚开始看这题目时，很容易想到的是用归并的思想，分别从头遍历两个数组直到第 k 个元素，但是这样的时间复杂度是 `O(k)` ，这跟先使用归并算法再直接查找元素的时间复杂度 `O(m + n)` 查不多，但是都不满足题目的要求。题目要求的时间复杂度为 `O(log (m + n))`，这很容易想到二分查找，而此题也正是用二分查找来解决。

首先，将查找中位数的问题转换为查找两个有序数组中第 k 个元素的问题，接着，在每个数组中，分别查找第 k/2 个元素，将这两个元素进行比较，对于比较小的那个元素，其所在数组中的前面和自己这总共 k / 2 个元素必然包含在两个数组合并后的前 k 个元素之中，因此，在一轮进行查找时就可以跳过这部分元素，在剩下的元素中查找第 k/2 个元素，以此递归下去，直到最后一个元素为止。

此过程时间复杂度为 `O(log k)` 由于 k < m + n，所以满足题目的 `O(log (m + n))` 的要求。

代码如下

```java
class Solution 
{
    /**
     * 查找两个给定的有序数组合并后第 k 个元素，k 从 1 开始计数
     * @param a      第一个数组
     * @param aStart 第一个数组开始元素下标
     * @param b      第二个数组
     * @param bStart 第二个数组开始元素下标
     * @param k      查找第 k 个元素，从 1 开始计数
     * @return       给定数组合并后的第 k 个元素
     */
    private int findKthNum(int[] a, int aStart, int[] b, int bStart, int k)
    {
        if (aStart >= a.length)
        {
            return b[bStart + k - 1];
        }
        
        if (bStart >= b.length)
        {
            return a[aStart + k - 1];
        }
        
        if (k == 1)
        {
            return Math.min(a[aStart], b[bStart]);
        }
        
        // 分别获取两个数组中第 k/2 元素所在的下标，如果超过数组长度，则为该数组最后元素的下标
        int aKIndex = Math.min(aStart + k / 2  - 1, a.length - 1);
        int bKIndex = Math.min(bStart + k - k / 2 - 1, b.length - 1);
        
        // 第一个数组的第 k/2 个元素小于或等于第二个数组时，
        // 说明第一个数组的前 k/2 个元素必然包含在合并后的前 k 个元素之中，
        // 此时可以去除第一个数组的前 k/2 个元素，在剩下的元素中查找第 k/2 个元素。
        // 相反，如果第一个数组的第 k/2 个元素大于第二个数组时
        // 说明第二个数组的前 k/2 个元素必然包含在合并后的前 k 个元素之中，
        // 此时可以去除第二个数组的前 k/2 个元素，在剩下的元素中查找第 k/2 个元素。
        if (a[aKIndex] <= b[bKIndex])
        {
            return findKthNum(a, aKIndex + 1, b, bStart, k - (aKIndex - aStart + 1));
        }
        else
        {
            return findKthNum(a, aStart, b, bKIndex + 1, k - (bKIndex - bStart + 1));
        }
    }
    
    public double findMedianSortedArrays(int[] nums1, int[] nums2) 
    {
        if ((nums1.length + nums2.length) % 2 == 0)
        {
            int firstNum = findKthNum(nums1, 0, nums2, 0, (nums1.length + nums2.length) / 2);
            int secondNum = findKthNum(nums1, 0, nums2, 0, (nums1.length + nums2.length) / 2 + 1);
            return (double) (firstNum + secondNum) / 2;
        }
        else 
        {
            return (double) findKthNum(nums1, 0, nums2, 0, (nums1.length + nums2.length) / 2 + 1);
        }
    }
}
```

# 参考资料

1. 一个程序渣渣的小后院. 每天一道LeetCode-----两个有序数组合并后的第K个数[J/OL].https://blog.csdn.net/sinat_35261315/article/details/78249251, 2017-10-16