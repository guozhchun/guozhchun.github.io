---
title: leetCode-89:Gray Code
date: 2019-04-06 23:22:07
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个非负整数 n ，要求输出该整数的二进制对应的格雷码序列（对于某个二进制数，改变其中一位数字，将其变成另一个全新的数字，这个数字在之前没出现过，由这些数字构成的序列称为格雷码序列），其中第一个数字必须以 0 开头。题目链接：**[点我](<https://leetcode.com/problems/gray-code/>)**

<!-- more -->

# 样例输入输出

> 输入：2
>
> 输出：[0,1,3,2]
>
> 解释：其过程如下
>
> 00
>
> 01
>
> 11
>
> 10
>
> 
>
> 也可以输出：[0,2,3,1]，其构造过程如下
>
> 00
>
> 10
>
> 11
>
> 01

> 输入：0
>
> 输出：[0]

# 问题解法

## 暴力破解法

此方法主要是将上一个产生的数字对应的二进制数，从左到右，依次改变其中一位的数字（由 0 变成 1，或由 1 变成 0），然后判断这新产生的数字是否在已经产生的数字的集合中，如果该数字存在集合中，则恢复改变当前位置的数字，改变下一个位置的数字，再次进行判断，如果不在则加入该集合，并将这个新产生的数字作为上一个产生的数字进行下一轮循环，直到找到所有的数字为止。代码如下

```java
class Solution 
{
    public List<Integer> grayCode(int n) 
    {
        int[] bits = new int[n];
        int count = 1 << n;
        List<Integer> nums = new ArrayList<>(count);
        nums.add(0);
        for (int i = 1; i < count; i++)
        {
            for (int j = 0; j < n; j++)
            {
                reverseBitNumber(bits, j);
                Integer num = getNum(bits);
                if (!nums.contains(num))
                {
                    nums.add(num);
                    break;
                }
                reverseBitNumber(bits, j);
            }
        }
        
        return nums;
    }
    
    // 将二进制数字转成十进制数字
    private int getNum(int[] bits)
    {
        int length = bits.length;
        int num = 0;
        for (int i = 0; i < length; i++)
        {
            num = (bits[i] << (length - i - 1)) | num;
        }
        
        return num;
    }
    
    // 翻转数组中某位的元素，由 0 变成 1，由 1 变成 0
    private void reverseBitNumber(int[] bits, int index)
    {
        if (bits[index] == 0)
        {
            bits[index] = 1;
        }
        else 
        {
            bits[index] = 0;
        }
    }
}
```

## 高位01对半增加法

此做法主要参考[https://leetcode.com/problems/gray-code/discuss/29891/Share-my-solution](https://leetcode.com/problems/gray-code/discuss/29891/Share-my-solution)。主要思想是：先构造 n - 1 的格雷码序列，然后将这个序列翻转产生另一个序列（这两个序列是关于某条直线对称的），然后在前一个序列的每个数字的前面补 0 构成数字 n 的格雷码序列的前半部分，在后一个序列的每个数字的前面补 1 构成数字 n 的格雷码序列的后半部分。

例如：对于数字 3，要想产生 3 的格雷码序列，需要先产生 2 的格雷码序列，要想产生 2 的格雷码序列，需要先产生 1 的格雷码序列，那么产生的顺序如下

```
1 的格雷码序列：
0
1

2 的格雷码序列：
00
01
------ 分割线，如果将上部分的最高为的 0 去掉，将下部分的最高位的 1 去到，那么上下两部分是关于这条线对称的，而且上半部分去掉最高位的 0 之后，剩下的内容就是数字 1 的格雷码序列
11
10

3 的格雷码序列：
000
001
011
010
------ 分割线，如果将上部分的最高为的 0 去掉，将下部分的最高位的 1 去到，那么上下两部分是关于这条线对称的，而且上半部分去掉最高位的 0 之后，剩下的内容就是数字 2 的格雷码序列
110
111
101
100
```

代码如下

```java
class Solution 
{
    public List<Integer> grayCode(int n) 
    {
        List<Integer> nums = new ArrayList<>(n);
        nums.add(0);
        for (int i = 0; i < n; i++)
        {
            for (int j = nums.size() - 1; j >= 0; j--)
            {
                Integer nextNum = nums.get(j) | (1 << i);
                nums.add(nextNum);
            }
        }
        
        return nums;
    }
}
```

# 参考资料

1. [https://leetcode.com/problems/gray-code/discuss/29891/Share-my-solution](https://leetcode.com/problems/gray-code/discuss/29891/Share-my-solution)