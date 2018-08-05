---
title: jvm 类文件结构
date: 2018-07-29 20:10:40
tags: [java, jvm]
categories: jvm
---

# 前言

java 文件经过编译后会产生 class 文件，这是一个存储字节码的文件，它可以被 java 虚拟机解析并运行。事实上，java 虚拟机并不是只能解析 java 语言编译产生的 class 文件，它也可以解析其他语言产生的 class 文件，事实上，java 虚拟机是一种语言无关的平台，只要 class 文件满足 java 虚拟机特定的语法和结构化约束，就能在 java 虚拟机解析运行。

<!-- more -->

# 总述

class 文件是一组以 8 字节为基础单位的二进制流，各个数据项目严格按照规范和大端模式（高字节存放在地址低位，低字节存放在地址高位）的顺序紧凑地排列在 class 文件中，没有任何分隔符。

class 文件中只有两种数据类型，无符号数和表，其中无符号数用来描述数字、索引引用、数量值或者按照 UTF-8 编码构成字符串值。分别以 u1、u2、u4、u8 代表 1 个字节、2 个字节、4 个字节、8个字节的无符号数。表是由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯性地以 `_info` 结尾。

class 文件的总体数据结构如下所示，其主要的顺序是：文件类型信息  -->  文件版本号  -->  常量信息  -->  文件访问权限信息  -->  类索引  -->  父类索引  -->  接口索引信息  -->  字段表信息  -->  方法表信息  -->  属性表信息。

| 名称                | 描述                                           | 类型           | 数量                    |
| ------------------- | ---------------------------------------------- | -------------- | ----------------------- |
| magic               | “魔数”，用于确认文件是否能够被 java 虚拟机解析 | u4             | 1                       |
| minor_version       | class 文件次版本号                             | u2             | 1                       |
| major_version       | class 文件主版本号                             | u2             | 1                       |
| constant_pool_count | 常量池中常量数量                               | u2             | 1                       |
| constant_pool       | 常量池中的常量表                               | cp_info        | constant_pool_count - 1 |
| access_flags        | 访问标志，用于标识类或接口的访问信息           | u2             | 1                       |
| this_class          | 类索引                                         | u2             | 1                       |
| super_class         | 父类索引                                       | u2             | 1                       |
| interfaces_count    | 接口索引的数量                                 | u2             | 1                       |
| interfaces          | 接口索引                                       | u2             | interfaces_count        |
| fields_count        | 字段表数量                                     | u2             | 1                       |
| fields              | 字段表                                         | field_info     | fields_count            |
| methods_count       | 方法表数量                                     | u2             | 1                       |
| methods             | 方法表                                         | method_info    | methods_count           |
| attributes_count    | 属性表数量                                     | u2             | 1                       |
| attributes          | 属性表                                         | attribute_info | attributes_count        |

本文中将以下面的简单例子产生的字节码进行分析

```java
public class TestJvmClassStructure
{
    private int m;
    public int inc()
    {
        return m + 1;
    }
}
```

产生的字节码如下

```
cafe babe 0000 0034 0013 0a00 0400 0f09
0003 0010 0700 1107 0012 0100 016d 0100
0149 0100 063c 696e 6974 3e01 0003 2829
5601 0004 436f 6465 0100 0f4c 696e 654e
756d 6265 7254 6162 6c65 0100 0369 6e63
0100 0328 2949 0100 0a53 6f75 7263 6546
696c 6501 001a 5465 7374 4a76 6d43 6c61
7373 5374 7275 6374 7572 652e 6a61 7661
0c00 0700 080c 0005 0006 0100 1554 6573
744a 766d 436c 6173 7353 7472 7563 7475
7265 0100 106a 6176 612f 6c61 6e67 2f4f
626a 6563 7400 2100 0300 0400 0000 0100
0200 0500 0600 0000 0200 0100 0700 0800
0100 0900 0000 1d00 0100 0100 0000 052a
b700 01b1 0000 0001 000a 0000 0006 0001
0000 0001 0001 000b 000c 0001 0009 0000
001f 0002 0001 0000 0007 2ab4 0002 0460
ac00 0000 0100 0a00 0000 0600 0100 0000
0600 0100 0d00 0000 0200 0e
```

# “魔数”

“魔数”主要是用于确认这个文件是否能够被 java 虚拟机解析的文件。class 文件中主要使用文件开始的前 4 个字节表示“魔数”，只有当值是 0xcafebabe 时，才表明这个 class 文件是能够被虚拟机解析的文件。

使用“魔数”而不是文件的后缀名进行判断主要是出于安全方面的考虑，因为文件后缀名能够随意更改。现在有很多存储文件都使用“魔数”进行文件类别的校验，如 gif。文件格式的制定者可以自由地选择魔数值，只要这个魔数值没有被广泛使用并且不会产生二义性即可。

上述例子中，前四个字节`cafe babe`表明了这是一个 class 文件

# 版本号

跟在“魔数”后面的四个字节，即第 5 ~ 8 个字节，表明 class 文件的版本号。其中第 5、第 6 个字节表明 class 文件的次版本号，第 7、第 8 个字节表明文件的主版本号。这个版本号表明该 class 文件可以被高于或等于这个版本的虚拟机解析，使用 JDK8 编译产生的版本号是 0x00000034，表明可以被 JDK8 或高于 JDK8 以上的虚拟机解析执行。

上述例子中，第 5 ~ 8 个字节`0000 0034`表明这适用于 JDK8 及以上版本的虚拟机解析运行。

# 常量池

常量池是跟在版本号后面的字节码。其主要存放两种数据：字面量（Literal）和符号引用（Symbolic References）。其中字面量类似于 java 语言的常量，而符号引用则属于编译原理方面的概念，主要包括：类和接口的全限定名、字段的名称和描述符、方法的名称和描述符。

由于常量池中的常量数量不固定，因此在常量吃入口有一项 u2 类型的数据代表常量池中常量的数量（称为常量池容量）。但是有一点需要注意，其技术从 1 开始而不是从 0 开始，这就意味者如果常量池容量为 10，则常量的数量为 10 - 1 = 9。class 文件结构中只有常量池的容量计数是从 1 开始，对于其他集合类型，包括接口索引集合、字段表集合、方法表集合等的容量计数都是从 0 开始的。

常量池中共有 14 中常量，其结构如下所示

| 常量                                  | 项目                             | 类型 | 描述                                                         |
| ------------------------------------- | -------------------------------- | ---- | ------------------------------------------------------------ |
| CONSTANT_Utf8_info                    | tag                              | u1   | 区分常量类型，值为 1                                         |
|                                       | length                           | u2   | UTF-8 编码的字符串占用的字节数                               |
|                                       | bytes                            | u1   | 长度为 length 的 UTF-8 编码的字符串                          |
| CONSTANT_Integer_info                 | tag                              | u1   | 区分常量类型，值为 3                                         |
|                                       | bytes                            | u4   | 按照高位在前（大端模式）存储的 int 值                        |
| CONSTANT_Float_info                   | tag                              | u1   | 区分常量类型，值为 4                                         |
|                                       | bytes                            | u4   | 按照高位在前（大端模式）存储的 float 值                      |
| CONSTNAT_Long_info                    | tag                              | u1   | 区分常量类型，值为 5                                         |
|                                       | bytes                            | u8   | 按照高位在前（大端模式）存储的 long 值                       |
| CONSTANT_Double_info                  | tag                              | u1   | 区分常量类型，值为 6                                         |
|                                       | bytes                            | u8   | 按照高位在前（大端模式）存储的 double 值                     |
| CONSTANT_Class_info                   | tag                              | u1   | 区分常量类型，值为 7                                         |
|                                       | index                            | u2   | 指向全限定名常量项的索引                                     |
| CONSTANT_String_info                  | tag                              | u1   | 区分常量类型，值为 8                                         |
|                                       | index                            | u2   | 指向字符串字面量的索引                                       |
| CONSTANT_Fieldref_info                | tag                              | u1   | 区分常量类型，值为 9                                         |
|                                       | index                            | u2   | 指向声明字段的类或接口描述符 CONSTANT_Class_info 的索引项    |
|                                       | index                            | u2   | 指向字段描述符 CONSTANT_NameAndType 的索引项                 |
| CONSTANT_Method-<br>ref_info          | tag                              | u1   | 区分常量类型，值为 10                                        |
|                                       | index                            | u2   | 指向声明方法的类描述符 CONSTANT_Class_info 的索引项          |
|                                       | index                            | u2   | 指向名称及类型描述符 CONSTANT_NameAndType 的索引项           |
| CONSTANT_Interface-<br>Methodref_info | tag                              | u1   | 区分常量类型，值为 11                                        |
|                                       | index                            | u2   | 指向声明方法的类描述符 CONSTANT_Class_info 的索引项          |
|                                       | index                            | u2   | 指向名称及类型描述符 CONSTANT_NameAndType 的索引项           |
| CONSTANT_Name-<br>AndType_info        | tag                              | u1   | 区分常量类型，值为 12                                        |
|                                       | index                            | u2   | 指向字段或方法名称常量项的索引                               |
|                                       | index                            | u2   | 指向字段或方法描述符常量项的索引                             |
| CONSTANT_Method-<br>Handle_info       | tag                              | u1   | 区分常量类型，值为 15                                        |
|                                       | reference_kind                   | u1   | 值必须在 1 ~9 之间（包括 1 和 9），它决定了方法句柄的类型。方法句柄类型的值表示方法句柄的字节码行为 |
|                                       | reference_index                  | u2   | 值必须是对常量池的有效索引                                   |
| CONSTANT_Method-<br>Type_info         | tag                              | u1   | 区分常量类型，值为 16                                        |
|                                       | reference_kind                   | u1   | 值必须是对常量池的有效索引，常量池在该索引处的项必须是 CONSTANT_Utf8_info 结构，表示方法的描述符 |
| CONSTANT_Invoke-<br>Dynamic_info      | tag                              | u1   | 区分常量类型，值为 18                                        |
|                                       | bootstrap_me-<br>thod_attr_index | u2   | 值必须是对当前 Class 文件中引导方法表的 bootstrap_methods[] 数组的有效索引 |
|                                       | name_and_type_index              | u2   | 值必须是对当前常量池的有效索引，常量池在该索引处必须是 CONSTANT_NameAndType_info 结构，表示方法名和方法描述符 |

上述例子中，第 9 ~ 181 个字节表示的是常量池中的常量内容，具体表示含义如下

```
00 13                     常量池中的数量，值是19，说明有18个常量项
0a                        CONSTANT_Methodref_info常量项，tag
00 04                     指向声明方法的类描述符CONSTANT_Class_info的索引项, index, 值是4
00 0f                     指向名称及类型描述符CONSTANT_NameAndType_info的索引项, index, 值是15
09                        CONSTANT_Fieldref_info常量项，tag
00 03                     指向声明字段的类或接口描述符CONSTANT_Class_info的索引项, index, 值是3
00 10                     指向字段描述符CONSTANT_NameAndType_info的索引项, index, 值是16
07                        CONSTANT_Class_info常量项，tag
00 11                     指向全限定名称常量项的索引，index, 值是17
07                        CONSTANT_Class_info常量项，tag
00 12                     指向全限定名称常量项的索引，index, 值是18，
01                        CONSTANT_Utf8_info常量项，tag
00 01                     UTF-8编码的字符串占用的字节数，length, 值是1
6d                        长度为 1 的UTF-8编码字符串，bytes, 值是m
01                        CONSTANT_Utf8_info常量项，tag
00 01                     UTF-8编码的字符串占用的字节数，length, 值是1
49                        长度为 1 的UTF-8编码字符串，bytes, 值是I
01                        CONSTANT_Utf8_info常量项，tag
00 06                     UTF-8编码的字符串占用的字节数，length, 值是6
3c 69 6e 69 74 3e         长度为 6 的UTF-8编码字符串，bytes, 值是<init>
01                        CONSTANT_Utf8_info常量项，tag
00 03                     UTF-8编码的字符串占用的字节数，length, 值是3
28 29 56                  长度为 3 的UTF-8编码字符串，bytes, 值是()V
01                        CONSTANT_Utf8_info常量项，tag
00 04                     UTF-8编码的字符串占用的字节数，length, 值是4
43 6f 64 65               长度为 4 的UTF-8编码字符串，bytes, 值是Code
01                        CONSTANT_Utf8_info常量项，tag
00 0f                     UTF-8编码的字符串占用的字节数，length, 值是15
4c 69 6e 65 4e 75 6d 62 65 72 54 61 62 6c 65    长度为 15 的UTF-8编码字符串，bytes, 值是LineNumberTable
01                                              CONSTANT_Utf8_info常量项，tag
00 03                                           UTF-8编码的字符串占用的字节数，length, 值是3
69 6e 63                                        长度为 3 的UTF-8编码字符串，bytes, 值是inc
01                                              CONSTANT_Utf8_info常量项，tag
00 03                                           UTF-8编码的字符串占用的字节数，length, 值是3
28 29 49                                        长度为 3 的UTF-8编码字符串，bytes, 值是()I
01                                              CONSTANT_Utf8_info常量项，tag
00 0a                                           UTF-8编码的字符串占用的字节数，length, 值是10
53 6f 75 72 63 65 46 69 6c 65                   长度为 3 的UTF-8编码字符串，bytes, 值是SourceFile
01                                              CONSTANT_Utf8_info常量项，tag
00 1a                                           UTF-8编码的字符串占用的字节数，length, 值是26
54 65 73 74 4a 76 6d 43 6c 61 73 73 53 74 72 75 63 74 75 72 65 2e 6a 61 76 61       长度为 26 的UTF-8编码字符串，bytes, 值是TestJvmClassStructure.java
0c                                                      CONSTANT_NameAndType_info常量项, tag 
00 07                                                   指向该字符安或方法名称常量项的索引项, index, 值是7
00 08                                                   指向该字符安或方法描述符常量项的索引项, index, 值是8
0c                                                      CONSTANT_NameAndType_info常量项, tag 
00 05                                                   指向该字符安或方法名称常量项的索引项, index, 值是5
00 06                                                   指向该字符安或方法描述符常量项的索引项, index, 值是6
01                                                      CONSTANT_Utf8_info常量项，tag
00 15                                                   UTF-8编码的字符串占用的字节数，length, 值是21
54 65 73 74 4a 76 6d 43 6c 61 73 73 53 74 72 75 63 74 75 72 65      长度为 21 的UTF-8编码字符串，bytes, 值是TestJvmClassStructure
01                                                      CONSTANT_NameAndType_info常量项, tag
00 10                                                   UTF-8编码的字符串占用的字节数，length, 值是16
6a 61 76 61 2f 6c 61 6e 67 2f 4f 62 6a 65 63 74         长度为 16 的UTF-8编码字符串，bytes, 值是java/lang/Object
```

执行 javap 命令对 class 文件进行反编译，可以看到其常量池如下所示，与上述字节码的解析相同

```java
Constant pool:
   #1 = Methodref          #4.#15         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#16         //  TestJvmClassStructure.m:I
   #3 = Class              #17            //  TestJvmClassStructure
   #4 = Class              #18            //  java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               inc
  #12 = Utf8               ()I
  #13 = Utf8               SourceFile
  #14 = Utf8               TestJvmClassStructure.java
  #15 = NameAndType        #7:#8          //  "<init>":()V
  #16 = NameAndType        #5:#6          //  m:I
  #17 = Utf8               TestJvmClassStructure
  #18 = Utf8               java/lang/Object
```

# 文件访问标志

在常量池结束之后，紧接着的两个字节代表访问标志（access_flags），这个标志用于识别一些类或者接口层次的访问信息，包括：这个 class 是类还是接口；是否定义为 public 类型；是否定义为 abstract 类型；如果是类的话，是否被声明为 final 等。具体的标志位及其标志的含义如下所示

| 标志名称       | 标志值 | 含义                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| ACC_PUBLIC     | 0x0001 | 是否为 public 类型，如果是，取值为 0x0001，否则为 0x0000     |
| ACC_FINAL      | 0x0010 | 是否被声明为 final，如果是，取值为 0x0010，否则为 0x0000，只有类可以设置 |
| ACC_SUPER      | 0x0020 | 是否允许使用 invokespecial字节码指令的新语义， invokespecial 指令的语义在 JDK 1.0.2 发生过改变，为了区别这条指令使用哪种语义， JDK 1.0.2 之后编译出来的类的这个标志都必须为真 |
| ACC_INTERFACE  | 0x0200 | 是否是一个接口，如果是，取值为 0x0200，否则为 0x0000         |
| ACC_ABSTARCT   | 0x0400 | 是否为 abstract 类型，对于接口或者抽象类来说，此值为 0x0400，其他类值为 0x0000 |
| ACC_SYNTHETIC  | 0x1000 | 该 class 文件是否非用户代码产生，如果是，取值为 0x1000，否则为 0x0000 |
| ACC_ANNOTATION | 0x2000 | 是否是一个注解，如果是，取值为 0x2000，否则为 0x0000         |
| ACC_ENUM       | 0x4000 | 是否是一个枚举，如果是，取值为 0x4000，否则为 0x0000         |

对于这些标志位，可能只会用到一部分，对于用到的标志位，将其标志值进行或运算得到最终的标志值，对于没用到的标志位，其值不参与运算（或者说其值为 0）。

对于上述的例子，第 182 ~ 183 个字节是`00 21`，是由 0x0001 和 0x0020 或运算得到的，所以ACC_PUBLIC、ACC_SUPER为真，ACC_FINAL、ACC_INFERFACE、ACC_ABSTRACT、ACC_SYNTHETIC、ACC_ANNOTATION、ACC_ENUM为假，即说明这是一个非 final 的被 public 修饰的类。

# 类索引、父类索引与接口索引集合

类索引（this_class）和父类索引（super_class）都是一个 u2 类型的数据，而接口索引集合（interfaces）是一组 u2 类型的数据的集合。class 文件中由这三项数据来确定这个类的继承关系，其中类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名，接口索引集合用于描述这个类实现的接口。

类索引和父类索引都指向一个类型为 CONSTANT_Class_info 的类描述符常量，通过 CONSTANT_Class_info 类型的常量中的索引值可以找到定义在 CONSTANT_Utf8_info 类型的常量中的全限定名字符串。

上述例子中，第 184 ~ 187 个字节表示类索引和父类索，引对应的值如下

```
00 03    本类索引，this_class，对应到第 3 个常量项，得知指向第 17 个常量项，值是TestJvmClassStructure
00 04    父类索引，super_calss，对应到第 4 个常量项，得知指向第 18 个常量项，值是java/lang/Object
```

对于接口索引集合，首先用一个 u2 的数据类型的数据来表示接口的数量 n ，紧接着跟着 n 个 u2 数据类型的接口索引，跟类索引和父类索引一样，也是指向一个类型为 CONSTANT_Class_info 的类描述符常量，通过 CONSTANT_Class_info 类型的常量中的索引值可以找到定义在 CONSTANT_Utf8_info 类型的常量中的全限定名字符串。然而，如果表示接口数量的 u2 数据类型的数据为 0，则后面接口的索引表不占任何字节，即后面直接跟着字段表集合。

上述例子中，第 188 ~ 189 个字节表示接口索引信息，由于没有接口，故接口数量为 0，其字节码如下

```
00 00    接口索引集合大小，值是0，表示没有实现接口
```

# 字段表集合

字段表（field_info）用于描述接口或者类中声明的变量。字段包括类级变量以及实例级变量，但是不包括在方法内部声明的局部变量。字段表集合中不会列出从父类或者父接口中继承而来的字段，但有可能列出原本 java 代码中不存在的字段，比如在内部类中为了保持对外部类的访问性，为自动添加指向外部类实例的字段。

字段表的结构如下所示

| 名称             | 类型           | 数量             | 描述                                         |
| ---------------- | -------------- | ---------------- | -------------------------------------------- |
| access_flags     | u2             | 1                | 字段修饰符，用于判断字段的访问权限等信息     |
| name_index       | u2             | 1                | 对常量池中常量的引用，表示字段的简单名称     |
| descriptor_index | u2             | 1                | 对常量池中常量的引用，表示字段和方法的描述符 |
| attributes_count | u2             | 1                | 属性表中属性的数量                           |
| attributes       | attribute_info | attributes_count | 属性表中属性的信息                           |

其中 access_flags 与文件访问标志类似，其定义的标志名称的值在该标志位为真时取对应的值，为假时取 0，最终将所有标志位进行与运算得到最后结果。其具体项及其含义如下所示

| 标志名称      | 标志值 | 含义                                                         |
| ------------- | ------ | ------------------------------------------------------------ |
| ACC_PUBLIC    | 0x0001 | 字段是否为 public 类型，如果是，取值为 0x0001，否则为 0x0000 |
| ACC_PRIVATE   | 0x0002 | 字段是否是 private 类型，如果是，取值为 0x0002，否则为 0x0000 |
| ACC_PROTECTED | 0x0004 | 字段是否是 protected 类型，如果是，取值为 0x0004，否则为 0x0000 |
| ACC_STATIC    | 0x0008 | 字段是否是 static 类型，如果是，取值为 0x0008，否则为 0x0000 |
| ACC_FINAL     | 0x0010 | 字段是否是 final 类型，如果是，取值为 0x0010，否则为 0x0000  |
| ACC_VOLATILE  | 0x0040 | 字段是否是 volatile 类型，如果是，取值为 0x0040，否则为 0x0000 |
| ACC_TRANSIENT | 0x0080 | 字段是否是transient 类型，如果是，取值为 0x0080，否则为 0x0000 |
| ACC_SYNTHETIC | 0x1000 | 字段是否由编译器自动产生，如果是，取值为 0x1000，否则为 0x0000 |
| ACC_ENUM      | 0x4000 | 字段是否是 enum 类型，如果是，取值为 0x4000，否则为 0x0000   |

上述例子中，第 190 ~ 199 个字节表示字段表信息，其具体含义如下

```
00 01        字段表中字段的数量，值是1
00 02        字段访问标志，access_flags，0x0002表示ACC_PRIVATE为真，其他标志符为假
00 05        字段的简单名称，name_index，对应到第 5 个常量，值是m
00 06        字段的描述符，descriptor_index，对应到第 6 个常量，值是I
00 00        字段的属性表中属性数量，值是0
```

# 方法表集合

方法表的结构与字段表一样，依次包括了访问标志（access_flags）、名称索引（name_index）、描述符索引（descriptor_index）、属性表集合（attributes）几项。其含义如下所示

| 名称             | 类型           | 数量             | 描述                                         |
| ---------------- | -------------- | ---------------- | -------------------------------------------- |
| access_flags     | u2             | 1                | 方法修饰符，用于判断方法的访问权限等信息     |
| name_index       | u2             | 1                | 对常量池中常量的引用，表示方法的简单名称     |
| descriptor_index | u2             | 1                | 对常量池中常量的引用，表示字段和方法的描述符 |
| attributes_count | u2             | 1                | 属性表中属性的数量                           |
| attributes       | attribute_info | attributes_count | 属性表中属性的信息                           |

其中 access_flags 与文件访问标志类似，其定义的标志名称的值在该标志位为真时取对应的值，为假时取 0，最终将所有标志位进行与运算得到最后结果。其具体项及其含义如下所示

| 标志名称         | 标志值 | 含义                                                         |
| ---------------- | ------ | ------------------------------------------------------------ |
| ACC_PUBLIC       | 0x0001 | 方法是否为 public 类型，如果是，取值为 0x0001，否则为 0x0000 |
| ACC_PRIVATE      | 0x0002 | 方法是否是 private 类型，如果是，取值为 0x0002，否则为 0x0000 |
| ACC_PROTECTED    | 0x0004 | 方法是否是 protected 类型，如果是，取值为 0x0004，否则为 0x0000 |
| ACC_STATIC       | 0x0008 | 方法是否是 static 类型，如果是，取值为 0x0008，否则为 0x0000 |
| ACC_FINAL        | 0x0010 | 方法是否是 final 类型，如果是，取值为 0x0010，否则为 0x0000  |
| ACC_SYNCHRONIZED | 0x0020 | 方法是否是 synchronized 类型，如果是，取值为 0x0020，否则为 0x0000 |
| ACC_BRIDGE       | 0x0040 | 方法是否由编译器产生的桥接方法，如果是，取值为 0x0040，否则为 0x0000 |
| ACC_VARAGS       | 0x0080 | 方法是否接收不定参数，如果是，取值为 0x0080，否则为 0x0000   |
| ACC_NATIVE       | 0x0100 | 方法是否是 native 类型，如果是，取值为 0x0100，否则为 0x0000 |
| ACC_ABSTRACT     | 0x0400 | 方法是否是 abstract 类型，如果是，取值为 0x0400，否则为 0x0000 |
| ACC_STRICTFP     | 0x0800 | 方法是否是 strictfp 类型，如果是，取值为 0x0800，否则为 0x0000 |
| ACC_SYNTHETIC    | 0x1000 | 方法是否由编译器自动产生，如果是，取值为 0x1000，否则为 0x0000 |

需要注意的是，虽然实例构造函数 `<init>` 和类构造函数 `<clinit>` 也可以由编译器自动生成，但是，这两个函数的 ACC_SYNTHETIC 仍然为假。另外，如果父类方法在子类中没有被重写（override），方法表集合中就不会出现来自父类的方法信息。

上述例子中，第 200 ~ 289 个字节表示方法表的信息，其具体含义如下

```
00 02          方法表中方法的数量，值是2
00 01          方法访问标志，access_flags，0x0001表示ACC_PUBLIC为真，其他标志符为假
00 07          方法的简单名称，name_index，对应到第 7 个常量，值是<init>
00 08          方法的描述符，descriptor_index，对应到第 8 个常量，值是()V
00 01          方法的属性表中属性数量，值是1
00 09          属性名称索引，attribute_name_index，对应到第 9 个常量，值是Code
00 00 00 1d    属性字节长度，attribute_length，值是29
00 01          Code属性中操作数栈深度的最大值，max_stack，值是1
00 01          Code属性中局部变量表所需的存储空间，max_locals，单位是Slot，值是1
00 00 00 05    字节码指令字节长度，值是5
2a             虚拟机指令 aload_0，将第一个引用类型本地变量推送值栈顶
b7             虚拟机指令 invokespecial，实例初始化方法，构造函数调用，后续有个 u2 类型参数说明调用的是哪个方法
00 01          invokespecial 参数，指向常量池中第 1 个常量，得知值是java/lang/Object."<init>":()V
b1             虚拟机指令 return，从当前方法返回 void
00 00          Code属性中异常表数量，exception_table_length，值是0
00 01          Code属性中属性数量，attributes_count，值是1
00 0a          属性名称索引，attribute_name_index，对应到第 10 个常量，值是LineNumberTable
00 00 00 06    属性字节长度，attribute_length，值是6
00 01          LineNumberTable属性中line_number_table数量，line_number_table_length，值是1
00 00 00 01    LineNumberTable属性中line_number_info表，包括一个u2的start_pc和一个u2的line_number，分别表示字节码相对于方法体开始的偏移量（值是0），后者表示java源码的行号（值是1）

               // init方法完毕

00 01          方法访问标志，access_flags，0x0001表示ACC_PUBLIC为真，其他标志符为假
00 0b          方法的简单名称，name_index，对应到第 11 个常量，值是inc
00 0c          方法的描述符，descriptor_index，对应到第 12 个常量，值是()I
00 01          方法的属性表中属性数量，值是1
00 09          属性名称索引，attribute_name_index，对应到第 9 个常量，值是Code
00 00 00 1f    属性字节长度，attribute_length，值是31
00 02          Code属性中操作数栈深度的最大值，max_stack，值是2
00 01          Code属性中局部变量表所需的存储空间，max_locals，单位是Slot，值是1
00 00 00 07    字节码指令字节长度，值是7
2a             虚拟机指令 aload_0，将第一个引用类型本地变量推送值栈顶
b4             虚拟机指令 getfield，获取指定类的实例域，并将其值压入栈顶，后续有个u2类型参数说明压入栈顶的是哪个变量
00 02          getfield 参数，指向常量池中第 2 个常量，得知值是TestJvmClassStructure.m:I
04             虚拟机指令 iconst_1，将int型 1 推送至栈顶
60             虚拟机指令 iadd，将栈顶两个 int 型相加，并将结果压入栈顶
ac             虚拟机指令 ireturn，从当前方法返回 int
00 00          Code属性中异常表数量，exception_table_length，值是0
00 01          Code属性中属性数量，attributes_count，值是1
00 0a          属性名称索引，attribute_name_index，对应到第 10 个常量，值是LineNumberTable
00 00 00 06    属性字节长度，attribute_length，值是6
00 01          LineNumberTable属性中line_number_table数量，line_number_table_length，值是1
00 00 00 06    LineNumberTable属性中line_number_info表，包括一个u2的start_pc和一个u2的line_number，分别表示字节码相对于方法体开始的偏移量（值是0），后者表示java源码的行号（值是6）
```

# 属性表集合

属性表是比较“自由”的一种数据结构，其不同于字段表或方法表等有严格的顺序限制，其可以自由地定义顺序，并且只要不与已有属性名重复，任何编译器都可以向属性表中写入自己定义的独特的不同于其他编译器的属性信息，在解析时，java 虚拟机会自动忽略其不认识的属性。因此，不同的厂商实现的编译器的属性表会有所不同。

为了能够正确解析 class 文件，java 虚拟机规范中预定义了一些属性，其一部分属性及其含义如下所示

| 属性名称                                 | 使用位置           | 含义                                                         |
| ---------------------------------------- | ------------------ | ------------------------------------------------------------ |
| Code                                     | 方法表             | java 代码编译成的字节码指令                                  |
| ConstantValue                            | 字段表             | final 关键字定义的常量值                                     |
| Deprecated                               | 类、方法表、字段表 | 被声明为 deprecated 的方法和字段                             |
| Exceptions                               | 方法表             | 方法抛出的异常                                               |
| EnclosingMethod                          | 类文件             | 仅当一个类为局部类或者匿名类时才能拥有这个属性，这个属性用于标识这个类所在的外围方法 |
| InnerClasses                             | 类文件             | 内部类列表                                                   |
| LineNumberTable                          | Code属性           | java 源码的行号与字节码指令的对应关系                        |
| LocalVariableTable                       | Code属性           | 方法的局部变量描述                                           |
| StackMapTable                            | Code属性           | JDK 1.6 中新增的属性，供新的类型检查验证器（Type Checker）检查和处理目标方法的局部变量和操作数栈所需要的类型是否匹配 |
| Signature                                | 类、方法表、字段表 | JDK 1.5 中新增的属性，这个属性用于支持泛型情况下的方法签名，在 java 语言中，任何类、接口、初始化方法或成员的泛型签名如果包含了类型变量（Type Variables）或参数化类型（Parameerized Types），则 Signature 属性会为它记录泛型签名信息。由于 java 的泛型采用擦除法实现，在为了避免类型信息被擦除后导致签名混乱，需要这个属性记录泛型中的相关信息 |
| SourceFile                               | 类文件             | 记录源文件名称                                               |
| SourceDebugExtension                     | 类文件             | JDK 1.6 中新增的属性，SourceDebugExtension 属性用于存储额外的调试信息。譬如在进行 JSP 文件调试时，无法通过 java 堆栈来定位到 JSP 文件的行号，JSR-45 规范为这些非 java 语言编写，却需要编译成字节码并运行在 java 虚拟机中的程序提供了一个进行调试的标准机制，使用 SourceDebugExtension 属性就可以用于存储这个标准所新加入的调试信息 |
| Synthetic                                | 类、方法表、字段表 | 标识方法或字段为编译器自动生成的                             |
| LocalVariableTypeTable                   | 类                 | JDK 1.5 中新增的属性，它使用特征签名代替描述符，是为了引入泛型语法之后能描述泛型参数化类型 |
| RuntimeVisibleAnnotations                | 类、方法表、字段表 | JDK 1.5 中新增的属性，为动态注解提供支持。RuntimeVisibleAnnotations 属性用于指明哪些注解是运行时（实际上运行时就是进行反射调用）可见的 |
| RuntimeInvisibleAnnotations              | 类、方法表、字段表 | JDK 1.5 中新增的属性，与 RuntimeVisibleAnnotations 属性作用刚好相反，用于指明哪些注解是运行时不可见的 |
| RuntimeVisibleParameter-<br>Annotaions   | 方法表             | JDK 1.5 中新增的属性，与 RuntimeVisibleAnnotations 属性类似，只不过作用对象为方法参数 |
| RuntimeInvisibleParameter-<br>Annotaions | 方法表             | JDK 1.5 中新增的属性，与 RuntimeInvisibleAnnotations属性类似，只不过作用对象为方法参数 |
| AnnotationDefault                        | 方法表             | JDK 1.5 中新增的属性，用于记录注解类元素的默认值             |
| BootstrapMethods                         | 类文件             | JDK 1.7 中新增的属性，用于保存 invokedynamic 指令引用的引导方法限定符 |

对于每个属性，它的名称需要从常量池中引用一个 CONSTANT_Utf8_info 类型的常量来表示，而属性值的结构则是自己定义的，值需要通过一个 u4 的长度属性说明属性值占用的位数即可。属性表的结构如下

| 名称                 | 类型 | 数量 | 描述 |
| -------------------- | ---- | ---- | ---- |
| attribute_name_index | u2   | 1    |对常量池中常量的引用，表示属性类型名称|
| attribute_length | u4 | 1 |该类型属性所占的字节码长度|
| info | u1 | attribute_length |该类型属性的具体字节码|

## Code 属性

Code属性主要是用于存放 java 方法体的相关内容的字节码信息。其结构如下所示

| 名称                   | 类型           | 数量                   | 描述                                                         |
| ---------------------- | -------------- | ---------------------- | ------------------------------------------------------------ |
| attribute_name_index   | u2             | 1                      | 属性名称，指向 CONSTANT_Utf8_info 的常量索引，固定是 “Code”  |
| attribute_length       | u4             | 1                      | 属性内容信息占有的字节码长度                                 |
| max_stack              | u2             | 1                      | 操作数栈深度的最大值。在方法执行的任何时候操作数栈都不会超过这个深度。虚拟机运行的时候需要根据这个值来分配栈帧中的操作栈深度 |
| max_locals             | u2             | 1                      | 局部变量表所需的存储空间，单位是 Solt。Slot 是虚拟机为局部变量分配内存所使用的最小单位。对于 byte、char、float、int、short、boolean 和 retruanAddress 等长度不超过 32 位的数据类型，用 1 个 Slot 存放。对于 double 和 long 这两种 64 位的数据类型则需要 2 个Slot。 |
| code_length            | u4             | 1                      | 字节码指令长度                                               |
| code                   | u1             | code_length            | 字节码指令                                                   |
| exception_table_length | u2             | 1                      | 显示异常处理表长度，用于表示方法体内的异常                   |
| exception_table        | exception_info | exception_table_length | 显示异常处理表信息                                           |
| attributes_count       | u2             | 1                      | 属性表中属性数量                                             |
| attributes             | attribute_info | attributes_count       | 属性表中属性信息                                             |

其中异常表（exception_table）的数据结构如下，其表示在 start_pc 到 end_pc (不包括 end_pc 本身)之间出现了类型为 catch_type 或者子类的异常，则跳转到 handle_pc 处继续执行。当 catch_type 值是 0 时，代表任意异常情况都需要转向 handler_pc 处进行处理。

| 名称       | 类型 | 数量 | 含义                                                         |
| ---------- | ---- | ---- | ------------------------------------------------------------ |
| start_pc   | u2   | 1    | 可能发生异常的开始的地方                                     |
| end_pc     | u2   | 1    | 被包裹的异常块的结束位置                                     |
| handler_pc | u2   | 1    | 处理异常的入口处                                             |
| catch_type | u2   | 1    | 指向一个 CONSTANT_Class_info 型常量的索引，表示异常的类型，如果为 0，表示接受任何类型 |

## Exceptions 属性

Exceptions 属性的作用是列举出方法中可能抛出的受检异常（Checked Exceptions），也就是方法描述是在 throws 关键字后面列举的异常。不同于 Code 属性中的异常，Code 属性中的异常表示的是方法体内的 try-catch块。Exceptions 属性的结构如下所示

| 名称                  | 类型 | 数量                 | 含义                                                         |
| --------------------- | ---- | -------------------- | ------------------------------------------------------------ |
| attribute_name_index  | u2   | 1                    | 属性名称，指向 CONSTANT_Utf8_info 的常量索引，固定是 “Exceptions” |
| attribute_length      | u4   | 1                    | 属性内容信息占有的字节码长度                                 |
| number_of_exceptions  | u2   | 1                    | 方法抛出的受检异常的数量                                     |
| exception_index_table | u2   | number_of_exceptions | 一个指向常量池中的 CONSTANT_Class_info 型常量的索引，表示受检异常的类型 |

## LineNumberTable 属性

LineNumberTable 属性用于描述 java 源码行号与字节码行号（字节码的偏移量）之间的对应关系。这并不是运行时的必需属性，但是默认为生成到 class 文件中，可以在 javac 中分别使用 -g:none 选项来取消生成这项信息，也可以使用 -g:lines 选项来生成这项信息。其结构如下所示

| 名称                     | 类型             | 数量                     | 含义                                                         |
| ------------------------ | ---------------- | ------------------------ | ------------------------------------------------------------ |
| attribute_name_index     | u2               | 1                        | 属性名称，指向 CONSTANT_Utf8_info 的常量索引，固定是 “LineNumberTable” |
| attribute_length         | u4               | 1                        | 属性内容信息占有的字节码长度                                 |
| line_number_table_length | u2               | 1                        | line_number_table 项的数量                                   |
| line_number_table        | line_number_info | line_number_table_length | 字节码与 java 源码行号对应关系                               |

其中 line_number_table 的数据结构如下所示

| 名称        | 类型 | 数量 | 含义           |
| ----------- | ---- | ---- | -------------- |
| start_pc    | u2   | 1    | 字节码的偏移量 |
| line_number | u2   | 1    | java 源码行号  |

## LocalVariableTable 属性

LocalVariableTable 属性用于描述栈帧中局部变量表中变量与 java 源码中定义的变量之间的关系，这不是运行时必需的属性，但是默认会生成到 class 文件中。可以在 javac 命令中使用 -g:none 选项来取消生成这项信息，也可以使用 -g:vars 选项要求生成这项信息。其结构如下所示

| 名称                             | 类型                | 数量                             | 含义                                                         |
| -------------------------------- | ------------------- | -------------------------------- | ------------------------------------------------------------ |
| attribute_name_index             | u2                  | 1                                | 属性名称，指向 CONSTANT_Utf8_info 的常量索引，固定是 “LocalVariableTable” |
| attribute_length                 | u4                  | 1                                | 属性内容信息占有的字节码长度                                 |
| local_variable_table-<br>_length | u2                  | 1                                | 局部变量表数量                                               |
| local_variable_table             | local_variable_info | local_variable_table-<br>_length | 局部变量表信息                                               |

其中 local_variable_table 的结构如下所示

| 名称             | 类型 | 数量 | 含义                                                         |
| ---------------- | ---- | ---- | ------------------------------------------------------------ |
| start_pc         | u2   | 1    | 局部变量的生命周期开始的字节码偏移量                         |
| length           | u2   | 1    | 局部变量的生命周期的作用范围的长度                           |
| name_index       | u2   | 1    | 指向常量池中 CONSTANT_Utf8_info 型常量的索引，表示局部变量的名称 |
| descriptor_index | u2   | 1    | 指向常量池中 CONSTANT_Utf8_info 型常量的索引，表示局部变量的描述符 |
| index            | u2   | 1    | 表示局部变量在栈帧局部变量表中 Slot 的位置。当这个变量的数据类型是 64 位类型（double 和 long）时，它占用的 Slot 为 index 和 index + 1 两个 |

## SourceFile 属性

SourceFile 属性用于记录生成这个 class 文件的源码文件名称，这是一个可选的属性，默认情况下会生成。可以在javac 命令中使用 -g:none 选项来取消生成这项信息，也可以使用 -g:source 选项明确要求生成这项信息。其结构如下图所示

| 名称                 | 类型 | 数量 | 含义                                                         |
| -------------------- | ---- | ---- | ------------------------------------------------------------ |
| attribute_name_index | u2   | 1    | 属性名称，指向 CONSTANT_Utf8_info 的常量索引，固定是 “SourceFile” |
| attribute_length     | u4   | 1    | 属性内容信息占有的字节码长度                                 |
| source_index         | u2   | 1    | 指向常量池中 CONSTANT_Utf8_info 型常量的索引，表示源码文件的文件名 |

上述例子中，第 290 ~ 299 个字节表示 SourceFile 属性信息，其具体含义如下

```\
00 01               属性数量，值是1
00 0d               属性名称索引，attribute_name_index，对应到第 13 个常量，值是SourceFile
00 00 00 02         属性字节长度，attribute_length，值是2
00 0e               SourceFile属性中源码文件名称索引，sourcefile_index，指向第 14 个常量，值是TestJvmClassStructure.java
```

## ConstantValue 属性

ConstantValue 属性的作用是通知虚拟机自动为静态变量赋值，只有被 static 关键字修改时的变量（类变量）才可以使用这个属性。其结构如下所示

| 名称                 | 类型 | 数量 | 含义                                                         |
| -------------------- | ---- | ---- | ------------------------------------------------------------ |
| attribute_name_index | u2   | 1    | 属性名称，指向 CONSTANT_Utf8_info 的常量索引，固定是 “ConstantValue” |
| attribute_length     | u4   | 1    | 属性内容信息占有的字节码长度                                 |
| constantvalue_index  | u2   | 1    | 指向常量池中字面常量的引用。根据字段类型的不同，字面量可以是 CONSTANT_Long_info、CONSTANT_Float_info、CONSTANT_Double_info、CONSTANT_String_info 常量中的一种 |

## InnerClasses 属性

InnerClasses 属性用于记录内部类与宿主类之间的关联，其结构如下所示

| 名称                 | 类型               | 数量              | 含义                                                         |
| -------------------- | ------------------ | ----------------- | ------------------------------------------------------------ |
| attribute_name_index | u2                 | 1                 | 属性名称，指向 CONSTANT_Utf8_info 的常量索引，固定是 “InnerClasses” |
| attribute_length     | u4                 | 1                 | 属性内容信息占有的字节码长度                                 |
| number_of_classes    | u2                 | 1                 | 需要记录的内部类的数量                                       |
| inner_classes        | inner_classes_info | number_of_classes | 需要记录的内部类的信息                                       |

其中 inner_classes 的结构如下所示

| 名称                     | 类型 | 数量 | 含义                                                         |
| ------------------------ | ---- | ---- | ------------------------------------------------------------ |
| inner_class_info_index   | u2   | 1    | 指向常量池中 CONSTANT_Class_info 型常量的索引，表示内部类的符号引用 |
| outer_class_info_index   | u2   | 1    | 指向常量池中 CONSTANT_Class_info 型常量的索引，表示宿主类的符号引用 |
| inner_name_index         | u2   | 1    | 指向常量池中 CONSTANT_Utf8_info 型常量的索引，表示内部类的名称，如果是匿名内部类，则值是 0 |
| inner_class_access_flags | u2   | 1    | 内部类的访问标志                                             |

其中 inner_class_access_flags 文件访问标志类似，其定义的标志名称的值在该标志位为真时取对应的值，为假时取 0，最终将所有标志位进行与运算得到最后结果。其具体项及其含义如下所示

| 标志名称       | 标志值 | 含义                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| ACC_PUBLIC     | 0x0001 | 内部类是否为 public 类型，如果是，取值为 0x0001，否则为 0x0000 |
| ACC_PRIVATE    | 0x0002 | 内部类是否是 private 类型，如果是，取值为 0x0002，否则为 0x0000 |
| ACC_PROTECTED  | 0x0004 | 内部类是否是 protected 类型，如果是，取值为 0x0004，否则为 0x0000 |
| ACC_STATIC     | 0x0008 | 内部类是否是 static 类型，如果是，取值为 0x0008，否则为 0x0000 |
| ACC_FINAL      | 0x0010 | 内部类是否是 final 类型，如果是，取值为 0x0010，否则为 0x0000 |
| ACC_INTERFACE  | 0x0020 | 内部类是否是接口，如果是，取值为 0x0020，否则为 0x0000       |
| ACC_ABSTRACT   | 0x0400 | 内部类是否是 abstract 类型，如果是，取值为 0x0400，否则为 0x0000 |
| ACC_SYNTHETIC  | 0x1000 | 内部类是否由编译器自动产生，如果是，取值为 0x1000，否则为 0x0000 |
| ACC_ANNOTATION | 0x2000 | 内部类是否是一个注解，如果是，取值为 0x2000，否则为 0x0000   |
| ACC_ENUM       | 0x4000 | 内部类是否是一个枚举类型，如果是，取值为 0x4000，否则为 0x0000 |

## Deprecated 属性

Deprecated 属性用于表示某个类、字段或者方法是否已经被程序作者标注为不推荐使用，对应 java 代码中的 `@deprecated` 注解。这是一个布尔属性，只存在有和没有的区别，没有属性值的概念。也就是说，要么在 class 文件中 没有这个属性，要么存在这个属性，但是属性长度为 0。其结构如下

| 名称                 | 类型 | 数量 | 含义                                                         |
| -------------------- | ---- | ---- | ------------------------------------------------------------ |
| attribute_name_index | u2   | 1    | 属性名称，指向 CONSTANT_Utf8_info 的常量索引，固定是 “Deprecated” |
| attribute_length     | u4   | 1    | 值为固定值，0x00000000                                       |

## Synthetic 属性

Synthetic 属性代表此字段或者方法并不是由 java 源码直接产生的，而是由编译器自行添加的。需要注意的是，实例构造器 `<init>` 和类构造器 `<clinit>` 虽然也可以由编译器自动生成，但是此时值为假。与 Deprecated 属性一样，这也是一个布尔属性。其结构如下所示

| 名称                 | 类型 | 数量 | 含义                                                         |
| -------------------- | ---- | ---- | ------------------------------------------------------------ |
| attribute_name_index | u2   | 1    | 属性名称，指向 CONSTANT_Utf8_info 的常量索引，固定是 “Synthetic” |
| attribute_length     | u4   | 1    | 值为固定值，0x00000000                                       |

## Signature 属性

Signature 属性主要用于记录泛型签名信息，其结构如下所示

| 名称                 | 类型 | 数量 | 含义                                                         |
| -------------------- | ---- | ---- | ------------------------------------------------------------ |
| attribute_name_index | u2   | 1    | 属性名称，指向 CONSTANT_Utf8_info 的常量索引，固定是 “Signature” |
| attribute_length     | u4   | 1    | 属性内容信息占有的字节码长度                                 |
| signature_index      | u2   | 1    | 指向常量池中 CONSTANT_Utf8_info 型常量的索引，表示类签名、方法类型签名或字段类型签名 |

 ## BootstrapMethods 属性

BootstrapMethods 属性用于保存 invokedynamic 指令引用的引导方法限定符。如果某个类文件结构的常量池中曾经出现过 CONSTANT_InvokeDynamic_info 类型的常量，那么这个类文件的属性表中必须存在一个明确的 BootstrapMethods 属性，另外，即使 CONSTANT_InvokeDynamic_info 类型的常量在常量池中出现过多次，类文件的属性表中最多也只有一个 BootstrapMethods 属性。其结构如下所示

| 名称                  | 类型             | 数量                  | 含义                                                         |
| --------------------- | ---------------- | --------------------- | ------------------------------------------------------------ |
| attribute_name_index  | u2               | 1                     | 属性名称，指向 CONSTANT_Utf8_info 的常量索引，固定是 “BootstrapMethods” |
| attribute_length      | u4               | 1                     | 属性内容信息占有的字节码长度                                 |
| num_bootstrap_methods | u2               | 1                     | 引导方法限定符的数量                                         |
| bootstrap_methods     | bootstrap_method | num_bootstrap_methods | 引导方法信息                                                 |

其中 bootstrap_methods 结构如下

| 名称                         | 类型 | 数量                         | 含义                                                         |
| ---------------------------- | ---- | ---------------------------- | ------------------------------------------------------------ |
| bootstrap_method_ref         | u2   | 1                            | 指向常量池中 CONSTANT_MethodHandle_info 型常量的索引         |
| num_bootstrap_arg-<br>uments | u2   | 1                            | bootstrap_arguments[] 数组成员的数量                         |
| bootstrap_arguments          | u2   | num_bootstrap_arg-<br>uments | 指向常量池中字面常量的引用。值必须是 CONSTANT_String_info、CONSTANT_Class_info、CONSTANT_Integer_info、 CONSTANT_Long_info、CONSTANT_Float_info、CONSTANT_Double_info、CONSTANT_MethodHandle_info、CONSTANT_MethodType_info 常量中的一种 |

# 附录

## 例子完整字节码解析

```
ca fe ba be               格式，固定值
00 00 00 34               JDK版本号
00 13                     常量池中的数量，值是19，说明有18个常量项
0a                        CONSTANT_Methodref_info常量项，tag
00 04                     指向声明方法的类描述符CONSTANT_Class_info的索引项, index, 值是4
00 0f                     指向名称及类型描述符CONSTANT_NameAndType_info的索引项, index, 值是15
09                        CONSTANT_Fieldref_info常量项，tag
00 03                     指向声明字段的类或接口描述符CONSTANT_Class_info的索引项, index, 值是3
00 10                     指向字段描述符CONSTANT_NameAndType_info的索引项, index, 值是16
07                        CONSTANT_Class_info常量项，tag
00 11                     指向全限定名称常量项的索引，index, 值是17
07                        CONSTANT_Class_info常量项，tag
00 12                     指向全限定名称常量项的索引，index, 值是18，
01                        CONSTANT_Utf8_info常量项，tag
00 01                     UTF-8编码的字符串占用的字节数，length, 值是1
6d                        长度为 1 的UTF-8编码字符串，bytes, 值是m
01                        CONSTANT_Utf8_info常量项，tag
00 01                     UTF-8编码的字符串占用的字节数，length, 值是1
49                        长度为 1 的UTF-8编码字符串，bytes, 值是I
01                        CONSTANT_Utf8_info常量项，tag
00 06                     UTF-8编码的字符串占用的字节数，length, 值是6
3c 69 6e 69 74 3e         长度为 6 的UTF-8编码字符串，bytes, 值是<init>
01                        CONSTANT_Utf8_info常量项，tag
00 03                     UTF-8编码的字符串占用的字节数，length, 值是3
28 29 56                  长度为 3 的UTF-8编码字符串，bytes, 值是()V
01                        CONSTANT_Utf8_info常量项，tag
00 04                     UTF-8编码的字符串占用的字节数，length, 值是4
43 6f 64 65               长度为 4 的UTF-8编码字符串，bytes, 值是Code
01                        CONSTANT_Utf8_info常量项，tag
00 0f                     UTF-8编码的字符串占用的字节数，length, 值是15
4c 69 6e 65 4e 75 6d 62 65 72 54 61 62 6c 65    长度为 15 的UTF-8编码字符串，bytes, 值是LineNumberTable
01                                              CONSTANT_Utf8_info常量项，tag
00 03                                           UTF-8编码的字符串占用的字节数，length, 值是3
69 6e 63                                        长度为 3 的UTF-8编码字符串，bytes, 值是inc
01                                              CONSTANT_Utf8_info常量项，tag
00 03                                           UTF-8编码的字符串占用的字节数，length, 值是3
28 29 49                                        长度为 3 的UTF-8编码字符串，bytes, 值是()I
01                                              CONSTANT_Utf8_info常量项，tag
00 0a                                           UTF-8编码的字符串占用的字节数，length, 值是10
53 6f 75 72 63 65 46 69 6c 65                   长度为 3 的UTF-8编码字符串，bytes, 值是SourceFile
01                                              CONSTANT_Utf8_info常量项，tag
00 1a                                           UTF-8编码的字符串占用的字节数，length, 值是26
54 65 73 74 4a 76 6d 43 6c 61 73 73 53 74 72 75 63 74 75 72 65 2e 6a 61 76 61       长度为 26 的UTF-8编码字符串，bytes, 值是TestJvmClassStructure.java
0c                        CONSTANT_NameAndType_info常量项, tag 
00 07                     指向该字符安或方法名称常量项的索引项, index, 值是7
00 08                     指向该字符安或方法描述符常量项的索引项, index, 值是8
0c                        CONSTANT_NameAndType_info常量项, tag 
00 05                     指向该字符安或方法名称常量项的索引项, index, 值是5
00 06                     指向该字符安或方法描述符常量项的索引项, index, 值是6
01                        CONSTANT_Utf8_info常量项，tag
00 15                     UTF-8编码的字符串占用的字节数，length, 值是21
54 65 73 74 4a 76 6d 43 6c 61 73 73 53 74 72 75 63 74 75 72 65      长度为 21 的UTF-8编码字符串，bytes, 值是TestJvmClassStructure
01                        CONSTANT_NameAndType_info常量项, tag
00 10                     UTF-8编码的字符串占用的字节数，length, 值是16
6a 61 76 61 2f 6c 61 6e   长度为 16 的UTF-8编码字符串，bytes, 值是java/lang/Object

                          // 常量池完毕

00 21                     访问标志，ACC_PUBLIC、ACC_SUPER为真，ACC_FINAL、ACC_INFERFACE、ACC_ABSTRACT、ACC_SYNTHETIC、ACC_ANNOTATION、ACC_ENUM为假，故值为 0x0001 | 0x0021 = 0x0021
00 03                     本类索引，this_class，对应到第 3 个常量项，得知指向第 17 个常量项，值是TestJvmClassStructure
00 04                     父类索引，super_calss，对应到第 4 个常量项，得知指向第 18 个常量项，值是java/lang/Object
00 00                     接口索引集合大小，值是0，表示没有实现接口
00 01                     字段表中字段的数量，值是1
00 02                     字段访问标志，access_flags，0x0002表示ACC_PRIVATE为真，其他标志符为假
00 05                     字段的简单名称，name_index，对应到第 5 个常量，值是m
00 06                     字段的描述符，descriptor_index，对应到第 6 个常量，值是I
00 00                     字段的属性表中属性数量，值是0

                          // 字段表完毕

00 02                     方法表中方法的数量，值是2
00 01                     方法访问标志，access_flags，0x0001表示ACC_PUBLIC为真，其他标志符为假
00 07                     方法的简单名称，name_index，对应到第 7 个常量，值是<init>
00 08                     方法的描述符，descriptor_index，对应到第 8 个常量，值是()V
00 01                     方法的属性表中属性数量，值是1
00 09                     属性名称索引，attribute_name_index，对应到第 9 个常量，值是Code
00 00 00 1d               属性字节长度，attribute_length，值是29
00 01                     Code属性中操作数栈深度的最大值，max_stack，值是1
00 01                     Code属性中局部变量表所需的存储空间，max_locals，单位是Slot，值是1
00 00 00 05               字节码指令字节长度，值是5
2a                        虚拟机指令 aload_0，将第一个引用类型本地变量推送值栈顶
b7                        虚拟机指令 invokespecial，实例初始化方法，构造函数调用，后续有个 u2 类型参数说明调用的是哪个方法
00 01                     invokespecial 参数，指向常量池中第 1 个常量，得知值是java/lang/Object."<init>":()V
b1                        虚拟机指令 return，从当前方法返回 void
00 00                     Code属性中异常表数量，exception_table_length，值是0
00 01                     Code属性中属性数量，attributes_count，值是1
00 0a                     属性名称索引，attribute_name_index，对应到第 10 个常量，值是LineNumberTable
00 00 00 06               属性字节长度，attribute_length，值是6
00 01                     LineNumberTable属性中line_number_table数量，line_number_table_length，值是1
00 00 00 01               LineNumberTable属性中line_number_info表，包括一个u2的start_pc和一个u2的line_number，分别表示字节码相对于方法体开始的偏移量（值是0），后者表示java源码的行号（值是1）

                          // init方法完毕

00 01                     方法访问标志，access_flags，0x0001表示ACC_PUBLIC为真，其他标志符为假
00 0b                     方法的简单名称，name_index，对应到第 11 个常量，值是inc
00 0c                     方法的描述符，descriptor_index，对应到第 12 个常量，值是()I
00 01                     方法的属性表中属性数量，值是1
00 09                     属性名称索引，attribute_name_index，对应到第 9 个常量，值是Code
00 00 00 1f               属性字节长度，attribute_length，值是31
00 02                     Code属性中操作数栈深度的最大值，max_stack，值是2
00 01                     Code属性中局部变量表所需的存储空间，max_locals，单位是Slot，值是1
00 00 00 07               字节码指令字节长度，值是7
2a                        虚拟机指令 aload_0，将第一个引用类型本地变量推送值栈顶
b4                        虚拟机指令 getfield，获取指定类的实例域，并将其值压入栈顶，后续有个u2类型参数说明压入栈顶的是哪个变量
00 02                     getfield 参数，指向常量池中第 2 个常量，得知值是TestJvmClassStructure.m:I
04                        虚拟机指令 iconst_1，将int型 1 推送至栈顶
60                        虚拟机指令 iadd，将栈顶两个 int 型相加，并将结果压入栈顶
ac                        虚拟机指令 ireturn，从当前方法返回 int
00 00                     Code属性中异常表数量，exception_table_length，值是0
00 01                     Code属性中属性数量，attributes_count，值是1
00 0a                     属性名称索引，attribute_name_index，对应到第 10 个常量，值是LineNumberTable
00 00 00 06               属性字节长度，attribute_length，值是6
00 01                     LineNumberTable属性中line_number_table数量，line_number_table_length，值是1
00 00 00 06               LineNumberTable属性中line_number_info表，包括一个u2的start_pc和一个u2的line_number，分别表示字节码相对于方法体开始的偏移量（值是0），后者表示java源码的行号（值是6）

                          // 方法表完毕

00 01                     属性数量，值是1
00 0d                     属性名称索引，attribute_name_index，对应到第 13 个常量，值是SourceFile
00 00 00 02               属性字节长度，attribute_length，值是2
00 0e                     SourceFile属性中源码文件名称索引，sourcefile_index，指向第 14 个常量，值是TestJvmClassStructure.java
```

## 例子完整 javap 反编译输出

```java
Classfile /C:/Users/zhchun/Desktop/TestJvmClassStructure.class
  Last modified 2018-7-29; size 299 bytes
  MD5 checksum f683f6070c8a0820e2fdd9adf16d6c1d
  Compiled from "TestJvmClassStructure.java"
public class TestJvmClassStructure
  SourceFile: "TestJvmClassStructure.java"
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#15         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#16         //  TestJvmClassStructure.m:I
   #3 = Class              #17            //  TestJvmClassStructure
   #4 = Class              #18            //  java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               inc
  #12 = Utf8               ()I
  #13 = Utf8               SourceFile
  #14 = Utf8               TestJvmClassStructure.java
  #15 = NameAndType        #7:#8          //  "<init>":()V
  #16 = NameAndType        #5:#6          //  m:I
  #17 = Utf8               TestJvmClassStructure
  #18 = Utf8               java/lang/Object
{
  public TestJvmClassStructure();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0       
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return        
      LineNumberTable:
        line 1: 0

  public int inc();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0       
         1: getfield      #2                  // Field m:I
         4: iconst_1      
         5: iadd          
         6: ireturn       
      LineNumberTable:
        line 6: 0
}
```

# 参考资料

[1] 周志明. 深入理解Java虚拟机：JVM高级特性与最佳实践[M]. 北京：机械工业出版社，2013