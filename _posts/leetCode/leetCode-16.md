---
title: leetCode-16:3Sum Closest
date: 2018-06-04 22:45:12
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一个无重复数字的整形数组和一个目标数字，要求找出数组中三个数字之和与目标数字最相近。题目链接：**[点我](https://leetcode.com/problems/3sum-closest/description/)**

<!-- more -->

# 样例输入输出

> 输入：数组：[-1, 2, 1, -4]，目标数字：1
>
> 输出：2
>
> 解释：(-1 + 2 + 1 = 2)

# 问题解法

## 暴力查询

此种方法最为简单，就是用三层循环，找出数组中三个数字的所有组合，并将组合的和与目标数字进行比较，将最相近的那项值输出。代码如下

```java
class Solution 
{
    public int threeSumClosest(int[] nums, int target) 
    {
        if (nums == null || nums.length == 0)
        {
            return 0;
        }
        
        if (nums.length == 1)
        {
            return nums[0];
        }
        
        if (nums.length == 2)
        {
            return nums[0] + nums[1];
        }
        
        int length = nums.length;
        int minVal = Integer.MAX_VALUE;
        int result = 0;
        for (int i = 0; i < length; i++)
        {
            for (int j = i + 1; j < length; j++)
            {
                for (int k = j + 1; k < length; k++)
                {
                    int sum = nums[i] + nums[j] + nums[k];
                    if (Math.abs(sum - target) < minVal)
                    {
                        minVal = Math.abs(sum - target);
                        result = sum;
                    }
                }
            }
        }
        
        return result;
    }
}
```

## 双头指针查询

使用暴力查询的方法虽然能够解决此题，但是时间复杂度过高，有`O(n^3)`，如果对数组进行排序，再使用双头指针进行遍历，则可以减少一层循环，使时间复杂度为`O(n^2)`。具体做法如下

* 对数组进行升序排序，获取有序数组
* 用一层循环遍历数组，获取三项中的第一项的值
* 在内部循环中，定义首尾两个指针，依次判断首指针数、尾指针数、外层循环数三者之和与目标数字之间的差距
  * 如果三者之和大于目标数字，则将尾指针向前移动一位，同时判断此时三者之和和目标数字之间的差距是否最小，如果最小，则记录此数。
  * 如果三者之和小于目标数字，则将首指针向后移动一位，同时判断此时三者之和和目标数字之间的差距是否最小，如果最小，则记录此数。
  * 如果三者之和等于目标数字，则目标数字就是要寻找的答案（因为相差为0），直接返回目标数字。

现在证明这种方法为什么有效。由于有一层循环对数组进行变量，每次循环取出的数组值作为三个数字中的第一个数字，因此只需要证明内部循环两个数字之和与目标数字和第一个数字之差之间的距离最小即可。由于在每层循环下，目标数字与第一个数字之差`m = |targetNum - firstNum|`是固定的，所以，只需要找出`nums[startIndex] + nums[endIndex] `与`m`的差距最小即可。如果内部循环采用两层循环变量数组取出所有的组合，无疑可以保证得到正确答案。而采用双头指针的方式进行循环，在`startIndex`和`endIndex`相互靠拢的过程中，两个指针产生的组合都会进行比较，假设最后到达`startIndex + k`的位子，此方式相比暴力查询的方式是少了对`nums[startIndex]`到`nums[startIndex + k - 1]`这中间的组合进行比较，也少了`nums[startIndex + k + 1]`到`nums[endIndex]`中间的组合进行比较。因此只需要证明`nums[startIndex]`到`nums[startIndex + k - 1]`中的任意两个数之和与`m`之间的差距大于`nums[startIndex] + nums[endIndex] `与`m`的差距，`nums[startIndex + k + 1]`到`nums[endIndex]`中的任意两个数之和与`m`直接的差距大于`nums[startIndex] + nums[endIndex] `与`m`的差距即可。

现在证明`nums[startIndex]`到`nums[startIndex + k - 1]`中的任意两个数之和与`m`之间的差距大于`nums[startIndex] + nums[endIndex] `与`m`的差距。假设a、b是`nums[startIndex]`到`nums[startIndex + k - 1]`中的任意两个数，则有`a < b < nums[startIndex] < nums[endIndex]`，且`a + nums[endIndex] < m`，`b + nums[endIndex] < m`，`a + b < m`。后面这三个表达式可以从移动首指针向后一位得出结论。因此只需要证明`m - a - b > |nums[startIndex] + nums[endIndex] - m|`

- 假设`nums[startIndex] + nums[endIndex] - m > 0`，则只需证明`m - a - b > nums[startIndex] + nums[endIndex] - m`。由于`a + nums[endIndex] < m`，`b + nums[endIndex] < m`，`nums[startIndex] < nums[endIndex]`，所以`a + b + nums[startIndex] + nums[endIndex] < a + b + nums[endIndex] + nums[endIndex] < m + m`，所以`m - a - b > nums[startIndex] + nums[endIndex] - m`得证。
- 假设`nums[startIndex] + nums[endIndex] - m < 0`，则只需证明`m - a - b > m - nums[startIndex] - nums[endIndex]`，由于`a < b < nums[startIndex] < nums[endIndex]`，所以`a + b < nums[startIndex] + nums[endIndex]`，所以`m - a - b > m - nums[startIndex] - nums[endIndex]`得证。

现在证明`nums[startIndex + k + 1]`到`nums[endIndex]`中的任意两个数之和与`m`直接的差距大于`nums[startIndex] + nums[endIndex] `与`m`的差。假设a、b是`nums[startIndex + k + 1]`到`nums[endIndex]`中的任意两个数，则有`nums[startIndex] < nums[endIndex] < a < b`，且`nums[startIndex] + a > m`， `nums[startIndex] + b > m`，`a + b > m`，后面这三个表达式可以从移动尾指针向前一位得出结论。因此只需要证明`a + b - m > |nums[startIndex] + nums[endIndex] - m|`

- 假设`nums[startIndex] + nums[endIndex] - m > 0`，则只需要证明`a + b - m > nums[startIndex] + nums[endIndex] - m`。由于`a > b > nums[endIndex] > nums[startIndex]`，因此`a + b > nums[startIndex] + nums[endIndex]`，所以`a + b - m > nums[startIndex] + nums[endIndex] - m `得证。
- 假设`nums[startIndex] + nums[endIndex] - m < 0`，则只需要证明`a + b - m > m - nums[startIndex] - nums[endIndex] `。由于`nums[startIndex] + a > m`， `nums[startIndex] + b > m`，`a + b > m`，所以`a + b + nums[startIndex] + nums[endIndex] > a + b + nums[startIndex] + nums[startIndex] > m + m`，所以`a + b - m > m - nums[startIndex] - nums[endIndex] `得证。

综上所述，此方法有效。

代码如下

```java
class Solution 
{
    public int threeSumClosest(int[] nums, int target) 
    {
        if (nums == null || nums.length == 0)
        {
            return 0;
        }
        
        if (nums.length == 1)
        {
            return nums[0];
        }
        
        if (nums.length == 2)
        {
            return nums[0] + nums[1];
        }
        
        Arrays.sort(nums);
        int length = nums.length;
        int minVal = Integer.MAX_VALUE;
        int result = 0;
        for (int i = 0; i < length; i++)
        {
            int startIndex = i + 1;
            int endIndex = length - 1;
            while (startIndex < endIndex)
            {
                int sum = nums[i] + nums[startIndex] + nums[endIndex];
                if (sum > target)
                {
                    endIndex--;
                    if (sum - target < minVal)
                    {
                        result = sum;
                        minVal = sum - target;
                    }
                }
                else if (sum < target)
                {
                    startIndex++;
                    if (target - sum < minVal)
                    {
                        result = sum;
                        minVal = target - sum;
                    }
                }
                else 
                {
                    return target;
                }
            }
        }
        
        return result;
    }
}
```

