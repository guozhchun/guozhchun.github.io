---
title: leetCode-45:Jump Game II
date: 2019-07-28 09:53:10
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个包含非负整数的数组，每个元素代表当前位置向右移动的最大距离，要求找出从第一个元素到最后一个元素之间最少需要经过几步跳转（题目保证数组中必然存在从第一个元素到最后一个元素的跳转路径）。题目链接：**[点我](https://leetcode.com/problems/jump-game-ii/)**

<!-- more -->

# 样例输入输出

> 输入：[2,3,1,1,4]
>
> 输出：2

> 输入：[2,4,1,1,2,0,5,1]
>
> 输出：4

# 问题解法

## 递归（Memory Limit Exceeded）

此题是要找出 `nums[0]` 到 `nums[length - 1]` 之间的最小跳转次数，可以转换成 `nums[0]` 到 `nums[k]` 的最小跳转次数加上 `nums[k]` 到 `nums[length - 1]` 之间的最小跳转次数 `step`，其中 k 取值范围是 `[0, length - 1]`，比较每次算出来的最小跳转次数 `step`，取其最小值就是 `nums[0]` 到 `nums[length - 1]` 之间的最小跳转次数。为了避免一些重复计算，用一个二维数组存储从起始位置到结束位置之间需要经过的最小跳转次数。代码如下

```java
class Solution 
{
    public int jump(int[] nums) 
    {
        if (nums == null || nums.length == 0)
        {
            return 0;
        }
        
        int length = nums.length;
        
        // steps 存储从起始位置到结束位置之间需要经过的最小跳转次数
        // steps[i][j] 表示从 i 到 j 需要经过的最小跳转次数，-1 表示不可达
        int[][] steps = new int[length][length];
        
        // 初始化
        for (int i = 0; i < length; i++)
        {
            for (int j = 1; j <= nums[i] && i + j < length; j++)
            {
                steps[i][i + j] = 1;
            }
        }
        
        jumpHelper(steps, 0, length - 1);
        
        return steps[0][length - 1];
    }
    
    /**
     * 计算从 from 到 to 之间需要经过的最小跳转次数
     * @param steps 存储从起始位置到结束位置之间需要经过的最小跳转次数
     *        steps[i][j] 表示从 i 到 j 需要经过的最小跳转次数，-1 表示不可达
     *        此数组的值会在本函数中发生改变
     * @param from 起始位置
     * @param to 结束位置
     * @return 从 from 到 to 之间需要经过的最小跳转次数
     */
    private int jumpHelper(int[][] steps, int from, int to)
    {
        if (from == to)
        {
            return 0;
        }
        
        if (steps[from][to] != 0)
        {
            return steps[from][to];
        }
        
        int step = -1;
        for (int i = from + 1; i < to; i++)
        {
            int left = jumpHelper(steps, from, i);
            if (left == -1)
            {
                continue;
            }
            
            int right = jumpHelper(steps, i, to);
            if (right == -1)
            {
                continue;
            }
            

            if (step == -1 || step > left + right)
            {
                step = left + right;
            }

        }

        steps[from][to] = step;
        return step;
    }
}
```

这种做法虽然可以得到正确答案，但是在输入数组长度较长时，会发生 `Memory Limit Exceeded`

## 动态规划

由于二维数组空间复杂度超了，所以尝试用一维数组来存储变量值。此时想到了动态规划。用 `steps[i]` 表示从第 `0` 个位置到第 `i` 个位置需要经过的最小跳转次数。然后循环变量原始数组，针对当前元素能跳转的距离，更新其能到达距离范围内的元素的最小跳转次数。最后返回 `step[length - 1]` 即是从第一个位置到最后一个位置所经过的最小跳转次数。代码如下

```java
class Solution 
{
    public int jump(int[] nums)
    {
        if (nums == null || nums.length == 0)
        {
            return 0;
        }
        
        int length = nums.length;
        
        // steps[i] 表示从 0 到 i 位置需要经过的最小跳转次数
        int[] steps = new int[length];
        for (int i = 0; i < length; i++)
        {
            for (int j = i + 1; j <= i + nums[i] && j < length; j++)
            {
                if (steps[j] == 0 || steps[j] > steps[i] + 1)
                {
                    steps[j] = steps[i] + 1;
                }
            }
        }
        
        return steps[length - 1];
    }
}
```

此解法虽然可以通过，但是耗时较长，用了 `337 ms`

## 贪心算法

此解法参考 [https://leetcode.com/problems/jump-game-ii/discuss/313805/java-Solution-with-comment-1ms-37.5mb-beat-100-both](https://leetcode.com/problems/jump-game-ii/discuss/313805/java-Solution-with-comment-1ms-37.5mb-beat-100-both)。主要是用**每次都选择下一次跳转能到达的最远距离来选择本次跳转应该到达的位置**的思想，这样就能使每次跳转都利益最大化从而达到最后的利益最大化。用一个变量来计数存储跳转的次数，该变量最后的值就是从第一个位置到最后一个位置跳转所需要的最小跳转次数。代码如下

```java
class Solution 
{
    public int jump(int[] nums) 
    {
        if (nums == null || nums.length <= 1)
        {
            return 0;
        }
        
        int fastest = nums[0];
        int step = 1;
        int start = 0;
        int end = fastest;
        while (end < nums.length - 1)
        {
            for (int i = start + 1; i <= end; i++)
            {
                if (i + nums[i] > fastest)
                {
                    fastest = i + nums[i];
                    start = i;
                }
            }
            
            end = fastest;
            step++;
        }
        
        return step;
    }
}
```

此解法性能比动态规划要好，耗时只有 `1 ms`

# 参考资料

[https://leetcode.com/problems/jump-game-ii/discuss/313805/java-Solution-with-comment-1ms-37.5mb-beat-100-both](https://leetcode.com/problems/jump-game-ii/discuss/313805/java-Solution-with-comment-1ms-37.5mb-beat-100-both)