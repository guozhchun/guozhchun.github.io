---
title: leetCode-90:Subsets II
date: 2018-06-12 23:32:08
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一个包含重复数字的整形数组，要求找出数组组成的集合的所有子集合（幂集），并且所有子集中不能包含相同的子集。题目链接：**[点我](https://leetcode.com/problems/subsets-ii/description/)**

<!-- more -->

# 样例输出

> 输入：[4,4,4,1,4]
>
> 输出：[[],[1],[4],[4,4],[4,4,4],[4,4,4,4],[1,4],[1,4,4],[1,4,4,4],[1,4,4,4,4]]

# 问题解法

此题跟[78:Subsets](https://leetcode.com/problems/subsets/description/)类似，只不过这里输入的数组是包含重复数字的，需要过滤掉重复数字避免产生重复的子集合。具体做法也类似，只是在遍历数组的过程中判断每个数字的重复次数，对重复数字的增加，只需取出已有的子集合，然后对每个子集合放入重复的数字1次得到新的子集合，再对原先的每个子集合放入重复的数字2次得到新的子集合，...，再对原先的每个子集合放入重复的数字n次得到新的子集合，最后将这些子集合合并得到截止到当前数字组成的集合的所有子集合。具体过程如下：

* 对数组进行排序
* 遍历数组的每个数字，找出连续的相同的数字的个数 k
  * 让 j 从 1开始循环到 k
    * 取出原先已有的每个子集合，增加 j 个当前的数字到该子集合中，产生新集合
    * 将产生的新集合添加到结果集中

例如：输入[2,1,2]，其过程如下

* 首先产生空集合[]，此时结果集中是[]
* 对数组排序，产生新数组[1,2,2]
* 遍历数组
  * 第一个数字1，只有一个重复数字，此时取出原有的结果集[]，增加1次当前数字1产生新的子集合[1]，增加到结果集中，此时结果集[],[1]
  * 第二个数字2，有两个重复数字，此时取出原有的结果集[],[1]
    * 对原有的结果集中的每个集合，增加1次当前数字2产生新的子集合[2],[1,2]
    * 对原有的结果集中的每个集合，增加2此当前数字2产生新的子集合[2,2],[1,2,2]
    * 将新产生的集合增加到结果集中，此时结果集[],[1],[2],[1,2],[2,2],[1,2,2]

代码如下

```java
class Solution 
{
    public List<List<Integer>> subsetsWithDup(int[] nums) 
    {
        List<List<Integer>> result = new ArrayList<>();
        result.add(new LinkedList<>());
        if (nums == null || nums.length == 0)
        {
            return result;
        }
        
        // 排序
        Arrays.sort(nums);
        
        int i = 0;
        int len = nums.length;
        while (i < len)
        {
            // 计算出连续的重复数字个数
            int duplicateCount = 1;
            while (i < len - 1 && nums[i] == nums[i + 1])
            {
                i++;
                duplicateCount++;
            }

            // 将重复的数字与之前得到的子集合并得到新的子集
            // 将这两个子集合并得到截止到目前位置的集合的所有子集
            int size = result.size();
            for (int j = 0; j < size; j++)
            {
                List<Integer> tempList = new LinkedList<>(result.get(j));
                for (int k = 0; k < duplicateCount; k++)
                {
                    List<Integer> newNumList = new LinkedList<>(tempList);
                    newNumList.add(nums[i]);
                    result.add(newNumList);
                    tempList = newNumList;
                }
            }
            
            i++;
        }
        
        return result;
    }
}
```

