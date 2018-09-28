---
title: leetCode-207:Course Schedule
date: 2018-08-26 11:25:43
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一个数字表示课程的数量，再给出一个二维数组表示课程的依赖关系，如 [1, 2] 表示课程 1 必须在课程 2 之前完成。要求判断给出的课程数量和课程的依赖关系，能否顺利学完所有的课程。题目链接：**[点我](https://leetcode.com/problems/course-schedule/description/)**

<!-- more -->

# 样例输入输出

> 输入：2, [[1, 0]] 
>
> 输出：true

> 输入：2, [[0, 1], [1, 0]]
>
> 输出：false
>
> 解释：[0, 1] 表示课程 0 需要在课程 1 之前完成学习，[1, 0] 表示课程 1 需要在课程 0 之前完成学习，这使得课程 0 和课程 1 之间有个依赖循环，所以无法正常顺利完成所有的课程学习。

# 问题解法

课程排序问题是典型的拓扑排序问题的应用，所以这题直接使用拓扑排序进行求解即可。

## 普通版本

使用一个 map 来存储节点和这个节点的前向节点（依赖的节点），后续进行拓扑排序时，在队列中每取出一个入度为 0 的节点，则从 map 中的 value 值依赖列表中删除课程值，如果删除后依赖列表为空，则表明此时这个 map 的 key 值代表的课程没有其他依赖的课程，可以取出放入队列中（即，从 map 中删除此元素），在遍历完所有入度是 0 的节点后，如果 map 中仍然存在元素，则表明有课程存在循环依赖的关系，不能正常完成所有课程的学习。

```java
class Solution 
{
    public boolean canFinish(int numCourses, int[][] prerequisites) 
    {
        // 计算每个节点的入度
        Map<Integer, List<Integer>> nodeIn = new HashMap<>(numCourses);
        for (int i = 0; i < prerequisites.length; i++)
        {
            Integer from = prerequisites[i][0];
            Integer to = prerequisites[i][1];
            
            List<Integer> inList = nodeIn.getOrDefault(to, new LinkedList<>());
            inList.add(from);
            nodeIn.put(to, inList);
        }
        
        // 将入度为零的节点放入课程列表中
        List<Integer> courseList = new LinkedList<>();
        for (int i = 0; i < numCourses; i++)
        {
            Integer course = i;
            if (!nodeIn.containsKey(course))
            {
                courseList.add(course);
            }
        }
        
        // 依次取出课程列表中课程，并取出其他课程对此课程的依赖
        while (!courseList.isEmpty())
        {
            Integer course = courseList.remove(0);
            Iterator<Map.Entry<Integer, List<Integer>>> iterator = nodeIn.entrySet().iterator();
            while (iterator.hasNext())
            {
                Map.Entry<Integer, List<Integer>> entry = iterator.next();
                List<Integer> inList = entry.getValue();
                inList.remove(course);   // 去除其他课程对当前课程的依赖
                // 如果其他课程没有任何依赖的课程，则将其加入课程列表中，同时删除其依赖关系
                if (inList.isEmpty())
                {
                    courseList.add(entry.getKey());
                    iterator.remove();
                }
            }
        }
        
        // 在遍历完课程列表后，如果还存在依赖关系，则说明存在循环依赖
        return nodeIn.isEmpty();
    }
}
```

## 改进版本

此版本是对上一个版本的改进。主要是直接用一个数组来表示各个课程的节点入度数，替换掉 map 的表示方式。同时在进行拓扑排序时，由原先对 map 的结构遍历和更新改成对数组的遍历和更新。此外，再用一个变量来表示已经排序的元素，当所有入度为 0 的节点遍历结束后，如果已经排序的元素数量和课程的总数相同，则说明可以正常顺利完成所有课程的学习，否则，存在循环依赖，不能完成所有课程的学习。

```java
class Solution 
{
    public boolean canFinish(int numCourses, int[][] prerequisites) 
    {
        int[] indegree = new int[numCourses];
        
        // 计算课程的入度
        for (int i = 0; i < prerequisites.length; i++)
        {
            indegree[prerequisites[i][1]]++;
        }
        
        int num = 0;
        // 将入度为零的节点放入课程列表中
        List<Integer> courseList = new LinkedList<>();
        for (int i = 0; i < numCourses; i++)
        {
            if (indegree[i] == 0)
            {
                courseList.add((Integer)i);
                num++;
            }
        }
        
        // 依次取出课程列表中课程，并取出其他课程对此课程的依赖
        while (!courseList.isEmpty())
        {
            int course = courseList.remove(0);
            for (int i = 0; i < prerequisites.length; i++)
            {
                // 其他课程依赖此课程，则删除此依赖关系
                if (prerequisites[i][0] == course)
                {
                    // 依赖当前课程的其他课程的入度减一
                    indegree[prerequisites[i][1]]--;
                    // 入度为零，表明不需要依赖其他课程了，将此课程取出放入课程列表中
                    if (indegree[prerequisites[i][1]] == 0)
                    {
                        courseList.add(prerequisites[i][1]);
                        num++;
                    }
                }
            }
        }
        
        return num == numCourses;
    }
}
```

