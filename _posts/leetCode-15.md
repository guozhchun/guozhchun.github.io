---
title: leetCode-15:3Sum
date: 2018-06-01 22:37:34
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一个数组，要求找出其中三个数相加值为 0 的所有组合。需要注意的是，输出不能包括相同的三元数组。例如：[1, 0, -1]、[1, -1, 0]、[-1, 0, 1]、[-1, 1, 0]、[1, -1, 1]、[0, 1, -1]这几个属于相同的三元数组，只需要输出其中个任何一个即可。题目链接：**[点我](https://leetcode.com/problems/3sum/description/)**

<!-- more -->

# 样例输入输出

> 输入：[-1, 0, 1, 2, -1, -4]
>
> 输出：[[-1, 0, 1],  [-1, -1, 2]]

# 问题解法

此问题不能使用暴力的做法，这样至少需要三层循环，时间复杂度为`O(n^3)`，再加上对三元数组的去重，可能需要增加额外的循环导致增加额外的复杂度。即使将三元数组用一个类来表示，用set对其进行去重，整个求解过程也需要花费`O(n^3)`的时间复杂度，很容易超时。因此需要寻求另外的解法方法。

使用暴力主要是因为数组是无序的，如果数组是有序的，可以使用双头指针来减少一层循环。主要做法如下

* 对数组进行升序排序得到有序的数组
* 对数组进行遍历，此次遍历是获得第三元数组中的第一个数，在此层遍历中，需要跳过已经遍历过的相同的项，这样既不会产生相同的三元数组，更能节省不必要的循环
* 在内部循环中，定义首尾两个指针，依次判断首指针数、尾指针数、外层循环数三者之和是否为0。
  * 如果为0，则加入结果链表中。同时移动首尾指针跳过相同的项
  * 如果大于0，说明尾指针的数过大，要将尾指针向前移动一位，然后继续循环比较
  * 如果小于0，说明首指针的数过小，要将首指针向后移动一位，然后继续循环比较

此过程，排序时间复杂度为`O(nlogn)`，获取所有结果是两层循环，时间负责度为`O(n^2)`，总体时间复杂度为`O(n^2)`。代码如下

```java
class Solution 
{
    public List<List<Integer>> threeSum(int[] nums) 
    {
        if (nums == null || nums.length < 3)
        {
            return new ArrayList<>();
        }
        
        Arrays.sort(nums);
        int len = nums.length;
        List<List<Integer>> result = new ArrayList<>();
        for (int i = 0; i < len; i++)
        {
            // 跳过相同项
            if (i > 0 && nums[i] == nums[i - 1])
            {
                continue;
            }
            
            int start = i + 1;
            int end = len - 1;
            while (start < end)
            {
                if (nums[i] + nums[start] + nums[end] == 0)
                {
                    List<Integer> triplet = new ArrayList<>(3);
                    triplet.addAll(Arrays.asList(nums[i], nums[start], nums[end]));
                    result.add(triplet);
                    
                    // 跳过相同项
                    while (start < len - 1 && nums[start] == nums[start + 1])
                    {
                        start++;
                    }
                    
                    // 跳过相同项
                    while (end > 0 && nums[end] == nums[end - 1])
                    {
                        end--;
                    }
                    
                    start++;
                    end--;
                }
                else if (nums[i] + nums[start] + nums[end] > 0)
                {
                    end--;
                }
                else 
                {
                    start++;
                }
            }
        }
        
        return result;
    }
}
```

# 参考来源

1. [https://leetcode.com/problems/3sum/discuss/7380/Concise-O(N2)-Java-solution](https://leetcode.com/problems/3sum/discuss/7380/Concise-O(N2)-Java-solution)
2. [https://leetcode.com/problems/3sum/discuss/134544/my-solution-in-java-O(n2)](https://leetcode.com/problems/3sum/discuss/134544/my-solution-in-java-O(n2))