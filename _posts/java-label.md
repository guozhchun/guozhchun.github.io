---
title: java标签（label）的使用
date: 2019-04-13 10:30:50
tags: java
categories: java
---

# 前言

最近在看一些源码的过程中发现使用了标签，当时竟然看不懂，在搜索了一下之后才发现这是 java 标签的用法。这在平时工作中很少用到，甚至都记不起来 java 还有这一个特性。所以，趁着这次机会，把 java 标签复习和记录一下，以便后续再出现类似的情况时能快速找到资料进行复习。

<!-- more -->

# java 标签的形式

java 标签是由一个字符串名称和一个冒号组成的标识符，如：`label:`。其唯一其作用的地方是刚好在迭代语句之前。也就是说，在标签和迭代语句之间不能有任何其他的代码，如下所示

```java
label:  // can not write any other code here 
while (condition) {}

label: // can not write any other code here 
for (condition)
```

# java 标签的作用

java 标签只有配合迭代语句一起使用才能发挥作用，这也是其唯一其作用的地方。其主要作用是用于控制循环的跳转和中断循环。当然，这两个功能单独由 `continue` 和 `break` 语句也能做到。但是单独地使用 `continue` 和 `break` 语句只是针对一个循环的，并不能控制外层的嵌套循环的跳转。如果将 java 标签和 `continue`、`break` 语句结合使用，就能控制外层嵌套的循环跳转，而这也正是 java 标签的独特之处。

# java 标签的用法

java 标签是放在循环语句之前，与 `continue` 和 `break` 语句相结合使用的。其形式如下

```java
outerLabel:
outer-iteration
{
    ...
    innerLabel:
    inner-iteration
    {
        ...
        // 此语句作用相同于单独使用 ‘continue;’
        // 都是跳过本次内部的循环，进入下一次内部循环中
        continue innerLabel;
        ...
        // 此语句作用相当于单独使用 ‘break;’
        // 都是跳出内部循环，继续执行外层循环
        break innerLabel;
        ...
        // 此语句作用是调出内部循环，同时跳过本次的外层循环，进入下一次的外层循环中
        // 这是单独使用一个 ‘continue‘ 或 ’break’ 无法做到的
        // 如果要实现类似的功能，需要在此处设置一个标志位，然后使用 'break' 中断内部循环
        // 然后再在外部循环中对该标志位进行判断，同时使用 'continue' 语句跳过外部的循环
        // 这样操作起来不如直接使用 'continue outerLabel;' 简洁清晰
        continue outerLabel;
        ...
        // 此语句作用是跳出内部循环和外部循环，即直接结束整个外层循环体（终止了两个循环）
        // 这是单独使用一个 ‘continue‘ 或 ’break’ 无法做到的
        // 如果要实现类似的功能，需要在此处设置一个标志位，然后使用 'break' 中断内部循环
        // 然后再在外部循环中对该标志位进行判断，同时使用 'break' 语句跳出外部的循环
        // 这样操作起来不如直接使用 'break outerLabel;' 简洁清晰
        break outerLabel;
        ...
    }
    ...
}
```

# java 标签的样例

以下是使用 java 标签的样例。有 10 此外层循环和 3 次内层循环。为了方便观察验证 continue 和 break 的区别，每次执行到第二次内层循环时，进行相应的操作。

```java
public class TestLabel
{
    public static void main(String[] args)
    {
        System.out.println("====Begin====");
        outerLoop:
        for (int i = 0; i < 10; i++)
        {
            innerLoop:
            for (int j = 0; j < 3; j++)
            {
                if (i == 1 && j == 1)
                {
                    System.out.println("i = " + i + ", j = " + j + " -- continue innerLoop");
                    // 会跳过内部循环，开始下一轮的内部循环
                    continue innerLoop;  // same with continue;(without using label)
                }
                
                if (i == 2 && j == 1)
                {
                    System.out.println("i = " + i + ", j = " + j + " -- break innerLoop");
                    // 会结束内部循环，但是不影响外部循环
                    break innerLoop;  // same with break;(without using label)
                }
                
                if (i == 3 && j == 1)
                {
                    System.out.println("i = " + i + ", j = " + j + " -- continue outerLoop");
                    // 会跳过内部循环和外部循环，开始下一轮的外部循环
                    continue outerLoop;
                }
                
                if (i == 4 && j == 1)
                {
                    System.out.println("i = " + i + ", j = " + j + " -- break outerLoop");
                    // 会结束内部循环和外部循环，执行循环块后面的语句
                    break outerLoop;
                }
                
                System.out.println("i = " + i + ", j = " + j);
            }
            System.out.println("------------");
        }
        System.out.println("====End====");
    }
}
```

样例输出结果

```
====Begin====
i = 0, j = 0
i = 0, j = 1
i = 0, j = 2
------------
i = 1, j = 0
i = 1, j = 1 -- continue innerLoop
i = 1, j = 2
------------
i = 2, j = 0
i = 2, j = 1 -- break innerLoop
------------
i = 3, j = 0
i = 3, j = 1 -- continue outerLoop
i = 4, j = 0
i = 4, j = 1 -- break outerLoop
====End====
```

从输出结果可以看出：`continue label` 语句会直接跳过循环中该语句后面的部分，直接跳到 label 处，执行下一个循环，而 `break label` 语句会直接跳过循环中该语句后面的部分，直接跳到 label 处，然后结束循环，执行循环块后面的代码。

# 总结

1. java 标签必须与循环语句使用，是循环语句的前面一个语句，才能发挥其效果。
2. `continue label` 语句会跳过循环块中后面的语句，直接跳到 `label` 处，然后执行下一个循环。
3. `break label` 语句会跳过循环块中后面的语句，直接跳到 `label` 处，然后结束循环，执行循环代码块后面的语句。
4. 在嵌套循环中，要在内部循环中直接控制外部循环的跳转，使用 java 标签语句可以更加简洁清晰地完成功能，特别是对于嵌套了多层内部循环的情况下更能体现其威力。
5. 引申一下，可以设想每个循环都有一个标签（没写的系统默认补上），单独使用 `continue` 语句就是使用 `continue label` 语句（系统自动转换），单独使用 `break` 语句就是使用 `break label` 语句（系统自动转换），其效果是完全一样的。

# 参考资料

[1] Eckel,B. Java编程思想（第四版）[M]. 北京：机械工业出版社，2007.6