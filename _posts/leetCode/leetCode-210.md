---
title: leetCode-210:Course Schedule II
date: 2020-04-11 20:38:55
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一个数字表示课程的数量，再给出一个二维数组表示课程的依赖关系，如 [0, 1] 表示课程 1 必须在课程 0 之前完成。要求找出课程的先后学习顺序，如果给出的课程依赖关系不能存在循环依赖，则返回空数组。题目链接：**[点我]( https://leetcode.com/problems/course-schedule-ii/)**

<!-- more -->

# 样例输入输出

> 输入：2, [[1, 0]] 
>
> 输出：[0,1]

> 输入：2, [[0, 1], [1, 0]]
>
> 输出：[]
>

# 问题解法

直接用拓扑排序进行求解，将每次的学习的课程保存在数组中，最后判断是否存在循环依赖关系，如果存在则返回空数组，否则返回结果数组。代码如下

```java
class Solution
{
    public int[] findOrder(int numCourses, int[][] prerequisites)
    {
        int[] nodeIns = new int[numCourses];
        for (int i = 0; i < prerequisites.length; i++)
        {
            nodeIns[prerequisites[i][0]]++;
        }
        
        List<Integer> courses = new ArrayList<>();
        for (int i = 0; i < numCourses; i++)
        {
            if (nodeIns[i] == 0)
            {
                courses.add(i);
            }
        }
        
        int[] studyCoures = new int[numCourses];
        int index = 0;
        while (!courses.isEmpty())
        {
            int course = courses.get(0);
            courses.remove(0);
            studyCoures[index] = course;
            index++;
            for (int i = 0; i < prerequisites.length; i++)
            {
                if (prerequisites[i][1] == course)
                {
                    nodeIns[prerequisites[i][0]]--;
                    if (nodeIns[prerequisites[i][0]] == 0)
                    {
                        courses.add(prerequisites[i][0]);
                    }
                }
            }
        }
        
        if (index == numCourses)
        {
            return studyCoures;
        }
        else
        {
            return new int[0];
        }
    }
}
```

