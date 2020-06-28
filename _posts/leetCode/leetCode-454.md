---
title: leetCode-454:4Sum II
date: 2019-12-01 21:18:41
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定四个相同长度的整形数组，要求找出从每个数组中挑出一个数字，使得这四个数字的和为 0 的组合个数。题目链接：**[点我](https://leetcode.com/problems/4sum-ii)**

<!-- more -->

# 样例输入输出

> 输入：A = [1, 2]， B = [-2, -1]，C = [-1, 2]，D = [0, 2]
>
> 输出：2

> 输入：A = [1], B = [1], C = [1], D = [1]
>
> 输出：0

# 问题解法

此题最直接简单的想法就是用四层 for 循环，但是这样会超时。另一种可取的方法就是：使用 map 存储其中两个元素的和及其出现的次数，再把另外两个元素相加，将得到的结果取相反数后与 map 中的数字进行比较，如果存在该数字，则将 map 中该数字出现的次数加到最终结果中。这样就只需要两次两层 for 循环，时间复杂度由 `O(n^4)` 降为 `O(n^2)`，代码如下：

```java
class Solution
{
    public int fourSumCount(int[] A, int[] B, int[] C, int[] D)
    {
        Map<Integer, Integer> map = new HashMap<>();
        int len = A.length;
        for (int i = 0; i < len; i++)
        {
            for (int j = 0; j < len; j++)
            {
                Integer sum = A[i] + B[j];
                Integer count = map.getOrDefault(sum, 0) + 1;
                map.put(sum, count);
            }
        }
        
        int total = 0;
        for (int i = 0; i < len; i++)
        {
            for (int j = 0; j < len; j++)
            {
                int temSum = C[i] + D[j];
                total = total + map.getOrDefault(-temSum, 0);
            }
        }
        
        return total;
    }
}
```

