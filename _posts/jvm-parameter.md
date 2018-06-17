---
title: 部分java虚拟机参数
date: 2018-06-16 16:45:29
tags: [java, jvm]
categories: java
---

 # 概述

在部分程序中，需要进行性能调优，此时需要改变java虚拟机的各个内存区域的默认大小或改变垃圾回收器的策略，而这需要通过改变java虚拟机参数来完成。本文主要记录其中几个常用的参数。

<!-- more -->

# 内存参数列表
| 参数                    | 说明                                                   | 例子                                             |
| ----------------------- | ------------------------------------------------------ | ------------------------------------------------ |
| -Xms                    | 堆的最小值                                             | -Xms20m表示堆的最小值为20m                       |
| -Xmx                    | 堆的最大值                                             | -Xmx20m表示堆的最大值为20m                       |
| -Xss                    | 栈的容量大小                                           | Xss128k：栈的大小为128k                          |
| -XX:PermSize            | 方法区内存初始化值                                     | -XX:PermSize=10M表示方法区初始化内存大小为10M    |
| -XX:MaxPermSize         | 方法区内存最大值                                       | -XX:MaxPermSize=10M表示方法区内存最大值为10M     |
| -XX:MaxDirectMemorySize | 直接内存大小，不指定时，默认与java堆最大值(-Xmx值)一样 | -XX:MaxDirectMemorySize=10M表示直接内存大小为10M |

# 垃圾收集器参数列表

| 参数                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| Xmn                            | 新生代大小。如：-Xmn10m表示新生代大小为10m                   |
| UseSerialGC                    | 虚拟机运行在Client模式下的默认值，打开此开关后，使用Serial + Serial Old的收集器组合进行内存回收 |
| UseParNewGC                    | 打开此开关后，使用ParNew + Serial Old的收集器组合进行内存回收 |
| UseConcMarkSweepGC             | 打开此开关后，使用ParNew + CMS + Serial Old的收集器组合进行内容回收。Serial Old收集器将作为CMS收集器出现Concurrent Mode Failure失败后的后背收集器使用 |
| UseParallelGC                  | 虚拟机运行在Server模式下的默认值，打开此开关后，使用ParallelScavenge + Serial Old(PS MarkSweep)的收集器组合进行内存回收 |
| UseParallelOldGC               | 打开此开关后，使用Parallel Scavenge + Parallel Old的收集器组合进行内存回收 |
| SurvivorRatio                  | 新生代中Eden区域与Survivor区域的容量比值，默认为8，代表Eden:Survivor=8:1 |
| PretenureSizeThreshold         | 直接晋升到老年代的对象大小，设置这个参数后，大于这个参数的对象将直接在老年代分配 |
| MaxTenuringThreshold           | 晋升到老年代的对象年龄。每个对象在坚持过一次Minor GC之后，年龄就增加1，当超过这个参数值时就进入老年代 |
| UseAdaptiveSizePolicy          | 动态调整java堆中各个区域的大小以及进入老年代的年龄           |
| HandlePromotionFailure         | 是否运行分配担保失败，即老年代的剩余空间不足以应付新生代的整个Eden和Survivor区的所有对象都存活的极端情况 |
| ParallelGCThreads              | 设置并行GC时进行内存回收的线程数                             |
| GCTimeRatio                    | GC时间占总时间的比率，默认值为99，即允许1%的GC时间。仅在使用Parallel Scavenge收集器时生效 |
| MaxGCPauseMillis               | 设置GC的最大停顿时间。仅在使用Parallel Scavenge收集器时生效  |
| CMSInitiatingOccupancyFraction | 设置CMS收集器在老年代空间被使用多少后出发垃圾收集。默认值为68%。仅在使用CMS收集器时生效 |
| UseCMSCompactAtFullCollection  | 设置CMS收集器在完成垃圾收集后是否要进行一次内存碎片整理。仅在CMS收集器时生效 |
| CMSFullGCsBeforeCompaction     | 设置CMS收集器在进行若干次垃圾收集后再启动一次内存碎片整理。仅在使用CMS收集器时生效 |

# 辅助参数列表

| 参数                            | 说明                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| -XX:+HeapDumpOnOutOfMemoryError | 在虚拟机出现内存溢出异常时Dump出当前的内存堆栈存储快照       |
| -XX:+PrintGCDetails             | 在发生垃圾收集行为时打印内存日志，并且在进程退出的时候输出当前的内存各区域分配情况 |



# 参考资料

[1] 周志明，深入理解Java虚拟机：JVM高级特性与最佳实践[M]，北京：机械工业出版社，2013