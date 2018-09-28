---
title: leetCode-113:Path Sum II
date: 2018-09-16 12:23:01
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一颗树和一个数字 sum ，要求寻找所有从树的根节点到叶子节点的路径，使得这个路径上的节点数字之和为 sum。题目链接：**[点我](https://leetcode.com/problems/path-sum-ii/description/)**

<!-- more -->

# 样例输入输出

> 输入：[5,4,8,11,null,13,4,7,2,null,null,5,1]
>            22
>
> 输出：[[5, 4, 11, 2],   [5, 8, 4, 5]]
>
> 解析：输入的树如下，存在两条路径：5 -> 4 -> 11 -> 2，5 -> 8 -> 4 -> 5，使得路径上数字之和为 22
>
> ```
      5
     / \
    4   8
   /   / \
  11  13  4
 /  \    / \
7    2  5   1
 ```


# 问题解法

此题比较简单，直接使用回溯算法进行求解即可。 代码如下

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution 
{
    private void findPathSum(TreeNode root, int sum, List<Integer> path, List<List<Integer>> result)
    {
        if (root == null)
        {
            return;
        }
        
        
        if (root.left == null && root.right == null && root.val == sum)
        {
            List<Integer> pathToSave = new ArrayList<>(path);
            pathToSave.add(root.val);
            result.add(pathToSave);
            return;
        }
        
        path.add(root.val);
        
        findPathSum(root.left, sum - root.val, path, result);   // 寻找左子树的路径
        findPathSum(root.right, sum - root.val, path, result);  // 寻找右子树的路径
        
        path.remove(path.size() - 1);  // 移除刚增加的元素，回溯
    }
    
    public List<List<Integer>> pathSum(TreeNode root, int sum) 
    {
        List<List<Integer>> result = new ArrayList<>();
        findPathSum(root, sum, new LinkedList<>(), result);
        return result;
    }
}
```

