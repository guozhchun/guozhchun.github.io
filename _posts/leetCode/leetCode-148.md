---
title: leetCode-148:Sort List
date: 2020-06-07 13:55:39
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个链表，要求用 `O(nlog n)` 的时间复杂度和 `O(1)` 的空间复杂度，对链表进行排序。题目链接：**[点我]( https://leetcode.com/problems/sort-list)**

<!-- more -->

# 样例输入输出

> 输入：4->2->1->3
>
> 输出：1->2->3->4

> 输入：-1->5->3->4->0
>
> 输出：-1->0->3->4->5

# 问题解法

使用归并排序算法，由于要求空间复杂度为常量，所以此处不用递归，而用递推。具体做法是：将链表按长度为 `n`（n = 1, 2, 4, 8, 16, ...）分割成不同的子链表，再将相邻的两个子链表进行归并排序。重复执行以上过程，直到 `n = length / 2`。代码如下

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution
{
    public ListNode sortList(ListNode head)
    {
        int length = countListLength(head);
        for (int i = 1; i <= length; i *= 2)
        {
            ListNode nextHead = head;
            ListNode prevEnd = null;
            while (nextHead != null)
            {
                ListNode first = nextHead;
                ListNode second = cutList(first, i);
                nextHead = cutList(second, i);
                ListNode tempHead = mergeList(first, second);
                if (prevEnd == null)
                {
                    head = tempHead;
                }
                else
                {
                    prevEnd.next = tempHead;
                }

                while (tempHead.next != null)
                {
                    tempHead = tempHead.next;
                }
                tempHead.next = nextHead;
                prevEnd = tempHead;
            }
        }

        return head;
    }

    private ListNode mergeList(ListNode first, ListNode second)
    {
        if (first == null)
        {
            return second;
        }

        if (second == null)
        {
            return first;
        }

        ListNode head;
        ListNode p1 = first;
        ListNode p2 = second;
        if (first.val < second.val)
        {
            head = first;
            p1 = first.next;
        }
        else
        {
            head = second;
            p2 = second.next;
        }

        ListNode current = head;
        while (p1 != null && p2 != null)
        {
            if (p1.val < p2.val)
            {
                current.next = p1;
                p1 = p1.next;
            }
            else
            {
                current.next = p2;
                p2 = p2.next;
            }
            current = current.next;
        }

        if (p1 == null)
        {
            current.next = p2;
        }
        else
        {
            current.next = p1;
        }

        return head;
    }

    /**
     * 截取链表前 n 个节点组成子链表，返回截取后剩下链表的首节点
     *
     * @param head 链表首节点
     * @param n    截取的长度
     * @return 截取后剩下链表的首节点
     */
    private ListNode cutList(ListNode head, int n)
    {
        ListNode current = head;
        int count = 1;
        while (count < n && current != null)
        {
            current = current.next;
            count++;
        }

        ListNode next = null;
        if (current != null)
        {
            next = current.next;
            current.next = null;
        }

        return next;
    }

    private int countListLength(ListNode head)
    {
        int length = 0;
        ListNode current = head;
        while (current != null)
        {
            length++;
            current = current.next;
        }

        return length;
    }
}
```

