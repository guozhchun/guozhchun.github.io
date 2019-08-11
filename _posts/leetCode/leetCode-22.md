---
title: leetCode-22:Generate Parentheses
date: 2019-05-11 12:10:28
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个整数，表示括号对的数量，要求找出所有合法的括号组合。题目链接：**[点我](https://leetcode.com/problems/generate-parentheses/)**

<!-- more -->

# 样例输入输出

> 输入：3
>
> 输出：{   "()()()", "()(())", "(()())", "(())()", "((()))"    }

> 输入：2
>
> 输出：{ "()()", "(())" }

# 问题解法

## 递归+剪枝

直接从第一对括号开始向后搜索直到最后一对括号匹配结束，在每次的搜索过程中，对拿到的字符串，从左到右依次插入一对括号，产生新的字符串，然后进入下一个递归。在产生新的字符串时，判断该字符串是否已经出现过，如果出现过则不进入下一轮递归，避免重复的搜索。代码如下

```java
class Solution 
{
    private void generateParenthesis(List<String> parentheses, String str, int n)
    {
        if (n == 0)
        {
            if (!parentheses.contains(str))
            {
                parentheses.add(str);
            }
            
            return;
        }
        
        int length = str.length();
        Set<String> newStrSet = new HashSet<>();
        for (int i = 0; i < length; i++)
        {
            String prefix = str.substring(0, i);
            String suffix = str.substring(i);
            String newStr = prefix + "()" + suffix;
            // 剪枝，避免重复的搜索
            if (newStrSet.contains(newStr))
            {
                continue;
            }
            newStrSet.add(newStr);
            generateParenthesis(parentheses, newStr, n - 1);
        }
    }
    
    public List<String> generateParenthesis(int n) 
    {
        List<String> result = new ArrayList<>();
        generateParenthesis(result, "()", n - 1);
        return result;
    }
}
```

虽然这种递归使用了剪枝进行优化，但是仍然超时了。

## 尾部递归

由于上面的递归方式超时，所以需要换另一种方法。由于这种思路从理论上来讲是可行的，所以将尝试将递归改成尾部递归进行求解，同时，在求结果集时，不使用 List 及判重，直接使用 Set 增加新的元素，最后再将 Set 转成 List 返回。实验证明，使用尾部递归的效率远远高于前一种递归加剪枝方法效率。使用这种方法也能保证在时间限制范围内完成所有的求解。代码如下

```java
class Solution 
{
    public List<String> generateParenthesis(int n)
    {
        if (n == 1)
        {
            return new ArrayList<String>(){{add("()");}};
        }
        
        List<String> preRestult = generateParenthesis(n - 1);
        Set<String> parentheses = new HashSet<>();
        for (String parenthes : preRestult)
        {
            int length = parenthes.length();
            for (int i = 0; i < length; i++)
            {
                String prefix = parenthes.substring(0, i);
                String suffix = parenthes.substring(i);
                String newParenthes = prefix + "()" + suffix;
                parentheses.add(newParenthes);
            }
        }
        return new ArrayList<>(parentheses);
    }
}
```

