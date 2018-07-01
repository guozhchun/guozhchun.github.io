---
title: jvm对象存活判定算法
date: 2018-07-01 11:59:14
tags: [java, jvm]
categories: java
---

# 概述

java虚拟机会自动进行内存管理和垃圾回收，在进行垃圾回收之前，需要先判定对象是否存活，只有对象“死去”（这里并不是指对象消失，而是指根据算法判定对象可以进行垃圾回收）才能被垃圾回收器进行回收。这里主要有两种对象存活判定算法：引用计数法和可达性分析算法。

<!-- more -->

# 引用计数法

引用计数法（Reference Counting）主要思想是通过给对象增加一个属性用于记录对象被引用的次数，当对象被其他对象引用时计数加一，当某个引用失效时计数减一，如果计数为0，则表示该对象不再被使用，可以在垃圾回收时进行回收。

引用计数法原理以及实现相对简单，一般情况下使用这个算法并没有太大的问题，但是其有一个很大的缺点，就是不能解决循环引用的问题：如果两个对象循环引用，但是并没有其他对象对这两个对象进行引用，从理论上来讲，这两个对象均是可以被垃圾回收的，但是如果是通过引用计数法的方法来判断是非可以回收的话，因为对这两个对象中的任何一个对象，都存在的引用，所以这两个对象都不会被垃圾回收。这在一定程度上来说会造成内存泄漏，所以目前市场上主流的java虚拟机里面没有选用引用计数法来管理内存。

# 可达性分析算法

可达性分析（Reachability Analysis）算法主要思想是通过一系列称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连（用图论的话来说，就是从GC Roots到这个对象不可达）时，则表示该对象不再被使用，可以在垃圾回收时进行回收。此方法可以避免引用计数法中的循环引用引起的内存泄漏。如下图所示，Object1、Object2在GC Root1的引用链上，Object3、Objcect4在GC Root2的引用链上，因此这四个对象都是存活的对象，不能进行垃圾回收。而Object5和Object6这两个对象虽然存储循环引用，但是并没有在任何GC Roots的引用链上，因此这两个对象是可以进行垃圾回收的对象。

![可达性算法图解](/images/jvm/reachabilityAnalysis.png)

在java语言中，可作为GC Roots的对象包括以下几种：

* 虚拟机栈（栈帧中的本地变量表）中引用的对象
* 方法区中类静态属性引用的对象
* 方法区中常量引用的对象
* 本地方法栈中JNI（即Native方法）引用的对象

# java中的引用

无论是通过引用计数法还是通过可达性分析算法来判断对象是否存活，都是根据对象是否被引用进行判断的。在java中有四种引用：强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Weak Reference）、虚引用（Phantom Reference）。这四种引用强度依次减弱。

* 强引用：强引用是代码中普遍存在的，只要强引用存在，垃圾回收器就不会回收强引用的对象。可以通过“Object obj = new Object()”之类的new语句建立强引用
* 软引用：软引用是用来描述一些有用但是非必须的对象。在垃圾回收时，如果内存充足，则不会回收软引用的对象，如果经过一次回收之后内存仍然不足，则会将软引用的对象进行回收。可以通过SoftReference类来实现软引用
* 弱引用：弱引用也是用来描述非必须对象的，但是其强度比软引用还弱。在垃圾回收时，无论内存是否充足，弱引用的对象都会被回收。可以通过WeakReference类来实现弱引用
* 虚引用也称为幽灵引用或者幻影引用，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构造影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被垃圾回收器回收时收到一个系统通知。可以通过PhantomReference类来实现虚引用

下面程序展示了软引用和弱引用的区别

对于软引用对象，垃圾回收时，由于内存充足，所以不会被回收。而对于弱引用对象，即使内存充足，在垃圾回收时也会被回收。而关于虚引用，则无论是在垃圾回收前还是垃圾回收后，均无法通过虚引用获取实例对象。

```java
class ReferenceType
{
    private String type;
    public ReferenceType(String type)
    {
        this.type = type;
    }
    
    @Override
    public String toString()
    {
        return "ReferenceType [type=" + type + "]";
    }
}

public class TestReference
{
    public static void main(String[] args)
    {
        ReferenceType strongReferece = new ReferenceType("strong reference");
        SoftReference<ReferenceType> softReference = new SoftReference<ReferenceType>(new ReferenceType("soft reference"));
        WeakReference<ReferenceType> weakReference = new WeakReference<ReferenceType>(new ReferenceType("weak reference"));
        PhantomReference<ReferenceType> phantomReference = new PhantomReference<ReferenceType>(new ReferenceType("phantom reference"), null);
        
        System.out.println("----before gc----");
        System.out.println("strong reference: " + strongReferece);
        System.out.println("soft reference: " + softReference.get());
        System.out.println("weak reference: " + weakReference.get());
        System.out.println("phantom reference: " + phantomReference.get());
        System.gc();
        System.out.println("----after gc----");
        System.out.println("strong reference: " + strongReferece);
        System.out.println("soft reference: " + softReference.get());
        System.out.println("weak reference: " + weakReference.get());
        System.out.println("phantom reference: " + phantomReference.get());
    }
}
```

程序输出结果如下

![引用例子输出结果](/images/jvm/referenceSampleOutput.png)

# 参考资料

[1] 周志明，深入理解Java虚拟机：JVM高级特性与最佳实践[M]，北京：机械工业出版社，2013