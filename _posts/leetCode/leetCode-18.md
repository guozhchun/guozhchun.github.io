---
title: leetCode-18:4Sum
date: 2019-01-05 10:50:57
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个可能包含重复数字的数组和一个目标数字，要求找出所有由数组中的 4 个元素构成的集合的和等于目标数字。结果集中不能包含重复集合。例如：[1, -1, 0, 0]、[-1, 1, 0, 0]、[1, 0, 0, -1]、[1, 0, -1, 0]、[-1, 0, 0, 1]、[-1, 0, 1, 0]、[0, 0, 1, -1]、[0, 0, -1, 1]、[0, 1, 0, -1]、[0, -1, 0, 1]、[0, 1, -1, 0]、[0, -1, 1, 0] 这几个集合属于同一个集合，只需要输出其中一个即可。题目链接：**[点我](https://leetcode.com/problems/4sum/)**

<!-- more -->

# 样例输入输出

> 输入：[1, 0, -1, 0, -2, 2]     0

> 输出：[[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]

# 问题解法

此题跟 [leetCode-15:3sum](https://guozhchun.github.io/leetCode/leetCode-15/) 类似，只是需要在其外层多加一层循环遍历另一个数字即可。代码如下

```java
class Solution 
{
    public List<List<Integer>> fourSum(int[] nums, int target) 
    {
        List<List<Integer>> result = new ArrayList<>();
        if (nums == null || nums.length < 4)
        {
            return result;
        }
        
        Arrays.sort(nums);
        int length = nums.length;
        for (int i = 0; i < length; i++)
        {
            // 跳过重复的数字，避免产生重复组合
            if (i != 0 && nums[i] == nums[i - 1])
            {
                continue;
            }
            
            for (int j = i + 1; j < length; j++)
            {
                // 跳过重复的数字，避免产生重复组合
                if (j != i + 1 && nums[j] == nums[j - 1])
                {
                    continue;
                }
                
                int start = j + 1;
                int end = length - 1;
                while (start < end)
                {
                    if (nums[i] + nums[j] + nums[start] + nums[end] == target)
                    {
                        List<Integer> temp = new ArrayList<>();
                        temp.addAll(Arrays.asList(nums[i], nums[j], nums[start], nums[end]));
                        result.add(temp);
                        
                        // 跳过重复元素
                        while (start < end && nums[end] == nums[end - 1])
                        {
                            end--;
                        }
                        
                        while (start < end && nums[start] == nums[start + 1])
                        {
                            start++;
                        }
                        
                        start++;
                        end--;
                    }
                    else if (nums[i] + nums[j] + nums[start] + nums[end] < target)
                    {
                        start++;
                    }
                    else 
                    {
                        end--;
                    }
                }
            }
        }
        
        return result;
    }
}
```

