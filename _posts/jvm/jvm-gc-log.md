---
title: GC 日志
date: 2018-07-11 22:15:42
tags: [java, jvm]
categories: jvm
---

# 前言

GC日志是分析性能问题的重要工具，通过查看GC日志可以对问题的解决方法有一个大致的方向。GC日志随着使用垃圾回收器的不同而有所差别，但是大致内容是相同的。本文主要以Serial垃圾回收器的GC日志进行说明。

<!-- more -->

# GC日志示例

![GC日志例子](/images/jvm/gcLog.png)

# GC日志说明

* 最开始的`[GC`表示这次GC的类型，如果是GC说明这次是Minor GC，如果是Full GC，说明这次是Full GC，Full GC发生需要stop-the-world
* 接着的`(Allocation Failure)`表示这次发生GC的原因，此处表示内存分配失败。
* 接着的`[DefNew: 7479K->511K(9216K), 0.0046090 secs]`表明这次发生GC的区域、发生GC前该区域使用的内存、发生GC后该区域使用的内存、该区域的总内存、发生GC在该区域的耗时。如此处表示发生GC的区域是新生代，新生代总内存9216KB，发生GC前使用了7479KB，发生GC后使用了511KB，GC在新生代的耗时是0.004609秒
* 接着的`7479K->6655K(19456K), 0.0046698 secs`表示发生GC前堆的已使用内存、发生GC后堆的已使用内存、发生GC的总耗时。例如此处表示发生GC前堆使用量是7479KB，发生GC后堆的使用量是6655KB，此次GC总共耗时0.0046698秒
* 接着的`[Times: user=0.00 sys=0.00, real=0.01 secs]`表示此次GC整个过程用户消耗的CPU时间、内核消耗的CPU时间和操作从开始到结束所经过的墙钟时间（Wall Clock Time，包括各种非运算的等待耗时，CPU时间不包括这些非运算的耗时）
* 接着的日志表示程序退出时各个内存区域的使用情况。`def new generation`表示新生代，`eden`表示Eden区域，`from`和`to`表示两Survivor区域，`tenured generation`表示老年代

# 参考资料

[1] 周志明. 深入理解Java虚拟机：JVM高级特性与最佳实践[M]. 北京：机械工业出版社，2013