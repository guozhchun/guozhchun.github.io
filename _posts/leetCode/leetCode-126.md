---
title: leetCode-126:Word Ladder II
date: 2020-05-31 12:18:30
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定两个单词 `beginWord`、`endWord` 和一个单词列表 `wordList`，要求找出将单词 `beginWord` 变成单词 `endWord` 的最小的变化序列的列表。每次变化要求只能变化一个字母，而且变化后的单词必须在 `wordList` 中。题目链接：**[点我](https://leetcode.com/problems/word-ladder-ii/)**

<!-- more -->

# 样例输入输出

> 输入：beginWord = "hit",       endWord = "cog",      wordList = ["hot","dot","dog","lot","log","cog"]
>
> 输出：[ ["hit","hot","dot","dog","cog"], ["hit","hot","lot","log","cog"] ]

> 输入：beginWord = "hit",       endWord = "cog",      wordList = ["hot","dot","dog","lot","log","cog"]
>
> 输出：[]

# 问题解法

## 解法一：BFS

此题跟 [LeetCode127](https://guozhchun.github.io/leetCode/leetCode-127) 类似，只不过 LeetCode127 只需要找出最小序列的长度，而此题是要找出最小序列的所有集合。此题仍然可以用广搜的算法进行遍历，由于最终要输出的是最小序列的集合，因此在遍历过程中，存储在队列中的数据结构是序列（路径），而不是字符串（单词）。代码如下

```java
class Solution
{
    public List<List<String>> findLadders(String beginWord, String endWord, List<String> wordList)
    {
        if (!wordList.contains(endWord))
        {
            return new ArrayList<>();
        }

        // key: word
        // value: the word list which can transfer from the word with change only one letter
        Map<String, Set<String>> transferMap = new HashMap<>();

        wordList.add(0, beginWord);
        for (int i = 0; i < wordList.size(); i++)
        {
            String current = wordList.get(i);
            Set<String> currentWords = transferMap.getOrDefault(current, new HashSet<>());
            transferMap.put(current, currentWords);
            for (int j = i + 1; j < wordList.size(); j++)
            {
                String next = wordList.get(j);
                Set<String> nextWords = transferMap.getOrDefault(next, new HashSet<>());
                transferMap.put(next, nextWords);
                if (isValid(current, next))
                {
                    currentWords.add(next);
                    nextWords.add(current);
                }
            }
        }
        wordList.remove(0);

        Set<String> used = new HashSet<>();
        boolean isFind = false;
        List<List<String>> result = new LinkedList<>();
        Queue<List<String>> queue = new LinkedList<>();
        queue.offer(Arrays.asList(beginWord));
        while (!queue.isEmpty() && !isFind)
        {
            for (int i = queue.size(); i > 0; i--)
            {
                List<String> currentList = queue.poll();
                String currentWord = currentList.get(currentList.size() - 1);
                used.add(currentWord);
                if (currentWord.equals(endWord))
                {
                    result.add(currentList);
                    isFind = true;
                    continue;
                }

                // can not traversal the wordList, or it will be time Time Limit Exceeded
                for (String word : transferMap.get(currentWord))
                {
                    if (!used.contains(word))
                    {
                        List<String> next = new LinkedList<>(currentList);
                        next.add(word);
                        queue.offer(next);
                    }
                }
            }
        }

        return result;
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

            if (count > 1)
            {
                return false;
            }
        }

        return count == 1;
    }
}
```

## 解法二：BFS + DFS

此解法是先用 BFS 查找最短的序列长度，在查找过程中，将每次单词变化的列表存在一个 map 中。在找到最短序列长度结束 BFS 后，根据存储的 map 用 DFS 算法复原单词变化的路径。算法如下

```java
class Solution
{
    public List<List<String>> findLadders(String beginWord, String endWord, List<String> wordList)
    {
        List<List<String>> result = new LinkedList<>();
        if (!wordList.contains(endWord))
        {
            return result;
        }

        Map<String, List<String>> map = new HashMap<>();
        Queue<String> queue = new LinkedList<>();
        queue.offer(beginWord);
        boolean isFind = false;
        while (!queue.isEmpty() && !isFind)
        {
            // 同一层级的一起清除，避免同一层级的单词相互转换从而导致路径节点数增多
            Set<String> currentSet = new HashSet<>();
            for (int i = queue.size(); i > 0; i--)
            {
                String current = queue.poll();
                wordList.remove(current);
                currentSet.add(current);
            }

            for (String current : currentSet)
            {
                if (current.equals(endWord))
                {
                    isFind = true;
                    break;
                }

                for (String word : wordList)
                {
                    if (isValid(current, word))
                    {
                        List<String> nextWordList = map.getOrDefault(current, new LinkedList<>());
                        nextWordList.add(word);
                        map.put(current, nextWordList);
                        queue.offer(word);
                    }
                }
            }
        }

        List<String> chain = new LinkedList<>();
        chain.add(beginWord);
        solve(beginWord, endWord, map, chain, result);

        return result;
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

            if (count > 1)
            {
                return false;
            }
        }

        return count == 1;
    }

    private void solve(String beginWord, String endWord, Map<String, List<String>> map, List<String> chain, List<List<String>> result)
    {
        if (beginWord.equals(endWord))
        {
            result.add(new LinkedList<>(chain));
            return;
        }

        List<String> next = map.get(beginWord);
        if (next == null)
        {
            return;
        }

        for (String word : next)
        {
            chain.add(word);
            solve(word, endWord, map, chain, result);
            chain.remove(chain.size() - 1);
        }
    }
}
```

