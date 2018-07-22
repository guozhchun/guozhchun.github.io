---
title: jvm内存分配策略
date: 2018-07-05 21:19:10
tags: [java, jvm]
categories: java
---

# 概述

虚拟机将内存区域划分为新生代和老年代，在新生代中又划分成一个Eden和两个Survivor区域。对象的内存分配，主要是在新生代的Eden区上进行分配，部分比较大的对象会直接分配在老年代中。虽然对象的分配根据使用的垃圾回收器不同而有所区别，但是总体上而言，主要有以下几个分配原则：对象优先分配在Eden区、大对象直接分配在老年代、长期存活的对象将进入老年代、动态对象年龄判定、空间分配担保。

<!-- more -->

# 对象优先分配在Eden区

一般情况下，对象是直接分配在Eden区的，如果Eden区空间不够，将发生一次Minor GC进行垃圾回收。如下程序所示，程序配置的jvm参数中使用了`-XX:+UseSerialGC`启动了Serial + Serial Old的垃圾回收器组合进行垃圾回收，使用`-Xms20M`限制堆的最小值为20MB，使用`-Xmx20M`限制堆的最大值为20MB，从而限制了堆的大小为20MB，使用`-Xmn10M`限制了新生代大小为10MB，从而确认了老年代大小为10MB，使用`-XX:SurvivorRatio=8`限制了Eden区大小为8MB，两个Survivor区分别为1MB，使用`-XX:+PrintGCDetails`打印内存回收日志，并在进程退出时输出当前的内存各区域分配情况。

在程序中，先分配了3个2MB的空间，此时Eden区有足够内存进行分配，当要再分配4MB的时候，Eden区内存不够（Eden区除了放置allocation1、allocation2、allocation3三个对象外，还存在着一些元数据信息）4M，所以触发一次Minor GC。

Minor GC发生后，由于Survivor区只有1MB，不足以容纳6MB，所以allocation1、allocation2、allocation3三个对象通过分配担保提前进入老年代，从GC日志中可用看出，老年代中内存使用率有60%，此时Eden区已经有足够容量放置allocation4对象了，所以将allocation4对象分配到Eden区，加上其他一些数据，可用看到Eden区占比53%。

````java
// 程序使用的jvm参数：-Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:+UseSerialGC
public class TestEdenAllocation
{
    private static final int _1MB = 1024 * 1024;
    public static void main(String[] args)
    {
        byte[] allocation1 = new byte[2 * _1MB];
        byte[] allocation2 = new byte[2 * _1MB];
        byte[] allocation3 = new byte[2 * _1MB];
        
        // 出现Minor GC 
        byte[] allocation4 = new byte[4 * _1MB];
    }
}
````

GC日志如下

```java
[GC (Allocation Failure) [DefNew: 7315K->508K(9216K), 0.0074568 secs] 7315K->6652K(19456K), 0.0075242 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
Heap
 def new generation   total 9216K, used 4850K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  53% used [0x00000000fec00000, 0x00000000ff03d8f0, 0x00000000ff400000)
  from space 1024K,  49% used [0x00000000ff500000, 0x00000000ff57f018, 0x00000000ff600000)
  to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
 tenured generation   total 10240K, used 6144K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  60% used [0x00000000ff600000, 0x00000000ffc00030, 0x00000000ffc00200, 0x0000000100000000)
 Metaspace       used 2463K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 271K, capacity 386K, committed 512K, reserved 1048576K
```

# 大对象直接分配在老年代

大对象指需要大量连续内存空间的对象。虚拟机中提供了一个-XX:PretenureSizeThreshold参数来设置大对象分配在老年代的条件，当分配的对象大小超过这个值时，直接分配在老年代中。

如下程序所示，设置-XX:PretenureSizeThreshold的值为2097152（也就是2MB，这个值不能按照Xms那样按MB单位计算，只能按B单位计算），表示当分配的对象大小超过2M时，会直接分配在老年代中。从GC日志中也可以看出，在老年代中，内存使用率是40%（4MB），这就是allocation对象的内存。

```java
// 程序使用的jvm参数：-Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8  -XX:PretenureSizeThreshold=2097152 -XX:+UseSerialGC
public class TestPreTenureSizeThreshold
{
    private static final int _1MB = 1024 * 1024;
    public static void main(String[] args)
    {
        byte[] allocation = new byte[4 * _1MB];
    }
}
```

GC日志如下

```java
Heap
 def new generation   total 9216K, used 1499K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  18% used [0x00000000fec00000, 0x00000000fed76fd0, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 4096K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  40% used [0x00000000ff600000, 0x00000000ffa00010, 0x00000000ffa00200, 0x0000000100000000)
 Metaspace       used 2494K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 271K, capacity 386K, committed 512K, reserved 1048576K
```

需要注意的是PretenureSizeThreshold参数只对Serial和ParNew两款回收器有效，Parallel Scavenge回收器不认识这个参数。如下程序所示，使用Parallel Scavenge回收器，即使设置了这个参数，对象还是分配在Eden区中。从GC日志中可以看出，Eden区内存使用率为68%，而老年代中内存使用率为0

```java
// 程序使用的jvm参数：-Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8  -XX:PretenureSizeThreshold=2097152 -XX:+UseParallelGC
public class TestPreTenureSizeThreshold
{
    private static final int _1MB = 1024 * 1024;
    public static void main(String[] args)
    {
        byte[] allocation = new byte[4 * _1MB];
    }
}
```

GC日志如下

```java
Heap
 PSYoungGen      total 9216K, used 5595K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 8192K, 68% used [0x00000000ff600000,0x00000000ffb76ec0,0x00000000ffe00000)
  from space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
  to   space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
 ParOldGen       total 10240K, used 0K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  object space 10240K, 0% used [0x00000000fec00000,0x00000000fec00000,0x00000000ff600000)
 Metaspace       used 2494K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 271K, capacity 386K, committed 512K, reserved 1048576K
```

# 长期存活的对象将进入老年代

对象一般分配在新生代的Eden区中，但是存活对象也不会一直在新生代中，否则新生代很容易空间不足，因此当存活对象在新生代中到达一定时间后会移动到老年代中。为此，虚拟机给每个对象定义了一个对象年龄（Age）计算器：如果对象在Eden中并经过一次Minor GC后仍然存活，并且能够被survivor区容纳，则该对象将被移动到survivor区中，并且将对象年龄设置为1，后续每经过一次Minor GC后对象仍然存活，则对象年龄加一，当对象年龄到达一个阈值（默认是15）时，将对象移动到老年代中。虚拟机中提供了-XX:MaxTernuringThreshold参数来表示将新生代对象移动到老年代对象的阈值。

如下程序所示，设置-XX:MaxTernuringThreshold参数的值为1，表示新生代对象年龄超过1后将其移动到老年代中。从GC日志中可以看出，在程序退出时，surivor from区的空间使用率是0，而老年代中空间使用率是84%：allocation2（8MB） + allocation3（8MB） + allocation1（512KB） + 一些元数据信息（511KB）

```java
// 程序使用的jvm参数：-Xms40M -Xmx40M -Xmn20M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:+UseSerialGC -XX:MaxTenuringThreshold=1 -XX:+PrintTenuringDistribution
public class TestMaxTenuringThreshold
{
    private static final int _1MB = 1024 * 1024;
    public static void main(String[] args)
    {
        byte[] allocation1 = new byte[_1MB / 2];
        byte[] allocation2 = new byte[8 * _1MB];
        // 此处发生一次Minor GC
        byte[] allocation3 = new byte[8 * _1MB];
        // 此处再次发生Minor GC
        byte[] allocation4 = new byte[8 * _1MB];
    }
}
```

GC日志如下

```java
[GC (Allocation Failure) [DefNew
Desired survivor size 1048576 bytes, new threshold 1 (max 1)
- age   1:    1048416 bytes,    1048416 total
: 10670K->1023K(18432K), 0.0091166 secs] 10670K->9215K(38912K), 0.0091897 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [DefNew
Desired survivor size 1048576 bytes, new threshold 1 (max 1)
- age   1:        120 bytes,        120 total
: 9543K->0K(18432K), 0.0074626 secs] 17735K->17407K(38912K), 0.0075102 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
Heap
 def new generation   total 18432K, used 8356K [0x00000000fd800000, 0x00000000fec00000, 0x00000000fec00000)
  eden space 16384K,  51% used [0x00000000fd800000, 0x00000000fe0290e0, 0x00000000fe800000)
  from space 2048K,   0% used [0x00000000fe800000, 0x00000000fe800078, 0x00000000fea00000)
  to   space 2048K,   0% used [0x00000000fea00000, 0x00000000fea00000, 0x00000000fec00000)
 tenured generation   total 20480K, used 17407K [0x00000000fec00000, 0x0000000100000000, 0x0000000100000000)
   the space 20480K,  84% used [0x00000000fec00000, 0x00000000ffcffe80, 0x00000000ffd00000, 0x0000000100000000)
 Metaspace       used 2496K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 272K, capacity 386K, committed 512K, reserved 1048576K
```

当将-XX:MaxTernuringThreshold参数值设置成15时，GC日志如下。可以发现，程序退出时，survivior from区中空间使用率有49%：allocation1（512KB）+ 一些元数据（511KB），而老年代中使用率只有80%：allocation2（8MB） + allocation3（8MB）

```java
[GC (Allocation Failure) [DefNew
Desired survivor size 1048576 bytes, new threshold 15 (max 15)
- age   1:    1048416 bytes,    1048416 total
: 10670K->1023K(18432K), 0.0075451 secs] 10670K->9215K(38912K), 0.0076104 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [DefNew
Desired survivor size 1048576 bytes, new threshold 15 (max 15)
- age   1:        120 bytes,        120 total
- age   2:    1048160 bytes,    1048280 total
: 9543K->1023K(18432K), 0.0077775 secs] 17735K->17407K(38912K), 0.0078239 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 def new generation   total 18432K, used 9379K [0x00000000fd800000, 0x00000000fec00000, 0x00000000fec00000)
  eden space 16384K,  51% used [0x00000000fd800000, 0x00000000fe0290e0, 0x00000000fe800000)
  from space 2048K,  49% used [0x00000000fe800000, 0x00000000fe8ffed8, 0x00000000fea00000)
  to   space 2048K,   0% used [0x00000000fea00000, 0x00000000fea00000, 0x00000000fec00000)
 tenured generation   total 20480K, used 16384K [0x00000000fec00000, 0x0000000100000000, 0x0000000100000000)
   the space 20480K,  80% used [0x00000000fec00000, 0x00000000ffc00020, 0x00000000ffc00200, 0x0000000100000000)
 Metaspace       used 2496K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 272K, capacity 386K, committed 512K, reserved 1048576K
```

# 动态对象年龄判定

为了能更好地适应不同程序的内存情况，虚拟机并不是永远要求对象的年龄必须达到了MaxTernuringThreshold才能移动到老年代中。如果在survivor中相同年龄的对象大小总和大于survivor空间的一半，年龄大于或等于该年龄的对象将直接进入老年代中。

如下程序所示，设置MaxTernuringThreshold的值为15，在发生第一次Minor GC时，allocation1和allocation2对象及部分元数据对象被移动到survivor区，并且设置对象年龄为1，allocation3对象被移动到老年代中，当第二次Minor GC时，allocation4对象被移动到老年代中，survivor区中allocation1和allocation2及部分元数据对象总和大于1024KB，且他们年龄相同，这满足大于survivor区的一半空间的要求，因此他们被移动到老年代中。从GC日志也可以看出，survivor区空间使用率为0，而老年代中空间使用率为87%（allocation1、allocation2、allocation3、allocation4、一些元数据对象）

```java
// 程序使用的jvm参数：-Xms40M -Xmx40M -Xmn20M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:+UseSerialGC -XX:MaxTenuringThreshold=1 -XX:+PrintTenuringDistribution
public class TestMaxTenuringThreshold
{
    private static final int _1MB = 1024 * 1024;
    public static void main(String[] args)
    {
        byte[] allocation1 = new byte[_1MB / 2];
        byte[] allocation2 = new byte[_1MB / 2];
        byte[] allocation3 = new byte[8 * _1MB];
        // 此处再次发生Minor GC
        byte[] allocation4 = new byte[8 * _1MB];
        // 此处发生一次Minor GC
        byte[] allocation5 = new byte[8 * _1MB];
    }
}
```

GC日志如下

```java
[GC (Allocation Failure) [DefNew
Desired survivor size 1048576 bytes, new threshold 1 (max 15)
- age   1:    1572720 bytes,    1572720 total
: 11182K->1535K(18432K), 0.0066021 secs] 11182K->9727K(38912K), 0.0066686 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [DefNew
Desired survivor size 1048576 bytes, new threshold 15 (max 15)
- age   1:        120 bytes,        120 total
: 10055K->0K(18432K), 0.0079043 secs] 18247K->17919K(38912K), 0.0079458 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 def new generation   total 18432K, used 8356K [0x00000000fd800000, 0x00000000fec00000, 0x00000000fec00000)
  eden space 16384K,  51% used [0x00000000fd800000, 0x00000000fe0290e0, 0x00000000fe800000)
  from space 2048K,   0% used [0x00000000fe800000, 0x00000000fe800078, 0x00000000fea00000)
  to   space 2048K,   0% used [0x00000000fea00000, 0x00000000fea00000, 0x00000000fec00000)
 tenured generation   total 20480K, used 17919K [0x00000000fec00000, 0x0000000100000000, 0x0000000100000000)
   the space 20480K,  87% used [0x00000000fec00000, 0x00000000ffd7fe90, 0x00000000ffd80000, 0x0000000100000000)
 Metaspace       used 2497K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 272K, capacity 386K, committed 512K, reserved 1048576K
```

# 空间分配担保

新生代中使用复制算法进行垃圾回收，而新生代又分成一个大的Eden区和两个小的Survivor区，当发生Minor GC时，是将Eden和一个Survivor中的存活对象复制到另一个Survivor区中。这就存在一个风险：如果存活的对象过多，Survivor区容纳不下，进行Minor GC并不能将所有的存活对象复制到Survivor中。此时就需要老年代进行空间分配担保，将Survivor无法容纳的对象直接移动到老年代中。更进一步，如果老年代也容纳不下这些存活对象呢（这种情况称为担保失败（Handle Promotion Failure）），那就进行一次Full GC。

事实上，在JDK 6 Update 24版本之后，只要老年代中连续空间大于新生代对象总大小或者大于新生代对象历次晋升到老年代的平均大小，虚拟机就会进行一次Minor GC（担保失败则进行一次Full GC）,否则直接进行Full GC

# 其他

## Minor GC和Full GC的区别

Minor GC：指发生在新生代的垃圾回收，发生频率高，回收速度快

Full GC：也称Major GC，指发生在老年代的垃圾回收，发生频率低，回收速度慢。一般情况下，发生Majorl GC会伴随着至少一次Minor GC，但是部分回收器（如Parallel Scavenge回收器）的回收策略也可以直接跳过Minor GC而直接出发Major GC

## 对象流程图

一个对象在新生代和老年代之间的流动如下图所示

```flow
start=>start: Start
isObjectSizeTooBig=>condition: 对象大小超过阈值
                              PretenureSizeThreshold
newObjectInTenuredArea=>operation: 在老年代中分配对象
newObjectInEdenArea=>operation: 在Eden区中分配对象
happenMinorGC=>operation: 运行一段时间发生Minor GC
isSurvivorCanContainObject=>condition: 对象是否能在
Survivor区中
存放
moveToSurvivor=>operation: 将对象移动到Survivor区
happenMinorGC1=>operation: 运行一段时间发生Minor GC
isObjectAgeTooBig=>condition: 对象年龄大小大于阈值
						    MaxTernuringThreshold
isSameAgeObjectSizeTooBig=>condition: 相同年龄的对
								   象大小总和大于
								   Survivor区一
								   半大小
moveToTenuredArea=>operation: 将对象移动到老年代
addAge=>operation: 对象年龄加一
end=>end: End

start->isObjectSizeTooBig(yes, right)->newObjectInTenuredArea->end
isObjectSizeTooBig(no)->newObjectInEdenArea->happenMinorGC->isSurvivorCanContainObject(no)->moveToTenuredArea->end
isSurvivorCanContainObject(yes)->moveToSurvivor->happenMinorGC1->isObjectAgeTooBig(yes)->moveToTenuredArea->end
isObjectAgeTooBig(no, left)->isSameAgeObjectSizeTooBig(yes)->moveToTenuredArea
isSameAgeObjectSizeTooBig(no, left)->addAge(left)->happenMinorGC1
```

# 参考资料

[1] 周志明. 深入理解Java虚拟机：JVM高级特性与最佳实践[M]. 北京：机械工业出版社，2013

[2]Abraham Han. java虚拟机长期存活的对象将进入老年代实验结果与书上不同?[J/OL]. https://www.zhihu.com/question/36339308/answer/68189398 , 2018-03-21