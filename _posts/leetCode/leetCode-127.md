---
title: leetCode-127:Word Ladder
date: 2020-05-27 23:27:52
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定两个单词 `beginWord`、`endWord` 和一个单词列表 `wordList`，要求找出将单词 `beginWord` 变成单词 `endWord` 的最小的变化序列的长度。每次变化要求只能变化一个字母，而且变化后的单词必须在 `wordList` 中。题目链接：**[点我](https://leetcode.com/problems/word-ladder/)**

<!-- more -->

# 样例输入输出：

> 输入：beginWord = "hit",       endWord = "cog",      wordList = ["hot","dot","dog","lot","log","cog"]
>
> 输出：5
>
> 解释：变化的序列为："hit" -> "hot" -> "dot" -> "dog" -> "cog"

> 输入：beginWord = "hit",       endWord = "cog",      wordList = ["hot","dot","dog","lot","log","cog"]
>
> 输出：0
>
> 解释：cog 不在 wordList 中

# 问题解法

用广搜算法进行遍历搜索，在搜索过程中，为了避免重复搜索，可以用一个数组来标识已经搜索过的单词。代码如下：

```java
class Solution
{
    public int ladderLength(String beginWord, String endWord, List<String> wordList)
    {
        int step = 1;
        int length = wordList.size();
        boolean[] used = new boolean[length];
        Queue<String> queue = new LinkedList<>();
        queue.offer(beginWord);
        while (!queue.isEmpty())
        {
            for (int k = queue.size(); k > 0; k--)
            {
                String current = queue.poll();
                if (current.equals(endWord))
                {
                    return step;
                }

                for (int i = 0; i < length; i++)
                {
                    if (!used[i] && isValid(current, wordList.get(i)))
                    {
                        queue.offer(wordList.get(i));
                        used[i] = true;
                    }
                }
            }
            step++;
        }

        return 0;
    }

    private boolean isValid(String word, String other)
    {
        int count = 0;
        int length = word.length();
        for (int i = 0; i < length; i++)
        {
            if (word.charAt(i) != other.charAt(i))
            {
                count++;
            }
        }

        return count == 1;
    }
}
```

