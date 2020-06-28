---
title: ThreadLocal 介绍
date: 2019-12-08 18:42:03
tags: java
categories: java
---

# 概述

`ThreadLocal` 是 java 提供的一个方便对象在本线程内不同方法中传递和获取的类。用它定义的变量，仅在本线程中可见和维护，不受其他线程的影响，与其他线程相互隔离。

<!-- more -->

虽然在本线程不同方法中使用变量，可以通过在方法中传入参数解决，但是当涉及多个方法甚至多个类时，为每个方法增加同样的参数将是一场噩梦，此时 `ThreadLocal` 就能很好地解决这个问题。它可以在本线程内任何一个地方赋值，在任何一个地方获取值，并且不用作为函数参数传入。这看起来像静态成员变量，但是 `ThreadLocal` 变量相比静态成员变量的一个优势就是，`ThreadLocal` 是线程隔离的，其值不会受另一个线程的影响，也不用考虑加锁或值被其他线程篡改的问题，而这些问题都是静态成员变量无法做到的。因此当涉及一个对象需要在很多不同方法之间传递时，应该考虑使用 `ThreadLocal` 对象来简化代码。

# 使用

`ThreadLocal` 通过 `set` 方法可以给变量赋值，通过 `get` 方法获取变量的值。当然，也可以在定义变量时通过 `ThreadLocal.withInitial` 方法给变量赋初始值，或者定义一个继承 `ThreadLocal` 的类，然后重写 `initialValue` 方法。

示例代码如下

```java
public class TestThreadLocal
{
    private static ThreadLocal<StringBuilder> builder = ThreadLocal.withInitial(StringBuilder::new);

    public static void main(String[] args)
    {
        for (int i = 0; i < 5; i++)
        {
            new Thread(() -> {
                String threadName = Thread.currentThread().getName();
                for (int j = 0; j < 3; j++)
                {
                    append(j);
                    System.out.printf("%s append %d, now builder value is %s, ThreadLocal instance hashcode is %d, ThreadLocal instance mapping value hashcode is %d\n", threadName, j, builder.get().toString(), builder.hashCode(), builder.get().hashCode());
                }

                change();
                System.out.printf("%s set new stringbuilder, now builder value is %s, ThreadLocal instance hashcode is %d, ThreadLocal instance mapping value hashcode is %d\n", threadName, builder.get().toString(), builder.hashCode(), builder.get().hashCode());
            }, "thread-" + i).start();
        }
    }

    private static void append(int num) {
        builder.get().append(num);
    }

    private static void change() {
        StringBuilder newStringBuilder = new StringBuilder("HelloWorld");
        builder.set(newStringBuilder);
    }
}
```

在例子中，定义了一个 `builder` 的 `ThreadLocal` 对象，然后启动 5 个线程，分别对 `builder` 对象进行访问和修改操作，这两个操作放在两个不同的函数 `append`、`change` 中进行，两个函数访问 `builder` 对象也是直接获取，而不是放入函数的入参中传递进来。
代码输出如下

```
thread-0 append 0, now builder value is 0, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 566157654
thread-0 append 1, now builder value is 01, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 566157654
thread-4 append 0, now builder value is 0, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 654647086
thread-3 append 0, now builder value is 0, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 1803363945
thread-2 append 0, now builder value is 0, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 1535812498
thread-1 append 0, now builder value is 0, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 2075237830
thread-2 append 1, now builder value is 01, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 1535812498
thread-3 append 1, now builder value is 01, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 1803363945
thread-4 append 1, now builder value is 01, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 654647086
thread-0 append 2, now builder value is 012, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 566157654
thread-0 set new stringbuilder, now builder value is HelloWorld, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 1773033190
thread-4 append 2, now builder value is 012, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 654647086
thread-4 set new stringbuilder, now builder value is HelloWorld, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 700642750
thread-3 append 2, now builder value is 012, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 1803363945
thread-3 set new stringbuilder, now builder value is HelloWorld, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 1706743158
thread-2 append 2, now builder value is 012, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 1535812498
thread-2 set new stringbuilder, now builder value is HelloWorld, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 1431127699
thread-1 append 1, now builder value is 01, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 2075237830
thread-1 append 2, now builder value is 012, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 2075237830
thread-1 set new stringbuilder, now builder value is HelloWorld, ThreadLocal instance hashcode is 796465865, ThreadLocal instance mapping value hashcode is 1970695360
```

从输出中 `1~6` 行可以看出，不同线程访问的是同一个 `builder` 对象（不同线程输出的 `ThreadLocal instance hashcode` 值相同），但是每个线程获得的 `builder` 对象存储的实例 `StringBuilder` 不同（不同线程输出的 `ThreadLocal instance mapping value hashcode` 值不相同）。

从输出中 `1~2`、`9~10`行可以看出，同一个线程中修改 `builder` 对象存储的实例的值时，并不会影响到其他线程的 `builder` 对象存储的实例（`thread-4` 线程改变存储的 `StringBuilder` 的值并不会引起 `thread-0` 线程的 `ThreadLocal instance mapping value hashcode` 值发生改变）

从输出中 `9~13` 行可以看出，一个线程对 `ThreadLocal` 对象存储的值发生改变时，并不会影响其他的线程（`thread-0` 线程调用 `set` 方法改变本线程 `ThreadLocal` 存储的对象值，本线程的 `ThreadLocal instance mapping value hashcode` 发生改变，但是 `thread-4` 的 `ThreadLocal instance mapping value hashcode` 并没有因此改变）。

# 原理

`ThreadLocal` 能在每个线程间进行隔离，其主要是靠在每个 `Thread` 对象中维护一个 `ThreadLocalMap` 来实现的。因为是线程中的对象，所以对其他线程不可见，从而达到隔离的目的。那为什么是一个 `Map` 结构呢。主要是因为一个线程中可能有多个 `ThreadLocal` 对象，这就需要一个集合来进行存储区分，而用 `Map` 可以更快地查找到相关的对象。

`ThreadLocalMap` 是 `ThreadLocal` 对象的一个静态内部类，内部维护一个 `Entry` 数组，实现类似 `Map` 的 `get` 和 `put` 等操作，为简单起见，可以将其看做是一个 `Map`，其中 `key` 是 `ThreadLocal` 实例，`value` 是 `ThreadLocal` 实例对象存储的值。

## set

当调用 `ThreadLocal` 的 `set` 方法给变量设置值时，`ThreadLocal` 对象会先获取本线程的 `ThreadLocalMap` 对象，然后将当前的 `ThreadLocal` 对象及要设置值作为键值对放入 `Map` 中。

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    // 获取当前线程的 ThreadLocalMap 对象
    ThreadLocalMap map = getMap(t);
    if (map != null)
        // this 指当前的 ThreadLocal 对象
        map.set(this, value);
    else
        // key 不存在，则创建 map 并设置值
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
    // threadLocals 是 Thread 中的一个变量，因此是线程隔离的，不会受其他线程影响
    // 其在 Thread 类中的定义如下：ThreadLocal.ThreadLocalMap threadLocals = null;
    return t.threadLocals;
}
```

## get

获取 `ThreadLocal` 存储的对象值时，需要调用 `get` 方法。此方法也是先获取本线程的 `ThreadLocalMap` 对象，然后将当前的 `ThreadLocal` 对象作为 `key` 从 `Map` 中获取对应的值，如果没有，则返回一个初始 `null`。

```java
public T get() {
    Thread t = Thread.currentThread();
    // 获取当前线程的 ThreadLocalMap 对象
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // this 指当前的 ThreadLocal 对象
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

# 内存泄漏

 `ThreadLocalMap` 中的 `key` 是一个 `ThreadLocal` 对象，且是一个弱引用，而 `value` 却是一个强引用。

```java
static class ThreadLocalMap {
    /**
      * The entries in this hash map extend WeakReference, using
      * its main ref field as the key (which is always a
      * ThreadLocal object).  Note that null keys (i.e. entry.get()
      * == null) mean that the key is no longer referenced, so the
      * entry can be expunged from table.  Such entries are referred to
      * as "stale entries" in the code that follows.
      */
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
    // 其他代码
}
```

毫无疑问，如果线程执行完关闭，那么线程的所有对象都会被销毁，此时不会存在内存泄漏的问题。此外，在执行 `get`、`set` 操作时，调用进入 `ThreadLocalMap` 内部的函数，会对 `Entry` 进行检查，如果 `key` 为空，也会将 `value` 设置为空，让其可以被垃圾回收。所以一般情况下也不会造成内存泄漏。

```java
// get 或 set 方法，满足一定条件时会进入 expungeStaleEntry 方法
// 此方法内部会将 key 为 null 的 Entry 的 value 设置为 null，从而使得其可以被垃圾回收
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // 设置 value 值为 null，清空引用，让其可以被 GC 回收
    // expunge entry at staleSlot
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    // Rehash until we encounter null
    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        if (k == null) {
            // 设置 value 值为 null，清空引用，让其可以被 GC 回收
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;

                // Unlike Knuth 6.4 Algorithm R, we must scan until
                // null because multiple entries could have been stale.
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

但是，存在一种情况，可能导致内存泄漏。如果在某一时刻，将 `ThreadLocal` 实例设置为 `null`，即没有 `ThreadLocal` 没有强引用了，如果发生 GC 时，由于 `ThreadLocal` 实例只存在弱引用，所以被回收了，但是 `value` 仍然存在一个当前线程连接过来的强引用，其不会被回收，只有等到线程结束死亡或者手动清空 `value`  或者等到另一个 `ThreadLocal` 对象进行 `get` 或 `set` 操作时刚好触发 `expungeStaleEntry` 函数并且刚好能够检查到本 `ThreadLocal` 对象 `key` 为空（概率太小），这样才不会发生内存泄漏。否则，`value` 始终有引用指向它，它也不会被 GC 回收，那么就会导致内存泄漏。虽然发生内存泄漏的概率比较小，但是为了保险起见，也建议在使用完 `ThreadLocal` 对象后调用一下 `remove` 方法清理一下值。

![内存泄漏](/images/threadlocal.png)

# 与线程池结合使用

由于线程池是会复用线程的，因此如果在线程任务中对 `ThreadLocal` 没有经过重新设值而直接读取值的话，可能读取到的是该线程上一个任务赋值的结果，而不是本次任务的初始值，从而导致一些意向不到的错误。如下所示，创建一个固定大小是 3 的线程池，但是往线程池中放入 5 个任务，则最后两个任务会复用之前创建的线程，此时调用 `ThreadLocal` 的 `get` 方法获取到的是上一个任务赋值的结果，而不是本线程的初始值（程序输出的第`4~5` 行就是复用了线程 `11` 和 `13`，第一次获取到的是也是上一个任务赋的值 `2`，而不是本线程的初始值 `1`）。

```java
public class TestThreadLocalExecutor
{
    private static ThreadLocal<Integer> id = ThreadLocal.withInitial(() -> 1);

    public static void main(String[] args)
    {
        ExecutorService executor = Executors.newFixedThreadPool(3);
        for (int i = 0; i < 5; i++)
        {
            executor.execute(() -> {
                long threadId = Thread.currentThread().getId();
                // 任务开始时重新赋值，否则可能读取到的是上一个任务的值
                // id.set(1);
                int before = id.get();
                increment();
                int after = id.get();

                System.out.printf("Thread id: %d, before increment: %d, after increment: %d\n", threadId, before, after);
            });
        }

        executor.shutdown();
    }

    private static void increment()
    {
        int result = id.get() + 1;
        id.set(result);
    }
}
```

程序输出如下

```
Thread id: 11, before increment: 1, after increment: 2
Thread id: 13, before increment: 1, after increment: 2
Thread id: 12, before increment: 1, after increment: 2
Thread id: 13, before increment: 2, after increment: 3
Thread id: 11, before increment: 2, after increment: 3
```

为了避免如上情况的发生，可以在每个任务开始时，为 `ThreadLocal` 对象重新设置初始值（在 `get` 方法前先调用 `set` 方法），或者使用原生的创建线程的方式（跳开线程池的方式）。