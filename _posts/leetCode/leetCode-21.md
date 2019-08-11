---
title: leetCode-21:Merge Two Sorted Lists
date: 2019-06-21 20:59:50
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定两个有序链表，要求将其合并成一个有序链表返回。题目链接：**[点我](https://leetcode.com/problems/merge-two-sorted-lists/)**

<!-- more -->

# 样例输入输出

> 输入：1->2->4, 1->3->4
>
> 输出：1->1->2->3->4->4

> 输入：1->2->3->4,   1
>
> 输出：1->1->2->3->4

# 问题解法

直接用归并法即可。代码如下

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution 
{
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) 
    {
        if (l1 == null)
        {
            return l2;
        }
        
        if (l2 == null)
        {
            return l1;
        }
        
        ListNode p1 = l1;
        ListNode p2 = l2;
        if (p1.val > p2.val)
        {
            p1 = l2;
            p2 = l1;
        }
        
        ListNode head = p1;
        while (p2 != null)
        {
            while (p1.next != null && p1.next.val < p2.val)
            {
                p1 = p1.next;
            }
            
            ListNode temp = p1.next;
            p1.next = p2;
            p1 = p2;
            p2 = temp;
        }
        
        return head;
    }
}
```

