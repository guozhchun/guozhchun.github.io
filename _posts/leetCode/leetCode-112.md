---
title: leetCode-112:Path Sum
date: 2018-09-16 11:37:05
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一颗树和一个数字 sum ，要求寻找是否存在从树的根节点到叶子节点的路径，使得这个路径上的节点数字之和为 sum。如果存在则返回 true，否则返回 false。题目链接：**[点我](https://leetcode.com/problems/path-sum/description/)**

<!-- more -->

# 样例输入输出

> 输入：[5,4,8,11,null,13,4,7,2,null,null,null,1]
>            22
>
> 输出：true
>
> 解析：输入的树如下，存在路径：5->4->11->2，和为 22，所以返回 true
>
> ```
       5
      / \
     4   8
    /   / \
   11  13  4
  /  \      \
 7    2      1
```

# 问题解法

问题题意描述得很清楚，解法也简单清晰，直接使用深搜遍历即可。代码如下

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
    public boolean hasPathSum(TreeNode root, int sum) 
    {
        if (root == null)
        {
            return false;
        }
        
        if (root.left == null && root.right == null && root.val == sum)
        {
            return true;
        }
        
        return hasPathSum(root.left, sum - root.val) || hasPathSum(root.right, sum - root.val);
    }
}
```

