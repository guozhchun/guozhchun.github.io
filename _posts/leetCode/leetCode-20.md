---
title: leetCode-20:Valid Parentheses
date: 2019-04-30 23:53:30
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个字符串，只包含`{`、`}`、`[`、`]`、`(`、`)`。要求判断这个字符串是否是符合括号的原则的。题目链接：**[点我](https://leetcode.com/problems/valid-parentheses/)**

<!-- more -->

# 样例输入输出

> 输入：{[]}
>
> 输出：true

> 输入：{[}]
>
> 输出：false

# 问题解法

使用栈存储左括号，遍历字符串，每次遇到左括号时压入栈，遇到右括号时，查看栈顶的左括号是否与这个右括号对应，如果对应，则将栈顶元素出栈，否则说明字符串不合法。当字符串遍历结束时，如果栈中元素为空，说明字符串合法，否则字符串不合法。代码如下

```java
class Solution 
{
    public boolean isValid(String s) 
    {
        if (s == null)
        {
            return true;
        }
        
        Stack<Character> stack = new Stack<>();
        int length = s.length();
        for (int i = 0; i < length; i++)
        {
            char ch = s.charAt(i);
            if (ch == '[' || ch == '(' || ch == '{')
            {
                stack.push(Character.valueOf(ch));
                continue;
            }
            
            if (stack.isEmpty())
            {
                return false;
            }
            
            char topCh = stack.pop();
            if (ch == ']' && topCh == '[')
            {
                continue;
            }
            
            if (ch == ')' && topCh == '(')
            {
                continue;
            }
            
            if (ch == '}' && topCh == '{')
            {
                continue;
            }
            
            return false;
        }
        
        return stack.isEmpty();
    }
}
```

