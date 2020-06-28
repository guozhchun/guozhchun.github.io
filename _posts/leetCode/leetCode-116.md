---
title: leetCode-116:Populating Next Right Pointers in Each Node
date: 2019-12-15 20:12:51
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个完全二叉树，要求填充每个节点的下一个节点（指向节点右边的兄弟节点，没有右边的节点则填充为空）。题目链接：**[点我](https://leetcode.com/problems/populating-next-right-pointers-in-each-node)**

<!-- more -->

# 样例输入输出

> 输入：[1,2,3,4,5,6,7]           
>
> 代表这是一个三层的二叉树，从上到下，从左到右的节点是：1，2，3，4，5，6，7。其中 1 的左孩子是 2，右孩子是 3，依次类推
>
> 输出：[1,#,2,3,#,4,5,6,7,#]
>
> 这是补充上节点右边节点后，按照从上到下，从左到右的节点的遍历的输出结果，其中 # 代表空

> 输入：[1]
>
> 输出：[1]

# 问题解法

按照先序遍历树的方式遍历每个节点，在每个节点上，将左孩子的 `next` 指针指向右孩子，将右孩子的 `next` 指针指向当前节点的 `next` 节点的左孩子（如果存在的话，不存在就是 `null`）。代码如下

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;
    public Node next;

    public Node() {}
    
    public Node(int _val) {
        val = _val;
    }

    public Node(int _val, Node _left, Node _right, Node _next) {
        val = _val;
        left = _left;
        right = _right;
        next = _next;
    }
};
*/
class Solution 
{
    public Node connect(Node root)
    {
        if (root == null)
        {
            return root;
        }
        
        if (root.left != null)
        {
            root.left.next = root.right;
        }
        
        if (root.next != null && root.right != null)
        {
            root.right.next = root.next.left;
        }
        
        connect(root.left);
        connect(root.right);
        
        return root;
    }
}
```

