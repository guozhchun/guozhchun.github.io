---
title: leetCode-60:Permutation Sequence
date: 2018-06-10 21:36:17
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定两个数字n和k，要求找出[1,2,3,...,n]的全排列的组合中的的第k个组合，题目链接：**[点我](https://leetcode.com/problems/permutation-sequence/description/)**

<!-- more -->

# 样例输入输出

> 输入：3 2
>
> 输出："132"
>
> 解释：3的全排列如下
>
> 1. 123
> 2. 132
> 3. 213
> 4. 231
> 5. 312
> 6. 321

# 问题解法

此题最简单粗暴的做法从第一个开始找全排列的组合，直到找到第k个全排列组合结束。但是如果k很大时，很容易超时。所以应该换一种方式思考。

从给出的样例中可以看出，对n的全排列组合，是先将1放在第1位，然后将剩下的数字（除了1之外的数字）做全排列，然后将2放在第1位，将剩下的数字（除了2之外的数字）做全排列，再将3放在第1位，将剩下的数字（除了3之外的数字）做全排列...最后将n放在第1位，将剩下的数字（除了n之外的数字）做全排列。

因此，我们可以将第一位数字单独拿出来，将剩下的数字的全排列组合的数量计算出来，然后与k进行比较，看第一位的数字应该放哪个数字合适，此时确定完第一位数字后，就已经完成了m个全排列组合的查找，剩下的k - m个全排列组合在除第一位数字外的剩下的数字的全排列组合中重复上述过程查找。从中可以看出，这是一个递归过程。程序代码如下

```java
class Solution 
{
    // 0 ~ 9 的阶乘
    int[] factorialNums = {1, 1, 2, 6, 24, 120, 720, 5040, 40320};
    
    private List<Character> getNums(int n)
    {
        List<Character> nums = new LinkedList<>();
        for (int i = 0; i < n; i++)
        {
            nums.add(Character.valueOf((char)(i + '1')));
        }
        
        return nums;
    }
    
    private void findPermutation(List<Character> remainNums, int k, List<Character> result)
    {
        if (k == 1)
        {
            result.addAll(remainNums);
            return;
        }
        
        int remainNumsSize = remainNums.size();
        // k - 1是为了避免 k % factorialNums[remainNumsSize - 1] == 0的情况
        // 此种情况表示剩下的remainNumsSize - 1的数的所有全排列组合中的最后一个组合
        // 此时，不需要更换现在的第一个数字的值
        // 例如remianNums = [1,2,3],k = 4,此时factorialNums[remainNumsSize - 1] = 2
        // 1作为首位的全排列组合数是2，2作为首位的全排列组合数是2，此时k = 4刚好落在2作为首位时，
        // 剩下的数字[1,3]全排列组合的最后一个组合[3,1]，此时并不需要更换首位的数字2变成3
        // 也就是说，更换的次数只有1次（由1变成2）
        int fistNumChangetimes = (k - 1) / factorialNums[remainNumsSize - 1];
        int remainK = k - fistNumChangetimes * factorialNums[remainNumsSize - 1];
        result.add(remainNums.get(fistNumChangetimes));
        remainNums.remove(fistNumChangetimes);
        
        findPermutation(remainNums, remainK, result);
    }
    
    private String getResultString(List<Character> characters)
    {
        StringBuilder result = new StringBuilder();
        for (Character ch : characters)
        {
            result.append(ch);
        }
        
        return result.toString();
    }
    
    public String getPermutation(int n, int k) 
    {
        List<Character> nums = getNums(n);
        List<Character> resultList = new LinkedList<>();
        findPermutation(nums, k, resultList);
        String result = getResultString(resultList);
        return result;
    }
}
```

