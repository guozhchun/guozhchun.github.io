---
title: leetCode-19:Remove Nth Node From End of List
date: 2018-10-29 20:32:13
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个链表和一个正整数 n ，要求从链表中删除倒数第 n 个节点并返回链表的头节点。其中输入的 n 保证在合法范围内。要求只能遍历一次链表。题目链接：**[点我](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)**

<!-- more -->

# 样例输入输出

> 输入：[1,2,3,4,5]    5
>
> 输出：[2,3,4,5]

> 输入：[1,2,3,4,5]    3
>
> 输出：[1,2,4,5]

# 问题解法

使用两个指针，其中一个指针先走，走到距离头节点 n 个节点时，后续该指针继续往后遍历时，另一个指针从头节点开始，随着第一个指针每次移动一个节点向后遍历直到链表尾节点。当第一个指针移动到尾节点时，如果与另一个指针的距离刚好是 n 个几点，则删除另一个指针后面的节点，否则说明 n 刚好等于链表的长度，即要删除的节点是首节点。代码如下：

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    // 题目保证输入 n 合法
    public ListNode removeNthFromEnd(ListNode head, int n) 
    {
        if (head == null)
        {
            return null;
        }
        
        ListNode pLeft = head;
        ListNode pRight = head;
        int gap = 0;
        while (pRight.next != null)
        {
            pRight = pRight.next;
            gap++;
            // gap 等于 n 时，说明找到了两个指针的间距，此时只需要继续遍历直到链表最后一个元素
            // 此时 pLeft 指针所指的下一个节点就是要删除的“倒数第 n 个节点”
            if (gap > n)
            {
                pLeft = pLeft.next;
            }
        }
        
        // 如果两个指针的间隔小于 n，说明 n 是链表的长度，也就是要删除第一个节点
        // 否则直接删除 pLeft指针的下一个节点即可
        if (gap < n)
        {
            head = head.next;
        }
        else
        {            
            pLeft.next = pLeft.next.next;
        }
        
        return head;
    }
}
```

