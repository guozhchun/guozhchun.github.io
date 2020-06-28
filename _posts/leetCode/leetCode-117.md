---
title: leetCode-117:Populating Next Right Pointers in Each Node II
date: 2019-12-22 18:00:49
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个二叉树，要求填充每个节点的下一个节点（指向节点右边的兄弟节点，没有右边的节点则填充为空）。在不算递归栈的情况下，要求空间复杂度是常量。题目链接：**[点我](https://leetcode.com/problems/populating-next-right-pointers-in-each-node-ii/)**

<!-- more -->

# 样例输入输出

> 输入：[1,2,3,4,5,null,7]
>
> 这是一个三层的二叉树，从上到下，从左到右的节点是：1，2，3，4，5，null，7。其中 1 的左孩子是 2，右孩子是 3，依次类推。其中 null 表示没有该节点，即 3 没有左孩子
>
> 输出：[1,#,2,3,#,4,5,7,#]
>
> 这是补充上节点右边节点后，按照从上到下，从左到右的节点的遍历的输出结果，其中 # 代表空

> 输入：[1]
>
> 输出：[1]

# 问题解法

此题跟 [leetCode-116](https://leetcode.com/problems/populating-next-right-pointers-in-each-node) 类似，唯一的不同是，116 是一颗完全二叉树，而本次仅仅是一个二叉树，中间部分节点可能没有孩子。鉴于此，仍然可以用递归来解答，只不过需要改变一下递归的顺序，不能是 本节点 -> 左孩子 -> 右孩子 的顺序了，因为这样的顺序并不能保证在下一层中节点存在空缺时 next 指针的正确连接（因为本层的 next 指针没有完全连接好，所以不能顺着本层找到下层的节点）。需要改成 本节点 -> next 节点 -> 本节点的左孩子（没有左孩子就用右孩子），这样就能保证对当前节点的孩子进行 next 指针匹配时，当前节点右边的节点的 next 指针都已经匹配完成，从而能够为孩子节点的 next 指针匹配找到正确的节点。另外，需要注意的是，按照递归顺序，在 next 递归中，会存在对节点重复遍历的情况，此时进行剪枝判断，如果 next 的节点的孩子节点的 next 指针已经做了匹配，则不用再次进行遍历。代码如下

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
        // root.left != null && root.left.next != null 是剪枝判断，避免重复遍历
        // root.right != null && root.right.next != null 是剪枝判断，避免重复遍历
        if (root == null || (root.left != null && root.left.next != null) || (root.right != null && root.right.next != null))
        {
            return root;
        }
        
        Node from;    // 当前节点的某个孩子节点，需要指向 父节点的兄弟节点的孩子
        Node first;   // 当前节点的某个孩子节点，下一层遍历开始的节点
        if (root.left != null)
        {
            first = root.left;
            if (root.right != null)
            {
                root.left.next = root.right;
                from = root.right;
            }
            else
            {
                from = root.left;
            }
        }
        else
        {
            first = root.right;
            from = root.right;
        }
        
        if (from != null)
        {
            Node to = null;
            Node temp = root.next;
            while (temp != null)
            {
                if (temp.left != null)
                {
                    to = temp.left;
                    break;
                }
                
                if (temp.right != null)
                {
                    to = temp.right;
                    break;
                }
                
                temp = temp.next;
            }
            
            from.next = to;
        }

        connect(root.next);
        connect(first);
        
        return root;
    }
}
```

