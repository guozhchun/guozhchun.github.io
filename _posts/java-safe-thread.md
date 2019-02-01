---
title: java 线程安全
date: 2018-11-17 15:28:01
tags: java
categories: java
---

# 定义

> 当多个线程访问某个对象时，不管运行是环境采用何种调度方式或者这些线程将如何交替执行，并且爱主调代码中不需要任何额外的同步或协同，这个对象都能表现出正确的行为，那么就称这个对象是线程安全的。
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Java 并发编程实战》

这个定义是严谨的，但是看起来有点绕，稍微不严谨一点的说法就是：当一段代码能够在多线程环境下正常运行得到正确结果，那这段代码就是线程安全的。

<!-- more -->

# java 线程安全分类

一般情况下，说道线程安全，只有两种结果：线程安全和线程不安全。但是从安全程度来判断，可以将其划分为以下 5 种类型：不可变、绝对线程安全、相对线程安全、线程兼容、线程对立。

## 不可变

当一个对象是不可变对象时，其一定是线程安全的，而且不需要提供方或调用方进行任何的同步操作。在 java 中，对于基本类型而言，被 final 修饰就是不可变对象，对于对象而言，其内部的成员都是不可变对象（基本类型是 final，引用类型递归判断不可变性）时，该对象是不可变的。例如，常用的 String 类就是一个不可变对象，对这个对象任何的操作都不会改变原先的值。

## 绝对线程安全

绝对线程安全指不论任何情况下都不需要调用方进行同步操作。这个有点苛刻，通常需要付出很大的甚至不切实际的代价。一般情况下，通常说 Vector 是线程安全的，因为这个类中的方法都是被 synchronized 修饰的同步方法。但是这个类并没有达到“绝对线程安全”的程度。因为以下例子（例子参考自：《深入理解Java虚拟机》）中显示这个类在某种特定的情况下是需要调用方进行同步的。

```java
public class TestVector
{
    private static Vector<Integer> vector = new Vector<>();
    
    public static void main(String[] args)
    {
        while(true)
        {
            for (int i = 0; i < 10; i++)
            {
                vector.add(i);
            }
            
            new Thread(new Runnable() {
                @Override
                public void run()
                {
                    for (int i = 0; i < vector.size(); i++)
                    {
                        vector.remove(i);
                    }
                }
            }).start();
        }
    }
}
```

运行上述程序会出现类似以下异常

```
Exception in thread "Thread-382" java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 13
	at java.util.Vector.remove(Unknown Source)
	at test.TestVector$1.run(TestVector.java:26)
	at java.lang.Thread.run(Unknown Source)
```

虽然 add 和 remove 都是同步方法，但是在 for 循环中，仍然可能存在这样一种情况：一个线程获取了变量 i ，准备执行 remove 操作，但是正好此时切换到另一个线程中执行，在另一个线程中执行了一些删除操作，刚好删除了 n 个数据导致 i 位置在 vector 范围之外，此时切回原先的线程执行 remove 操作就会出现 ArrayIndexOutOfBoundsException 异常。

要想解决上述问题，需要在 for 循环外部加入同步块，如下所示：

```java
public class TestVector
{
    private static Vector<Integer> vector = new Vector<>();
    
    public static void main(String[] args)
    {
        while(true)
        {
            for (int i = 0; i < 10; i++)
            {
                vector.add(i);
            }
            
            new Thread(new Runnable() {
                @Override
                public void run()
                {
                    synchronized (vector)
                    {
                        for (int i = 0; i < vector.size(); i++)
                        {
                            vector.remove(i);
                        }
                    }
                    
                }
            }).start();
        }
    }
}

```

## 相对线程安全

相对线程安全就是平时所说的线程安全。它保证的是任何情况（包括多线程）下，调用对象的**同一个方法**，不需要额外的同步操作，也能获得正确的结果。但是在多线程的环境下，调用对象的不同方法，则需要额外的同步操作才能获得正确的结果。例如上述例子中的 vector 对象，当多个线程分别调用 add 方法往 vector 中加入元素时，其不需要任何的同步操作也能得到正确的结果，而对于 ArrayList 对象而言，多线程调用 add 方法，如果不进行同步操作，则得到的结果很可能少于增加的元素数量，因为 ArrayList 的 add 方法既不是同步的，也不是原子的。

## 线程兼容

线程兼容指对象本身不是线程安全的，但是可以通过在调用端正确地使用同步手段来保证对象在并发环境中可以安全地使用。

## 线程对立

线程对立是指无论调用端是否采取了同步措施，都无法在多线程环境中并发使用的代码。由于 java 语言天使具备多线程特性，线程对立这种排斥多线程的代码是很少出现。但是仍然存在，常见的线程对立的操作有 Thread 类的 suspend() 和 resume() 方法以及 System.setIn()、System.setOut() 等。

# 线程安全的实现方法

上文已经说过，当一个对象是不可变对象时，其一定是线程安全的。那当一个对象是可变的，或者是一段代码块时，应该如何保证其线程安全呢。主要有以下三种方法：互斥同步（阻塞同步）、非阻塞同步、不共享数据。

## 互斥同步

互斥同步（阻塞同步）是常见的一种同步手段，主要是通过加锁的方式来保证资源共享的同步和正确性的。常见的加锁方法有两种，一种是 `synchronized`，另一种是 `ReentrantLock`。

### synchronized

synchronized 关键字可以用于同步方法，也可以用于同步代码块，当用于方法时，其默认会锁定方法所在的类实例或 Class 对象（根据方法是实例方法还是类方法来决定）。当用于同步代码块时，其需要一个参数用于充当锁，对这个对象进行加锁和解锁。

synchronized 同步块对同一个线程来说是可重入的，不会出现自己把自己锁死的问题，对于不同的线程，其会在已进入同步块的线程执行完同步块的代码之前，阻塞其他线程。当线程退出同步代码块时，其他线程被唤醒并争夺获得锁进入同步代码块，因此，这是一个非公平的锁。

使用 synchronized 关键字进行同步，其加锁和解锁的过程都是由虚拟机进行的，并不需要程序手动控制。

### ReentrantLock

ReentrantLock 也是一个可重入的锁，不过 ReentrantLock 的加锁和解锁需要程序自己控制。另外，相比 synchronized ，其具有以下三个特点：等待可中断、可实现公平锁、锁可以绑定多个条件。

* 等待可中断：指当持有所的线程长期不释放所的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。
* 公平锁：值多个线程在等待同一个锁时，必须按照申请所的时间顺序来依次获得锁。非公平锁则不保证这一点，在锁被释放时，任何一个等待锁的线程都有机会获得锁。 synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是可以通过带布尔值的构造函数要求使用公平锁。
* 锁绑定多个条件：指一个 ReentrantLock 对象可以同时绑定多个 Condition 对象，而在 synchronized 中，一个同步块只能绑定一个锁条件，如果要实现两个条件的锁，需要用两个同步块来实现。而对于 ReentrantLock 则无需这样做，只需要多次调用 newConditon() 方法即可。

## 非阻塞同步

互斥同步悲观地认为如果不进行加锁执行代码就会得到不正确的结果，因此其是通过加锁的方式来实现的，这就需要线程的阻塞和唤醒，是一种比较大的性能开销。非阻塞同步则认为乐观地认为大部分情况下是不会有冲突的，因此其是先执行代码，然后再检测是否发生冲突，如果冲突则回退操作并重复尝试，这种方式不涉及加锁及解锁，也不涉及线程的阻塞及唤醒，在某些情况下性能相对较好，但是对于部分经常发生冲突的情况而言，由于其经常发生冲突导致更新失效需要重试，可能会出现性能较差的情况。

非阻塞同步是需要硬件指令集的支持的，因为其需要保证更新操作和冲突检测这两个步骤都具有原子性。这类指令主要有：测试并设置（Test-and-Set）、获取并增加（Fetch-and-Increment）、交换（Swap）、比较并交换（Compare-and-Swap，检测 CAS）、加载链接/条件存储（Load-Link/Store-Condition，简称 LL/SC）。

java 在 `sun.misc.Unsafe` 包中的 `compareAdnSwapInt()` 等方法中提供了 CAS 的支持大，但是由于 Unsafe 并不是给用户调用的类，因此，java 在 `java.util.concurrent` 包中提供了一些封装了支持 CAS 操作的 `compareAndSet()`、`getAndIncreent()` 等方法的原子类，如 `AtomicInteger` 类，其 `incrementAndGet()` 方法底层就是用 CAS 实现的。

## 不共享数据

当数据只存在于本线程中，其他线程无法获取时，那围绕着这些数据进行相关操作的代码都是线程安全的，不需要其他任何的同步操作。

# 参考资料

[1] Braan Goetz等，Java 并发编程实战[M]，北京：机械工业出版社，2012
[2] 周志明，深入理解Java虚拟机：JVM高级特性与最佳实践[M]，北京：机械工业出版社，2013

