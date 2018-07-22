---
title: jvm性能分析工具
date: 2018-07-07 15:14:50
tags: [java, jvm]
categories: java
---

# 概述

JDK中的bin目录除了javac和java这两个经常用于编译和运行java程序的工具外，还有许多用于监控虚拟机和故障处理的工具，包括命令行工具jps、jstat、jinfo、jmap、jhat、jstack和可视化工具jconsole、jvisualvm。本文主要介绍jps、jstat、jinfo、jmap、jhat、jstack这几个命令行工具。他们的主要作用如下

| 名称   | 主要作用                                                     |
| ------ | ------------------------------------------------------------ |
| jps    | JVM Process Status Tool，显示指定系统内所有的HotSpot虚拟机进程 |
| jstat  | JVM Statistics Monitoring Tool，用于收集HotSpot虚拟机各方面的运行数据 |
| jinfo  | Configuration Info for Java，显示虚拟机配置信息              |
| jmap   | Memory Map for Java，生成虚拟机的内存转储快照（heapdump文件） |
| jhat   | JVM Heap Dump Browser，用于分析heapdump文件，它会建立一个HTTP/HTML服务器，让用户可以在浏览器上查看分析结果 |
| jstack | Stack Trace for Java，显示虚拟机的线程快照                   |

<!-- more -->

# jps：虚拟机进程状况工具

jps命令主要是列出正在运行的虚拟机进程，并显示虚拟机执行主类（Main Class，main函数所在的类）名称以及这些进程的本地虚拟机唯一ID（Local Virtual Machine Identifier，LVMID）。对本地虚拟机来说，LVMID与操作系统的进程ID（Process Identifier，PID）是一致的。这是使用频率最高的一个命令行工具，因为其他的命令行工具需要依赖LVMID，而LVMID可以通过jps查询得到。

## 命令格式

jps的命令格式如下

```shell
jps [options] [hostid]
```

其中options选项列表如下

| 选项 | 作用                        |
| ---- | --------------------------- |
| -q   | 只输出LVMID，省略主类的名称 |
|-m|输出虚拟机进程启动时传递给主main()函数的参数|
|-l|输出主类的全名，如果进程执行的是jar包，输出jar路径|
|-v|输出虚拟机进程启动时JVM参数|

hostid是可选参数，不输入时表示查询的是本机的虚拟机进程，输入时需要输入RMI注册表中的主机名。

## 例子

如下程序用来测试jps命令工具，该程序很简单，只是接收一个参数，然后不断将该参数打印出来。本文后续的例子将基于此例子进行说明。

```java
public class TestJvmTools
{
    public static void main(String args[]) throws Exception
    {
        if (args.length == 1)
        {
            while (true)
            {
                System.out.println(args[0]);
                Thread.sleep(20);
            }
        }
        System.out.println("no parameter pass");
    }
}
```

使用命令`java -Xms20M -Xmx20M TestJvmTools oneParameter `执行该程序，让程序一直处于运行状态，然后分别执行`jps -l`、`jps -m`、`jps -v`、`jps -q`命令，查看输出结果，如下图所示

![jps命令输出结果](/images/jvm/jps.png)

# jstat：虚拟机统计信息监视工具

jstat是用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或远程虚拟机进程中的类装载、内存、垃圾回收、JIT编译等运行数据。

## 命令格式

jstat命令格式如下

```shell
jstat [option vmid [interval[s|ms][count]]]
```

其中option表示希望查询的虚拟机信息，主要分为三类：类装载、垃圾回收、运行编译状况，具体选项及作用如下表所示

| 选项   | 作用                                               |
| ------ | -------------------------------------------------- |
| -class | 监视类装载、卸载数量、总空间以及类装载所耗费的时间 |
|-gc|监视java堆状况，包括Eden区、两个Survivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息|
|-gccapacity|监视内容与 -gc基本相同，但输出主要关注java堆各个区域使用到的最大、最小空间|
|-gcutil|监视内容与 -gc基本相同，但输出主要关注已使用空间站总空间的百分比|
|-gccause|与 -gcutil功能一样，但是会额外输出导致上一次GC产生的原因|
|-gcnew|监视新生代GC状况|
|-gcnewcapacity|监视内容与 -gcnew基本相同，输出主要关注使用到的最大、最小空间|
|-gcold|监视老年代GC状况|
|-gcoldcapacity|监视内容与 -gcold基本相同，输出主要关注使用到的最大、最小空间|
|-compiler|输出JIT编译器编译过的方法、耗时等信息|
|-printcompilation|输出已经被JIT编译的方法|
命令行中的vmid参数，如果是本地虚拟机进程，则与LVMID一致，如果是远程虚拟机进程，则vmid的格式如下

```shell
[protocol:][//]lvmid[@hostname[:port]/servername]
```

命令行中的interval和count代表查询间隔和次数，如果省略这两个参数，则只查询一次。例如以下命令表示每隔250毫秒查询一次进程10928垃圾回收情况，一共查询20次

```shell
jstat -gc 10928 250 20
```

## 例子

### 查看类装载信息

执行`jstat -class 10928`命令查看类的装载信息，如下图所示

![jstat查看类装载命令输出结果](/images/jvm/jstatClass.png)

输出结果中的参数说明如下

| 参数                           | 说明            |
| ------------------------------ | --------------- |
| Loaded                         | 加载class的数量 |
| Bytes（从左往右数第一个Bytes） | 所占用空间大小  |
| Unloaded                       | 未加载数量      |
| Bytes（从左往右数第二个Bytes） | 未加载占用空间  |
| Times                          | 装载所耗的时间  |

### 查看编译信息

分别执行`jstat -compiler 10928`、`jstat -printcompilation 10928`命令查看进程的编译相关信息，如下图所示

![jstat查看编译相关信息命令输出结果](/images/jvm/jstatCompile.png)

-compiler命令输出结果中的参数说明如下

| 参数         | 说明         |
| ------------ | ------------ |
| Compiled     | 编译数量     |
| Failed       | 编译失败数量 |
| Invilid      | 不可用数量   |
| Time         | 编译时间     |
| FailedTypes  | 失败类型     |
| FailedMethod | 失败的方法   |

-printcompilation命令输出结果参数说明如下

| 参数     | 说明                     |
| -------- | ------------------------ |
| Compiled | 最近编译方法的数量       |
| Size     | 最近编译方法的字节码数量 |
| Type     | 最近编译方法的编译类型   |
| Method   | 方法名标识               |

### 查看垃圾回收信息

分别执行`jstat -gc 10928`、`jstat -gccapacity 10928`、`jstat -gcutil 10928`、`jstat -gccause 10928`、`jstat -gcnew 10928`、`jstat -gcnewcapacity 10928`、`jstat -gcold 10928`、`jstat -gcoldcapacity 10928`命令查看垃圾回收的相关信息，如下图所示

![jstat查看垃圾回收相关信息命令输出结果](/images/jvm/jstatGC.png)

输出结果中参数说明如下

| 参数  | 说明                                |
| ----- | ----------------------------------- |
| S0C   | Survivor0的大小                     |
| S1C   | Survivor1的大小                     |
| S0U   | Survivor0区已使用空间大小           |
| S1U   | Survivor1区已使用空间大小           |
| EC    | Eden区的大小                        |
| EU    | Eden区已使用的空间大小              |
| OC    | 老年代（Old）区的大小               |
| OU    | 老年代区已使用空间大小              |
| MC    | 方法区大小                          |
| MU    | 方法区已使用空间大小                |
| CCSC  | 压缩类空间大小                      |
| CCSU  | 压缩类空间已使用大小                |
| YGC   | 年轻代垃圾回收次数，YGC表示Young GC |
| YGCT  | 年轻代垃圾回收总耗时                |
| FGC   | 老年代垃圾回收次数，FGC表示Full GC  |
| FGCT  | 老年代来及回收总耗时                |
| GCT   | 垃圾回收总消耗时间                  |
| NGCMN | 新生代最小容量                      |
| NGCMX | 新生代最大容量                      |
| NGC   | 当前新生代容量                      |
| OGCMN | 老年代最小容量                      |
| OGCMX | 老年代最大容量                      |
| OGC   | 当前老年代容量                      |
| MCMN  | 方法区最小容量                      |
| MCMX  | 方法区最大容量                      |
| CCSMN | 压缩类空间最小容量                  |
| CCSMX | 压缩类空间最大容量                  |
| S0    | Survivor0区                         |
| S1    | Survivor1区                         |
| E     | Eden区                              |
| O     | 老年代                              |
| M     | 方法区                              |
| CCS   | 压缩类                              |
| TT    | 对象在新生代存活的次数              |
| MTT   | 对象在新生代存活的最大次数          |
| DSS   | 期望的一个Survivor区的大小          |
| S0CMX | Survivor0区最大容量                 |
| S1CMX | Survivor1区最大容量                 |
| ECMX  | Eden区最大容量                      |

# jinfo：java配置信息工具

jinfo的作用是实时地查看和调整迅疾各项参数。使用jps -v命令可以查看虚拟机启动时显示配置的参数，而jinfo命令不仅可以查看虚拟机启动时显示配置的参数，也能查看未被显示指定的参数的系统默认值，还能动态修改参数的值。

## 命令格式

jinfo的命令格式如下

```shell
jinfo [option] pid
```

其中pid表示进程号，option是一个可选参数，如果是查看信息，则使用 -flag param，如`jinfo -flag InitialHeapSize 10928` ，如果是修改信息，则使用 -flag +param或 -flag -param或 -flag param=value，如`jinfo -flag InitialHeapSize=20480 10928`或`jinfo -flag +UseParallelGC 10928`，

## 例子

下图所示是jinfo的查看命令

![jinfo命令输出结果](/images/jvm/jinfo.png)

# jmap：java内存映像工具

jmap命令用于生成堆转储快照（即heapdump或dump文件），同时也可以用于查询finalize执行队列、java堆和永久代的详细信息，如空间使用率、当前用的是哪种垃圾回集器等

## 命令格式

jmap的命令格式如下

```shell
jmap [option] vmid
```

其中option是可选参数，其选项和含义如下表所示

| 选项           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| -dump          | 生成java堆转储快照。格式为：-dump:[live,]format=b,file=<filename\>，其中live子参数说明是否只dump出存活对象 |
| -finalizerinfo | 显示在F-Queue中等待Finalizer线程执行finalize方法的对象       |
| -heap          | 显示java堆详细信息，如使用哪种回收器、参数配置、分代状况等   |
| -histo         | 显示堆中对象统计信息，包括类、实例数量、合计容量             |
| -clstats       | 以ClassLoader为统计入口                                      |
| -F             | 当虚拟机进程对 -dump   没有响应时，可使用这个选项强制生产dump快照 |

## 例子

下图是执行命令`jmap -dump:format=b,file=D:/testDump 10928`，`jmap -heap 10928`的结果

![jmap命令执行结果](/images/jvm/jmap.png)

# jhat：虚拟机堆转储快照分析工具

jhat主要用于分析jmap产生的堆转储快照，jhat内置了一个微型的HTTP/HTML服务器，在生成dump文件的分析结果后可以下浏览器中查看。然而，由于分析dump文件需要耗费很大的内存，加上jhat产生的界面比较简陋，远不及可视化工具jvisualvm，所以一般情况下也不会使用这个命令对dump文件进行分析。

## 命令格式

jhat的命令格式如下，其中<dumpFile\>指dump文件

```shell
jhat <dumpFile>
```

## 例子

下图是执行`jhat D:/testDump`命令的结果（D:/testDump是“jmap：java内存映像工具”章节中例子生产的dump文件）。当控制台中输出`Server is ready`后，在浏览器中输入`http://localhost:7000`即可查看相应的分析结果

![jhat命令控制台输出结果](/images/jvm/jhatConsole.png)

![jhat命令浏览器查看结果](/images/jvm/jhatBrowser.png)

# jstack：java堆栈跟踪工具

jstack命令用于生成虚拟机当前时刻的线程快照，主要是用于定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。当线程出现停顿的时候通过jstack查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做些什么事情，或者等待着什么资源。

## 命令格式

jstack的命令格式如下

```shell
jstack [option] vmid
```

其中option是一个可选参数，选项列表如下

| 选项 | 作用                                         |
| ---- | -------------------------------------------- |
| -F   | 当正常输出的请求不被响应时，强制输出线程堆栈 |
| -l   | 除堆栈外，显示关于锁的附加信息               |
| -m   | 如果调用到本地方法的话，可以显示C/C++的堆栈  |

## 例子

下图是执行`jstack -l 10928`命令的结果

![jstack命令输出结果](/images/jvm/jstack.png)

# 参考资料

[1] 周志明. 深入理解Java虚拟机：JVM高级特性与最佳实践[M]. 北京：机械工业出版社，2013
[2] 南湖公明. jstat命令使用[J/OL]. https://www.cnblogs.com/lizhonghua34/p/7307139.html , 2017-08-08 