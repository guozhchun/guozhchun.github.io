---
title: leetCode-31:Next Permutation
date: 2018-06-08 21:24:49
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一个数组，要求找出比这个数组组成的数字大的最小的那个数字组合，如果没有找到，则输出这个数组组成的数字的最小值的组合。题目链接：**[点我](https://leetcode.com/problems/next-permutation/description/)**

<!-- more -->

# 样例输入输出

由于函数没有返回值，是直接更改的输入的数组，因此此处的输出指更改后的数组的值

> 输入：[1,1,2]
>
> 输出：[1,2,1]

> 输入：[3,2,1]
>
> 输出：[1,2,3]

# 问题解法

此解法主要参考leetcode上文章的解法，详细参考点击**[这里](https://leetcode.com/articles/next-permutation/)**

主要过程如下：

* 先从数组右边向左边查找，找到右边数字比左边数字大的值，记录此时较小的值，如果没找到，则逆序整个数组，结束。
* 从数组右边向左边查找，找出第一个比上一步中找到的值大的值，交换这两个值。此时第一步中记录的值后面的值呈降序排列的状态（后面证明）。
* 将第一步找到的值后面的降序排列的值逆序，使其呈升序排列，此时得到结果。

其过程大致如下图所示：

![31_Next_Permutation.gif](/images/31_Next_Permutation.gif)

> 说明：此图片来自[https://leetcode.com/articles/next-permutation](https://leetcode.com/articles/next-permutation/)

现在证明在交换两个数之后，后面的数字是降序排列的。由于交换前，后面片段的是降序排列的，所以只需要证明交换后前后三个数是降序的即可。假设要将 `a[i]` 和 `a[j]` 进行交换，则必有以下不等式成立

```
a[j + 1] < a[i] < a[j]                 
a[j + 1] < a[j] < a[j - 1]         
```

现在要证明交换后 `a[j + 1] <= a[i] <= a[j - 1]`。用反证法进行证明。

先证明 `a[j + 1] <= a[i]`。

```
假设 a[j + 1] > a[i]
与已知 a[j + 1] < a[i] 矛盾
故 a[j + 1] <= a[i] 成立
```

现在证明`a[i] <= a[j - 1]`

```
假设 a[i] > a[j - 1]
由于 a[j + 1] < a[j] < a[j - 1]
所以 a[i] > a[j - 1] > a[j]，与已知 a[i] < a[j]矛盾
故 a[i] <= a[j - 1] 成立
```

综上所述，在交换值后，后面的值仍然维持者降序的顺序。

代码如下：

```java
class Solution 
{
    // 找到值比右边数字小的下标，找不到返回-1
    private int findLessIndex(int[] nums)
    {
        for (int i = nums.length - 1; i >= 1; i--)
        {
            if (nums[i] > nums[i - 1])
            {
                return i - 1;
            }
        }
        
        return -1;
    }
    
    // 找到要交换的下标：要求值比nums[index]大，找不到返回-1
    private int findSwapIndex(int[] nums, int index)
    {
        for (int i = nums.length - 1; i > index; i--)
        {
            if (nums[i] > nums[index])
            {
                return i;
            }
        }
        
        return -1;
    }
    
    // 交换数组下标i和j的值
    private void swap(int[] nums, int i, int j)
    {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    
    // 对数组[startIndex,endIndex]范围内的数字进行逆序
    private void reverse(int[] nums, int startIndex, int endIndex)
    {
        int endRange = (endIndex - startIndex + 1) / 2;
        for (int i = 0; i < endRange; i++)
        {
            swap(nums, startIndex + i, endIndex - i);
        }
    }
    
    public void nextPermutation(int[] nums) 
    {
        int lessIndex = findLessIndex(nums);
        // 该数字是最大的数，直接逆序后返回
        if (lessIndex == -1)
        {
            reverse(nums, 0, nums.length - 1);
            return;
        }
        
        int swapIndex = findSwapIndex(nums, lessIndex);
        // 没找到要交换的数字，说明lessIndex位置后面的数字都比这个数大，直接逆序后面的数字
        if (swapIndex == -1)
        {
            reverse(nums, lessIndex + 1, nums.length - 1);
            return;
        }
        
        // 找打要交换的数字，则交换，此时lessIndex后面的数字是倒序排序的，需要将其升序排序得到最小值
        swap(nums, lessIndex, swapIndex);
        reverse(nums, lessIndex + 1, nums.length - 1);
    }
}
```

# 参考来源

[https://leetcode.com/articles/next-permutation/](https://leetcode.com/articles/next-permutation/)