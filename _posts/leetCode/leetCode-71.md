---
title: leetCode-71:Simplify Path
date: 2019-08-16 22:57:30
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个字符串表示 unix 系统下的绝对路径，要求将其转成最简单的等价路径。路径不能以 `/` 结尾（只有`/`的例外）。题目链接：**[点我](https://leetcode.com/problems/simplify-path/)**

<!-- more -->

# 样例输入输出

> 输入：`/home/`
>
> 输出：`/home`

> 输入：`/a/../../b/../c//.//`
>
> 输出：`/c`

# 问题解法

根据 `/` 将输入的路径进行分割，用一个新的数组存储最终路径的每个目录元素。遍历分割后的字符串数组，如果遇到 `..` 则将存储目录元素的数组中删掉一个元素，否则加入新的目录元素。最后将存在目录元素的字符串数组用 `/` 进行连接，即是最终的答案。代码如下：

```java
class Solution 
{
    public String simplifyPath(String path) 
    {
        String[] elements = path.split("/");
        String[] temp = new String[elements.length];
        int index = 0;
        for (int i = 1; i < elements.length; i++)
        {
            if (".".equals(elements[i]) || "".equals(elements[i]))
            {
                continue;
            }
            
            if ("..".equals(elements[i]))
            {
                index--;
                if (index < 0)
                {
                    index = 0;
                }
            }
            else
            {
                temp[index] = elements[i];
                index++;
            }
        }
        
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < index; i++)
        {
            sb.append("/").append(temp[i]);
        }
        
        if (sb.length() == 0)
        {
            sb.append("/");
        }
        
        return sb.toString();
    }
}
```

