---
title: leetCode-24:Swap Nodes in Pairs
date: 2018-11-10 19:26:39
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一个链表，要求按照顺序将链表中每两个节点分成一组，并将组中两个节点交换顺序，重新组成新的链表后返回链表的头节点指针。要求只能使用额外的常量空间。题目链接：**[点我](https://leetcode.com/problems/swap-nodes-in-pairs/)**

<!-- more -->

# 样例输入输出

> 输入：[1,2,3,4]
>
> 输出：[2,1,4,3]

> 输入：[1,2,3]
>
> 输出：[2,1,3]

# 问题解法

此题就是链表的基本使用。只需要定义两个指针，依次对相邻的两个节点进行顺序交换即可。为了能够使链表的第一个和第二个节点能够像其他节点一样进行顺序交换，可以定义一个新的临时头节点放在原先链表的头节点的前面，这样就能在一个循环中完成两个相邻节点顺序的交换。因为是只新建了一个节点，并不是对链表的每个节点都新建节点，所以其占用的空间是常量的，满足题目的要求。代码如下

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
    public ListNode swapPairs(ListNode head) 
    {
        // 为了方便使链表第一、二个节点能够跟链表中其他的节点一样进行顺序交换
        // 此处定义一个临时头节点放在原有链表头节点之前
        // 待交换节点完毕后，删除此临时头节点
        // 题目要求只能使用额外的常量空间，此处只是新建一个节点，并不是新建链表中每个节点，是满足常量空间要求的
        ListNode tempHeadNode = new ListNode(0);
        tempHeadNode.next = head;
        
        ListNode p2 = tempHeadNode;
        ListNode p1 = tempHeadNode.next;
        
        // p1 == null：说明链表是只有一个节点，或者，链表是双数节点，而且 p2 已经指向链表最后一个节点，已经对完成交换动作
        // p1.next == null：说明链表节点数大于 1，且是单数节点，此时 p1 已经指向链表最后一个节点，交换动作执行完毕
        while (p1 != null && p1.next != null)
        {
            // 交换相邻的两个节点
            p2.next = p1.next;
            p1.next = p1.next.next;
            p2.next.next = p1;
            
            // 移动指针，进入下一轮循环
            p2 = p1;
            p1 = p1.next;
        }
        
        // 删除临时头部节点，返回正常的头节点指针
        head = tempHeadNode.next;
        return head;
    }
}
```

