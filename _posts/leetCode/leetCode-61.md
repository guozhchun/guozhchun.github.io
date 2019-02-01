---
title: leetCode-61:Rotate List
date: 2019-01-18 21:35:17
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个链表和一个数字 k，要求将链表旋转 k 次（从最右端向最左端进行旋转）。题目链接：**[点我](https://leetcode.com/problems/rotate-list/)**

<!-- more -->

# 样例输入输出

> 输入：1->2->3->4->5->NULL, k = 2
>
> 输出：4->5->1->2->3->NULL
>
> 解释：
>
> 第一次旋转后的结果：5->1->2->3->4->NULL
>
> 第二次旋转后的结果：4->5->1->2->3->NULL

> 输入：0->1->2->NULL, k = 4
>
> 输出：2->0->1->NULL
>
> 解释：
>
> 第一次旋转后的结果：2->0->1->NULL
>
> 第二次旋转后的结果：1->2->0->NULL
>
> 第三次旋转后的结果：0->1->2->NULL
>
> 第四次旋转后的结果：2->0->1->NULL

# 问题解法

由于输入的整数 k 可能大于链表的长度，因此需要先求出链表的长度，对 k 进行取余，得出链表截断的地方，即旋转后的头指针的位置，然后根据此位置变换链表头指针和尾指针即可得到答案。代码如下：

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
    public ListNode rotateRight(ListNode head, int k) 
    {
        if (head == null || head.next == null)
        {
            return head;
        }
        
        ListNode p = head;
        int length = 1;
        while (p.next != null)
        {
            p = p.next;
            length++;
        }
        
        int step = length - k % length;
        if (step == length)
        {
            return head;
        }

        ListNode pSecond = head;
        for (int i = 1; i < step; i++)
        {
            pSecond = pSecond.next;
        }
        
        p.next = head;
        head = pSecond.next;
        pSecond.next = null;
        
        return head;
    }
}
```

