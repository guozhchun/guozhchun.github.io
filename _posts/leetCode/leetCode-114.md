---
title: leetCode-114:Flatten Binary Tree to Linked List
date: 2019-12-29 11:53:59
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个二叉树，要求将二叉树的节点按照先序遍历的方式将其转成链表，链表节点均在树的右节点。题目链接：**[点我](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/)**

<!-- more -->

# 样例输入输出

> 输入：[1,2,5,3,4,null,6]
>
> 这是一个三层的二叉树，从上到下，从左到右的节点是：1，2，5，3，4，null，6。其中 1 的左孩子是 2，右孩子是 5，依次类推。其中 null 表示没有该节点，即 5 没有左孩子
>
> 输出：[1,null,2,null,3,null,4,null,5,null,6] 
>
> 这是一个用树的方式来表示的链表，数节点上左孩子都是 null

> 输入：[1]
>
> 输出：[1]

# 问题解法

递归遍历树，先处理左节点，再处理右节点，在将左右孩子树转成链表时，分别记录链表的尾节点，用于在父节点将右孩子连接到左孩子上，否则还需要再次遍历左孩子的链表，造成时间的浪费。需要特别注意的是，在处理过程中，需要将树节点的左孩子置为 null，否则与输出不匹配。

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
    public void flatten(TreeNode root) 
    {
        flattenTree(root);
    }
    
    private TreeNode flattenTree(TreeNode root)
    {
        if (root == null)
        {
            return root;
        }
        
        TreeNode leftEnd = flattenTree(root.left);
        TreeNode rightEnd = flattenTree(root.right);
        
        if (leftEnd != null)
        {
            leftEnd.right = root.right;
            root.right = root.left;
            root.left = null;   // 必须置空，否则输出不匹配
        }
        
        if (rightEnd != null)
        {
            return rightEnd;
        }
        else if (leftEnd != null)
        {
            return leftEnd;
        }
        else
        {
            return root;
        }
    }
}
```

