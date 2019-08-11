---
title: leetCode-23:Merge k Sorted Lists
date: 2019-06-21 22:07:08
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定 k 个有序的链表，要求将其合并成一个有序的链表返回。题目链接：**[点我](https://leetcode.com/problems/merge-k-sorted-lists/)**

<!-- more -->

# 样例输入输出

> 输入：[1->4->5, 1->3->4, 2->6]
>
> 输出：1->1->2->3->4->4->5->6

> 输入：[]
>
> 输出：[]

# 问题解法

利用归并排序的思路，将列表中两两分组，对每组中的两个链表，运用[合并两个有序链表](https://guozhchun.github.io/2019/06/21/leetCode/leetCode-21/)的做法将其合并成一个有序的链表，将这些合并后的链表继续两两分组，重复上述过程，直到最后只剩下一个链表，就是合并后的有序的链表。代码如下

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
    private ListNode mergeTwoLists(ListNode l1, ListNode l2) 
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
    
    public ListNode mergeKLists(ListNode[] lists) 
    {
        if (lists == null || lists.length == 0)
        {
            return null;
        }
        
        while (lists.length > 1)
        {
            ListNode[] temp = new ListNode[(lists.length + 1) / 2];
            int index = 0;
            for (int i = 0; i < lists.length; )
            {
                if (i + 1 < lists.length)
                {
                    temp[index] = mergeTwoLists(lists[i], lists[i + 1]);
                    i += 2;
                }
                else
                {
                    temp[index] = lists[i];
                    i++;
                }
                
                index++;
            }
            
            lists = temp;
        }
        
        return lists[0];
    }
}
```

