---
title: leetCode-86:Partition List
date: 2019-03-17 22:31:44
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个链表和一个整数，要求将链表中节点值小于该整数的节点移动到链表中第一个节点值大于或等于该整数的节点前面去。在移动的过程中，要求移动的节点保持原有的相对顺序。题目链接：**[点我](https://leetcode.com/problems/partition-list/)**

<!-- more -->

# 样例输入输出

> 输入：head = 1->4->3->2->5->2,     x = 3
>
> 输出：1->2->2->4->3->5
>
> 解释：第一个大于等于 3 的节点是 4，因此将两个 2 的节点移动到 4 之前

> 输入：head = 5->4->3->2->1,   x = 3
>
> 输出：2->1->5->4->3
>
> 解释：第一个大于等于 3 的节点是 5，因此将 2 和 1 两个节点移动到 5 前面，并且保持原先的顺序

# 问题解法

先用一个循环，找到第一个大于或等于输入的整数的节点，此时该节点前即为待插入节点的区域。继续遍历链表，将后续每个小于该整数的节点从链表中移除，插入到前面找到的待插入节点区域中。为方便操作，不单独考虑首节点大于或等于输入整数的情况，可以新建一个头节点放在链表的前面，并将值设置为比原先链表头节点值小，这样可以直接进行循环判断，不用对头节点进行额外的考虑。代码如下

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
    public ListNode partition(ListNode head, int x) 
    {
        if (head == null || x == Integer.MIN_VALUE)
        {
            return head;
        }
        
        ListNode mockHead = new ListNode(x - 1);
        mockHead.next = head;
        ListNode insertNode = mockHead;
        ListNode current = mockHead;
        
        while (current.next != null)
        {
            if (current.next.val >= x)
            {
                insertNode = current;
                break;
            }
            current = current.next;
        }
        
        while (current.next != null)
        {
            if (current.next.val < x)
            {
                ListNode next = current.next;
                current.next = next.next;
                next.next = insertNode.next;
                insertNode.next = next;
                insertNode = insertNode.next;
                continue;
            }
            
            current = current.next;
        }
        
        return mockHead.next;
    }
}
```

