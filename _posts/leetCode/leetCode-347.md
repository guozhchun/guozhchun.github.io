---
title: leetCode-347:Top K Frequent Elements
date: 2018-06-30 21:36:47
tags: [leetCode, java]
categories: leetCode
---

# 问题描述

给出一个数组和一个数字k，要求找出数组中数字出现频率最高的k个数字。如果按照出现频率从高到低取数字时，发现在某个频率有 m 个相同的数字，在该频率前面共有 n 个数字，且 n + m > k，则从m个数字中任意取 k - n个数字，这 k - n个数字无顺序要求，也无特定数字要求。

此题要求时间复杂度小于O(nlog n)，其中 n 是数组的长度

<!-- more -->

# 样例输入输出

> 输入：nums = [][1,1,1,2,2,3] [1,1,1,2,2,3] , k = 2
>
> 输出：[1,2]

> 输入：nums = [1,1,1,3,3,3,2,2,2] , k = 2
>
> 输出：[1,2]或者[1,3]或者[2,3]。不能输出[1,2,3]

# 问题解法

此解法主要参考**[https://leetcode.com/problems/top-k-frequent-elements/discuss/81635/3-Java-Solution-using-Array-MaxHeap-TreeMap](https://leetcode.com/problems/top-k-frequent-elements/discuss/81635/3-Java-Solution-using-Array-MaxHeap-TreeMap)**

主要过程如下

* 先遍历数组，找出每个数字出现的次数，存放在map中
* 遍历map，用一个List数组来存放map中的数组，其中数组的下标是map的value值，表示数字出现的次数，数组下标对应的元素是一个List列表，存放数组中数字出现次数是这个下标值的数字。假设输入数组是[3,4,2,7,7,7,1,1,1,5,5,5,8,8,8,9,9,9,9,9]，则构造的数组如下图所示，数组下标5的位置存放数字9，说明9出现的次数是5次，数组下标位置3存放的数字列表是[7,1,5,8]，说明数字7、1、5、8出现的次数是3，数组下标位置1存放的数字列表是[3,4,2]，说明数字3、4、2出现的次数是1，数组下标位置4、6没有存放数字，说明没有数字出现的次数是4次或6次。

![leetCodde-347 bucket说明图](/images/leetCode347.png)

* 对第二步骤构造的数组，从后向前遍历，依次取出k个数字作为结果返回。例如以上图为例，假设k = 2，则返回[9,7]（数字9出现5次，数字7出现3此，当然也可以返回[9,1]或其他等价的结果），假设k = 1，则返回[9]，假设k = 5，则返回[9,7,1,5,8]（当然，也可以返回[9,7,5,1,8]或其他等价的结果）

整个过程时间复杂度为O(n)，低于题目的要求O(nlog n)

代码如下

```java
import java.util.Map.Entry;
class Solution 
{
    public List<Integer> topKFrequent(int[] nums, int k) 
    {
        // 步骤1：计算数字出现的次数
        Map<Integer, Integer> numsMap = new HashMap<>();
        for (int i = 0; i < nums.length; i++)
        {
            int numCount = numsMap.getOrDefault(nums[i], 0) + 1;
            numsMap.put(nums[i], numCount);
        }
        
        // 步骤2：构造 “数字出现次数-数字列表集合” 的List数组
        // numCountsBucket下标位置表示数字出现的个数
        // 下标位置对应的元素表示出现该次数的数字集合列表，列表为空时表示没有出现该数字
        // 由于数字出现次数最多为nums.length，所以构造的数组长度为nums.length + 1
        // 因此没有数字会出现 0 次，数组的 0 下标是预留的
        // 当然也可以构造nums.length长度的数组，只是后面取值时需要减一避免数组越界
        List<Integer>[] numCountsBucket = new List[nums.length + 1];
        for (Entry<Integer, Integer> entry : numsMap.entrySet())
        {
            int num = entry.getKey();
            int count = entry.getValue();
            List<Integer> numList = numCountsBucket[count];
            if (numList == null)
            {
                numList = new ArrayList<>();
                numList.add(num);
                numCountsBucket[count] = numList;
            }
            else 
            {
                numList.add(num);
            }
        }
        
        // 步骤3：从后往前遍历数组，依次取出k个数字作为结果集返回
        // 从出现次数最多的元素依次往下取，列表为空时，表示该没有数字出现该次数
        List<Integer> result = new ArrayList<>();
        for (int i = numCountsBucket.length - 1; i >= 0; i--)
        {
            List<Integer> numList = numCountsBucket[i];
            if (numList != null)
            {
                result.addAll(numList);
                k -= numList.size();
            }
            
            if (k == 0)
            {
                break;
            }
            // 由于可能m（m > k）个数字出现次数一样，此时取前k个即可，后续的数字不用返回
            // 例如输入数组[1,1,1,2,2,2,3,3,3]，k = 2，此时返回[1,2]或[1,3]或[2,3]
            // 而不应该返回[1,2,3]
            else if (k < 0)
            {
                while (k < 0)
                {
                    result.remove(result.size() - 1);
                    k++;
                }
                break;
            }
        }
        
        return result;
    }
}
```

# 参考资料

[https://leetcode.com/problems/top-k-frequent-elements/discuss/81635/3-Java-Solution-using-Array-MaxHeap-TreeMap](https://leetcode.com/problems/top-k-frequent-elements/discuss/81635/3-Java-Solution-using-Array-MaxHeap-TreeMap)