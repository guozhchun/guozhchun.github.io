---
title: leetCode-5:Longest Palindromic Substring
date: 2018-12-23 22:03:58
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个字符串，要求找出这个字符串中最长回文子字符串，如果有多个相同最大长度的回文子字符串，输出其中一个即可。题目链接：**[点我](https://leetcode.com/problems/longest-palindromic-substring/)**

<!-- more -->

# 样例输入输出

> 输入："babad"
>
> 输出："bab"    或者     "aba"

> 输入："cbbd"
>
> 输出："bb"

# 问题解法

此问题最简单的方式就是用暴力方法，即用两个 for 循环找出所有的子字符串，然后对每个字符串进行是否是回文字符串的判断。但是这样复杂度太高了，有 `O(n^3)`。因此需要采用其他的方法。

## 动态规划

用 `dp[i][j]` 表示由字符串的 `[i, j]` 位置构成的子字符的回文子字符串的长度。则可以得出以下的动态方程：

```
如果 dp[i + 1][j - 1] != 0，即 dp[i + 1][j - 1] 是一个回文字符串，并且 s[i] == s[j]
则   dp[i][j] = dp[i + 1][j - 1] + 2;
否则 dp[i][j] = 0
```

据此，可以写出如下代码

```java
class Solution {
    public String longestPalindrome(String s) 
    {
        if (s == null || s.length() == 0)
        {
            return "";
        }
        
        int length = s.length();
        // dp[i][j] 表示字符串的 [i, j] 范围组成的子字符串的回文字符串长度
        int[][] dp = new int[length][length];
        int maxLength = 1;
        int startIndex = 0;
        int endIndex = 0;

        for (int i = length - 1; i >= 0; i--)
        {
            for (int j = length - 1; j >= i; j--)
            {
                if (i == j)
                {
                    dp[i][j] = 1;
                    continue;
                }
                
                // 用 dp[i + 1][j - 1] != 0 来判断当前字符串去除首尾两个字符所组成的字符串是回文串
                // 由于当 i 和 j 相邻时， dp[i + 1][j - 1] == 0；所以需要将 i + 1 == j 单独拿出来作为一个判断条件
                if (s.charAt(i) == s.charAt(j) && (i + 1 == j || dp[i + 1][j - 1] != 0))
                {
                    dp[i][j] = dp[i + 1][j - 1] + 2;
                }

                if (dp[i][j] > maxLength)
                {
                    maxLength = dp[i][j];
                    startIndex = i;
                    endIndex = j;
                }
            }
        }
        
        return s.substring(startIndex, endIndex + 1);
    }
}
```

## 中心点扩展法

动态规划需要`O(n^2)`的空间，为了降低空间复杂度，可以参照将字符由中心位置向两侧依次递增构造回文字符串的做法进行求解。此方法主要是用一层循环遍历字符串每个字符，然后以这个字符为中心（或以这个字符和这个字符的下个字符为中心。主要是考虑奇偶数的情况），依次想两侧递增相同的字符，从而构成回文字符串。

此解法主要参考：[https://leetcode.com/articles/longest-palindromic-substring/](https://leetcode.com/articles/longest-palindromic-substring/)

代码如下：

```java
class Solution 
{
    public String longestPalindrome(String s)
    {
        if (s == null || s.length() == 0)
        {
            return "";
        }
      
        int length = s.length();
        int maxLength = 0;
        int startIndex = 0;
        int endIndex = 0;
        
        for (int i = 0; i < length; i++)
        {
            int len1 = getMaxLengthOfPalindromicString(s, i, i);
            int len2 = getMaxLengthOfPalindromicString(s, i, i + 1);
            int len = Math.max(len1, len2);
            if (len > maxLength)
            {
                maxLength = len;
                startIndex = i - (len - 1) / 2;
                endIndex = i + len / 2;
            }
        }
        
        return s.substring(startIndex, endIndex + 1);
    }
    
    private int getMaxLengthOfPalindromicString(String s, int left, int right)
    {
        int length = s.length();
        while (left >= 0 && right < length && s.charAt(left) == s.charAt(right))
        {
            left--;
            right++;
        }
        
        return right - left - 1;
    }
}
```

## Manacher 算法

此算法可以用`O(n)`的时间复杂度求解最长回文子字符串，解法主要参考：[https://articles.leetcode.com/longest-palindromic-substring-part-ii/](https://articles.leetcode.com/longest-palindromic-substring-part-ii/)

其主要过程如下：

* 先将字符串进行转换，在每个字符的左右位置插入标记字符 `#`，如：`abbc` 变为 `#a#b#b#c#`，这样做法主要是规避奇偶数的问题
* 用一个数组表示以新字符串中的每个字符为中心，向两边扩展，构成最大的回文字符串的过程中需要扩展的长度。例如：`#a#b#b#c#` 字符串对应的数组中各元素的值为 `0, 1, 0, 1, 2, 1, 0, 0, 0`。假设 `centerPos` 表示字符串中某个回文子字符串的中心位置，`rightPos` 表示上述回文字符串的右边边界，`i` 表示当前循环中指向字符位置，`minorI` 表示 `i` 关于 `centerPos` 的中心对称的位置。在计算数组元素值的过程中，主要有两个判断：
  * 如果 `i <= rightPos` 即 `i` 在当前回文子字符串中，且 `a[minorI] < rightPos - i `，即以 `minorI` 为中心构成的回文字符串在当前的回文子字符串中，那么回文串左右对称的特点可以得出以下结论：以 `i` 为中心构成的回文字符串与以 `minorI` 为中心构成的回文字符串相同，且一定在当前的回文子字符串中。因此 `a[i] == a[minorI]`
  * 如果不满足上述条件，那么则需要以当前位置为中心，依次向两边扩展，找到最大的回文字符串。

代码如下：

```java
class Solution 
{
    private String getReplaceString(String s)
    {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < s.length(); i++)
        {
            sb.append("#").append(s.charAt(i));
        }
        sb.append("#");
        return sb.toString();
    }
    
    public String longestPalindrome(String s)
    {
        if (s == null || s.length() == 0)
        {
            return "";
        }
        
        String newString = getReplaceString(s);
        int length = newString.length();
        int[] lens = new int[length];
        int centerPos = 1;
        int rightPos = 1;
        int maxLength = -1;
        int startIndex = 0;
        int endIndex = 0;
        for (int i = 1; i < length; i++)
        {
            int mirrorI = centerPos - (i - centerPos);
            if (i <= rightPos && lens[mirrorI] < rightPos - i)
            {
                lens[i] = lens[mirrorI];
                continue;
            }
            
            int tempLen = rightPos - i;
            if (tempLen < 0)  // rightPos < i
            {
                tempLen = 0;
            }
            while (i + tempLen + 1 < length && i - tempLen - 1 >= 0 && newString.charAt(i + tempLen + 1) == newString.charAt(i - tempLen - 1))
            {
                tempLen++;
            }
            lens[i] = tempLen;
            centerPos = i;
            rightPos = centerPos + lens[i];
            
            if (lens[i] > maxLength)
            {
                maxLength = lens[i];
                startIndex = (i - maxLength) / 2;
                endIndex = (i + maxLength) / 2 - 1;
            }
        }

        return s.substring(startIndex, endIndex + 1);
    }
}
```

# 参考资料

1. [https://leetcode.com/articles/longest-palindromic-substring/](https://leetcode.com/articles/longest-palindromic-substring/)
2. [https://articles.leetcode.com/longest-palindromic-substring-part-ii/](https://articles.leetcode.com/longest-palindromic-substring-part-ii/)