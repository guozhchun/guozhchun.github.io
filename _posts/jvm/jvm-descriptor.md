---
title: jvm 描述符
date: 2018-08-02 22:42:57
tags: [java, jvm]
categories: jvm
---

# 定义

描述符是 jvm 层次上的概念，针对 class 文件定义而成，主要用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）。

<!-- more -->

# 字段数据类型描述符

对于字段的数据类型，其描述符主要有以下几种

* 基本数据类型（byte、char、double、float、int、long、short、boolean）：除 long 和 boolean，其他基本数据类型的描述符用对应单词的大写首字母表示。long 用 J 表示，boolean 用 Z 表示。
* void：描述符是 V。
* 对象类型：描述符用字符`L`加上对象的全限定名表示，如 `String` 类型的描述符为 `Ljava/lang/String`。
* 数组类型：每增加一个维度则在对应的字段描述符前增加一个 `[` ，如一维数组 `int[]` 的描述符为 `[I`，二维数组 `String[][]` 的描述符为 `[[java/lang/String` 。

各类型的描述符汇总如下

| 数据类型     | 描述符                                                       |
| ------------ | ------------------------------------------------------------ |
| byte         | B                                                            |
| char         | C                                                            |
| double       | D                                                            |
| float        | F                                                            |
| int          | I                                                            |
| long         | J                                                            |
| short        | S                                                            |
| boolean      | Z                                                            |
| 特殊类型void | V                                                            |
| 对象类型     | L + 对象全限定名。如 `Ljava/lang/String` 表示 `String` 类型  |
| 数组类型     | 每增加一个维度则在对应的字段描述符前增加一个 `[` ，如一维数组 `int[]` 的描述符为 `[I`，二维数组 `String[][]` 的描述符为 `[[java/lang/String` |

# 方法描述符

用描述符来描述方法时，其格式为：`(参数列表描述符)返回值描述符`。其中参数列表按照方法的参数从左到右依次用描述符对参数进行描述，如果没有参数则不用写。

例如：方法`void init()`的描述符为 `()init`，方法`java.lang.String getName()`的描述符为`()Ljava/lang/String`，方法`int indexOf(char[] source, int sourceOffset, int sourceCount, char[] target, int targetOffset, int targetCount, int fromIndex)`的描述符为`([CII[CIII)I`。

# 参考资料

[1] 周志明. 深入理解Java虚拟机：JVM高级特性与最佳实践[M]. 北京：机械工业出版社，2013