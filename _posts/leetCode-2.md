---
title: leetCode-2:Add Two Numbers
date: 2018-05-29 20:53:08
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定两个链表，链表中的数字是非负整数，表示一个整数的逆序，要求将这两个链表进行相加。其实就是模拟大数相加的过程。题目链接：**[点我](https://leetcode.com/problems/add-two-numbers/description/)**

<!-- more -->

# 样例输入输出

> 输入： (2 -> 4 -> 3)   (5 -> 6 -> 4)
>
> 输出： (7 -> 0 -> 8)

> 输入： (1)  (9 -> 9 -> 2)
>
> 输出： (0 -> 0 -> 3)

# 问题解法

## 直接模拟相加

遍历链表，对遍历过的节点的值进行相加，再与前一个节点产生的进位进行相加，再将结果模10得到本新节点的值，将和除10得到新的进位与下个节点进行相加。代码实现如下

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
    private void handleRemainList(ListNode resultCurrent, ListNode p, int prevCarryNum)
    {
        while (prevCarryNum > 0)
        {
            int sum = prevCarryNum;
            if (p != null)
            {
                sum += p.val;
                p = p.next;
            }
            resultCurrent.next = new ListNode(sum % 10);
            resultCurrent = resultCurrent.next;
            prevCarryNum = sum / 10;
        }
        resultCurrent.next = p;
    }
    
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) 
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
        ListNode result = new ListNode((p1.val + p2.val) % 10);
        ListNode resultCurrent = result;
        int prevCarryNum = (p1.val + p2.val) / 10;
        p1 = p1.next;
        p2 = p2.next;
        while (p1 != null && p2 != null)
        {
            int sum = p1.val + p2.val + prevCarryNum;
            resultCurrent.next = new ListNode(sum % 10);
            prevCarryNum = sum / 10;
            resultCurrent = resultCurrent.next;
            p1 = p1.next;
            p2 = p2.next;
        }
        
        if (p1 == null)
        {
            handleRemainList(resultCurrent, p2, prevCarryNum);
        }
        else
        {
            handleRemainList(resultCurrent, p1, prevCarryNum);
        }
        
        return result;
    }
}
```

以上代码主要是同时遍历链表，遇到一个链表为空时，处理剩下的链表，代码有点啰嗦，leetcode上有更简便的写法，请参考**[这里](https://leetcode.com/articles/add-two-numbers/)**

## 使用BigInteger

本题是模拟两个大数相加的过程，在java库中已经有`BigInteger`类实现了大数相加、相减、相乘、相除等功能，因此可以直接使用这个类来完成本题。代码如下

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
import java.math.BigInteger;
class Solution 
{
    private String getStringNumber(ListNode p)
    {
        StringBuilder sb = new StringBuilder();
        while (p != null)
        {
            sb.append(p.val);
            p = p.next;
        }
        
        return sb.reverse().toString();
    }
    
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) 
    {
        if (l1 == null)
        {
            return l2;
        }
        
        if (l2 == null)
        {
            return l1;
        }
        
        String firstNum = getStringNumber(l1);
        String secondNum = getStringNumber(l2);
        
        BigInteger firstBigNum = new BigInteger(firstNum);
        BigInteger secondBigNum = new BigInteger(secondNum);
        
        String sumString = firstBigNum.add(secondBigNum).toString();
        StringBuilder sb = new StringBuilder(sumString).reverse();
        ListNode result = new ListNode(Integer.valueOf(Character.toString(sb.charAt(0))));
        ListNode current = result;
        for (int i = 1; i < sb.length(); i++)
        {
            current.next = new ListNode(Integer.valueOf(Character.toString(sb.charAt(i))));
            current = current.next;
        }
        return result;
    }
}
```

由于传入的参数是链表，且是逆序的数字，因此需要先将其转出正序整数的字符串，然后使用`BigInteger.add`进行相加。最后再将结果转出逆序的整数字符串，再构造链表返回。由于此过程涉及转换比较多，因此效率低下，不如第一种效率好。但是当传入参数为字符串并且要求返回的参数为字符串时，此种方法不比第一种差，且可读性较好。