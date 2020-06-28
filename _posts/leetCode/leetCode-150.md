---
title: leetCode-150:Evaluate Reverse Polish Notation
date: 2019-11-24 15:36:22
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个字符串数组，表示算术的后缀表达式，并且保证这个表达式是合法的，要求算出这个表达式的值。题目链接：**[点我](https://leetcode.com/problems/evaluate-reverse-polish-notation/)**

<!-- more -->

# 样例输入输出

> 输入：["2", "1", "+", "3", "*"]
>
> 输出：9
>
> 解释：((2 + 1) * 3) = 9

> 输入：["10", "6", "9", "3", "+", "-11", "*", "/", "*", "17", "+", "5", "+"]
>
> 输出：22
>
> 解释：((10 \* (6 / ((9 + 3) \* -11))) + 17) + 5 = ((10 \* (6 / (12 \* -11))) + 17) + 5 = ((10 \* (6 / -132)) + 17) + 5 = ((10 \* 0) + 17) + 5 = (0 + 17) + 5 = 17 + 5 = 22

# 问题解法

用栈来保存运算的数，遍历数组，如果遇到数字，则压入栈中，如果遇到运算符合，则弹出栈顶元素作为第二个操作数，再弹出栈顶元素作为第一个运算数，对其进行运算得到结果压入栈中。运算结束后，栈顶的元素就是表达式的结果。代码如下

```java
class Solution
{
    public int evalRPN(String[] tokens)
    {
        if (tokens == null || tokens.length == 0)
        {
            return 0;
        }
        
        Stack<Integer> stack = new Stack<>();
        for (int i = 0; i < tokens.length; i++)
        {
            if (tokens[i].equals("+"))
            {
                int second = stack.pop();
                int first = stack.pop();
                int result = first + second;
                stack.push(result);
            }
            else if (tokens[i].equals("-"))
            {
                int second = stack.pop();
                int first = stack.pop();
                int result = first - second;
                stack.push(result);
            }
            else if (tokens[i].equals("*"))
            {
                int second = stack.pop();
                int first = stack.pop();
                int result = first * second;
                stack.push(result);
            }
            else if (tokens[i].equals("/"))
            {
                int second = stack.pop();
                int first = stack.pop();
                int result = first / second;
                stack.push(result);
            }
            else
            {
                stack.push(Integer.parseInt(tokens[i]));
            }
        }
        return stack.pop();
    }
}
```

