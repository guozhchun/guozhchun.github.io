---
title: leetCode-99:Recover Binary Search Tree
date: 2020-06-21 18:34:00
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个二叉搜索树，其中有两个数字位置错了，要求在不改变树结构的前提下，将这两个数字交换，恢复成正常的二叉搜索树。题目链接：**[点我](https://leetcode.com/problems/recover-binary-search-tree)**

<!-- more -->

# 样例输入输出

> 输入：[1,3,null,null,2]
>
> 说明：树结构如下
>
>   1
>
> /
>
> 3
>
>   \
>
> ​    2
>
> 输出：[3,1,null,null,2]
>
> 说明：树结构如下
>
> ​    3
>
>   /
>
> 1
>
>   \
>
> ​    2

# 问题解法

对于二叉搜索树，使用中序遍历时，会得到一个升序的序列，现在给出的树中，有两个数字位置错误，因此，只需要使用中序遍历二叉树，每次遍历时判断当前节点是否比上一个节点小，如果小，则说明上一个节点或当前节点的数字是错的。待遍历结束后，将这两个节点的数字进行交换，就能在不改变树结构的情况下恢复正常的二叉搜索树。代码如下：

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
    TreeNode first = null;
    
    TreeNode second = null;
    
    TreeNode prev = null;
    
    public void recoverTree(TreeNode root)
    {
        findTargetNode(root);
        
        int temp = first.val;
        first.val = second.val;
        second.val = temp;
    }
    
    private void findTargetNode(TreeNode root)
    {
        if (root == null)
        {
            return;
        }
        
        findTargetNode(root.left);

        if (prev != null && prev.val >= root.val)
        {
            if (first == null)
            {
                first = prev;
            }
            second = root;
        }
        prev = root;
        
        findTargetNode(root.right);
    }
}
```

