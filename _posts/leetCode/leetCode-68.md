---
title: leetCode-68:Text Justification
date: 2020-05-24 12:50:06
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个由单词组成的数组和一个数字 `maxWidth`，要求将单词数组中的单词进行格式化输出，规则如下：

* 每一行只能包括单词和空格，单词之间用空格分隔，行的长度为 `maxWidth`
* 采用贪心策略，每一行要尽可能多的放置单词
* 空格的分布需要尽可能的均匀，如果空格不能均匀分布，则将空格放在单词左边而不是单词的右边
* 对于最后一行，单词间用一个空格分隔，剩余的长度用空格补充在最后一个单词后面。
* 对于一行只有一个单词，则剩余的长度用空格补充在单词后面

题目链接：**[点我](https://leetcode.com/problems/text-justification)**

<!-- more -->

# 样例输入输出

> 输入：words = ["This", "is", "an", "example", "of", "text", "justification."]    maxWidth = 16
>
> 输出：
>
> [
>    "This    is    an",
>    "example  of text",
>    "justification.  "
> ]

> 输入：words = ["What","must","be","acknowledgment","shall","be"]     maxWidth = 16
>
> 输出：
>
> [
>   "What   must   be",
>   "acknowledgment  ",
>   "shall be        "
> ]

# 问题解法

直接按照规则进行格式化即可，代码如下

```java
class Solution
{
    public List<String> fullJustify(String[] words, int maxWidth)
    {
        List<String> resultList = new LinkedList<>();
        int begin = 0;
        int end = 0;
        int currentWidth = 0;
        while (end < words.length)
        {
            if (currentWidth + words[end].length() <= maxWidth)
            {
                currentWidth = currentWidth + words[end].length() + 1;
                end++;
                if (end == words.length)
                {
                    resultList.add(getWordLine(words, maxWidth, begin, end));
                }
            }
            else
            {
                resultList.add(getWordLine(words, maxWidth, begin, end));
                begin = end;
                currentWidth = 0;
            }
        }

        return resultList;
    }

    private String getWordLine(String[] words, int maxWidth, int begin, int end)
    {
        StringBuilder line = new StringBuilder();

        // the last line, or the line has only one word
        if (end == words.length || end == begin + 1)
        {
            line.append(words[begin]);
            for (int i = begin + 1; i < end; i++)
            {
                line.append(" ").append(words[i]);
            }

            appendSpace(line, maxWidth - line.length());
        }
        else
        {
            int wordWidth = 0;
            for (int i = begin; i < end; i++)
            {
                wordWidth += words[i].length();
            }

            int totalSpace = maxWidth - wordWidth;
            int wordSpace = totalSpace / (end - begin - 1);
            int remainSpace = totalSpace % (end - begin - 1);

            for (int i = begin; i < end - 1; i++)
            {
                line.append(words[i]);
                appendSpace(line, wordSpace);
                if (remainSpace > 0)
                {
                    line.append(" ");
                    remainSpace--;
                }
            }

            line.append(words[end - 1]);
        }
        return line.toString();
    }

    private void appendSpace(StringBuilder line, int num)
    {
        for (int i = 0; i < num; i++)
        {
            line.append(" ");
        }
    }
}
```

