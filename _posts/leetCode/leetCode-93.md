---
title: leetCode-93:Restore IP Addresses
date: 2019-07-13 20:05:17
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给定一个只包含数字的字符串，要求找出这个字符串可以表示的 IP 的列表。题目链接：**[点我](https://leetcode.com/problems/restore-ip-addresses/)**

<!-- more -->

# 样例输入输出

> 输入：25525511135
>
> 输出：["255.255.11.135", "255.255.111.35"]

> 输入：10100
>
> 输出：["1.0.10.0", "10.1.0.0"]

# 问题解法

直接使用回溯算法，将原有的字符串拆分成四个子字符串，然后对这四个子字符串进行判断，如果能组成 IP，则将其拼接成 IP 放在结果集中。需要注意的是，为了防止输入的字符串长度过长导致递归超时，需要在进行递归前判断字符串长度是否有可能拆分成 IP。代码如下：

```java
class Solution 
{
    private boolean isNumsValid(List<String> nums)
    {
        for (String num : nums)
        {
            if (num.length() > 3 || (num.length() > 1 && num.charAt(0) == '0'))
            {
                return false;
            }
            
            if (Integer.parseInt(num) > 255)
            {
                return false;
            }
        }
        
        return true;
    }
    
    private void restoreIpAddresses(List<String> result, String str, int index, List<String> nums)
    {
        if (nums.size() == 3)
        {
            String lastNum = str.substring(index);
            nums.add(lastNum);
            if (isNumsValid(nums))
            {
                String ip = nums.get(0) + "." + nums.get(1) + "." + nums.get(2) + "." + nums.get(3);
                result.add(ip);
            }
            nums.remove(3);
            
            return;
        }
        
        for (int i = 1; i <= 3; i++)
        {
            int endIndex = index + i;
            if (endIndex >= str.length())
            {
                return;
            }
            
            String num = str.substring(index, endIndex);
            nums.add(num);
            restoreIpAddresses(result, str, endIndex, nums);
            nums.remove(nums.size() - 1);
        }
    }
    
    public List<String> restoreIpAddresses(String s) 
    {
        List<String> result = new LinkedList<>();
        if (s.length() < 4 || s.length() > 12)
        {
            return result;
        }
        
        restoreIpAddresses(result, s, 0, new ArrayList<>());
        return result;
    }
}
```

