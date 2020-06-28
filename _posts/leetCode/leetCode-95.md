---
title: leetCode-95:Unique Binary Search Trees II
date: 2020-05-17 10:50:02
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个整数 `n`，要求找出以数字 `1 ~ n` 构成的不同结构的二叉搜索树。题目链接：**[点我]( https://leetcode.com/problems/unique-binary-search-trees-ii/)**

<!-- more -->

# 样例输入输出

> 输入：3
>
> 输出： [[1,null,2,null,3],[1,null,3,2],[2,1,3],[3,1,null,null,2],[3,2,null,1]] 
>
> 说明：输出代表以下 5 颗二叉搜索树
>
> ![二叉搜索树](https://guozhchun.github.io/images/leetcode95.png)

> 输入：0
>
> 输出：[]

# 问题解法

对从 `1 ~ n` 中的每个数字，分别将其作为根节点进行划分，可以划分出左右两个子树，用递归方式对子树进行类似的构造获得左右子树的二叉搜索树列表，然后将其分别拼装到根节点上即可。代码如下：

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution
{
    public List<TreeNode> generateTrees(int n)
    {
        if (n < 1)
        {
            return new ArrayList<>();
        }
        
        return generateTrees(1, n);
    }
    
    private List<TreeNode> generateTrees(int start, int end)
    {
        List<TreeNode> treeNodes = new ArrayList<>();
        if (start > end)
        {
            treeNodes.add(null);
            return treeNodes;
        }
        
        if (start == end)
        {
            treeNodes.add(new TreeNode(start));
            return treeNodes; 
        }
        
        for (int i = start; i <= end; i++)
        {
            List<TreeNode> leftTrees = generateTrees(start, i - 1);
            List<TreeNode> rightTrees = generateTrees(i + 1, end);
            for (TreeNode leftTree : leftTrees)
            {
                for (TreeNode rightTree : rightTrees)
                {
                    TreeNode root = new TreeNode(i);
                    root.left = leftTree;
                    root.right = rightTree;
                    treeNodes.add(root);
                }
            }
        }
        
        return treeNodes;
    }
}
```



