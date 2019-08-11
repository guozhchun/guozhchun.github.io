---
title: leetCode-25:Reverse Nodes in k-Group
date: 2019-07-07 14:49:44
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个链表和一个正整数 k（k 的值小于链表的长度），要求将链表进行分组，每组 k 个节点，在每组中将链表进行倒序，如果最后一组没有 k 个节点，则不用进行逆序，最后将这些分组按顺序连接成新的链表并返回头节点。要求空间复杂度是 `O(1)`，并且不能改变链表节点中的值。题目链接：**[点我](https://leetcode.com/problems/reverse-nodes-in-k-group/)**

<!-- more -->

# 样例输入输出

> 输入：1->2->3->4->5->6->7->8    k = 3
>
> 输出：3->2->1->6->5->4->7->8

> 输入：1->2->3->4->5->6->7->8    k = 1
>
> 输出：1->2->3->4->5->6->7->8

# 问题解法

此题是链表的基本操作。只需要将链表进行分组，在每组中将链表进行倒序即可。需要注意的是保持链表的连续性，即多个倒序后的分组需要能够连接起来成一个新的链表。代码如下

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
    public ListNode reverseKGroup(ListNode head, int k) 
    {
        ListNode p = head;
        int length = 0;
        while (p != null)
        {
            length++;
            p = p.next;
        }
        
        // 为了便于操作，定义一个临时的头节点放在链表前面
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode pHead = dummy.next;
        ListNode prev = dummy;
        int index = 0;
        while (index + k <= length)
        {
            prev.next = reverseListNode(pHead, k);
            // k 个节点逆序后，pHead 指向的是该分组的最后一个节点
            // 为了使链表连续，此处需要更新 prev 指针位置
            prev = pHead;
            pHead = pHead.next;
            index += k;
        }
        
        return dummy.next;
    }
    
    /**
     * 将链表进行倒序
     * @param head   链表头节点
     * @param length 链表长度
     * @return 倒序后链表新的头节点
     */
    private ListNode reverseListNode(ListNode head, int length)
    {
        ListNode p = head;
        int count = 1;
        while (count < length)
        {
            ListNode pNext = p.next;
            p.next = pNext.next;
            pNext.next = head;
            head = pNext;
            count++;
        }
        
        return head;
    }
}
```

