---
title: 单例模式
date: 2018-06-19 20:22:09
tags: 设计模式
categories: 设计模式
---

# 定义

单例模式是用得比较多的一种设计模式，也是开发人员最熟悉的一种模式，其表示一个类只有一个实例对象，在《Head First 设计模式》中，其定义如下

> 单例模式确保一个类只有一个实例，并提供一个全局访问点。
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Head First 设计模式》

单例有多种写法，本文主要描述单类加载器下常见的几种写法。

<!-- more -->

# 简单方式

此写法在单线程环境下没有任何问题，同时也是懒加载的，对性能也无太大影响，但是在多线程环境下可能出现实例化多个对象的情况。当一个线程A在`if (instance == null)`执行后切换到另一个线程B，线程B判断到此时还未实例化对象，所以新建了一个对象返回，此时，切换到线程A继续执行，由于已经判断了instance为空，所以线程A也新建了一个对象返回。这是线程A和线程B得到的是两个不同的实例对象。程序如下

```java
class Singleton
{
    private static Singleton instance;
    private Singleton()
    {
        System.out.println(Thread.currentThread() + " Construct Singleton class!");
    }
    
    public static Singleton getInstance()
    {
        if (instance == null)
        {
            instance = new Singleton();
        }
        
        return instance;
    }
    
    // 其他函数，测试输出
    public void doSomething()
    {
        System.out.println(Thread.currentThread() + " do something!");
    }
}
```

测试程序如下（如无特殊说明，本文后续的例子测试程序均为此测试程序）。主要是在每个线程中获取单例，然后调用`doSomething`函数验证输出结果。

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class TestSingleton
{
    public static void main(String[] args)
    {
        // 新建5个线程，每个线程都调用Singleton.getInstance()
        // 通过构造函数中的输出看是否是线程安全
        ExecutorService exec = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++)
        {
            exec.execute(new Runnable() {
                @Override
                public void run()
                {
                    Singleton.getInstance().doSomething();
                }
            });
        }
        exec.shutdown();
    }
}
```

程序输出如下（*注意：由于是多线程，每次输出结果可能不一致*）

```
Thread[pool-1-thread-4,5,main] Construct Singleton class!
Thread[pool-1-thread-2,5,main] Construct Singleton class!
Thread[pool-1-thread-4,5,main] do something!
Thread[pool-1-thread-3,5,main] Construct Singleton class!
Thread[pool-1-thread-3,5,main] do something!
Thread[pool-1-thread-2,5,main] do something!
Thread[pool-1-thread-1,5,main] do something!
Thread[pool-1-thread-5,5,main] do something!
```

# 同步函数方式

这种方法是直接在`getInstance`函数上加上`synchronized`关键字，使其成为线程安全的函数。这样保持了其懒加载的特点，不会在类加载时就初始化整个类而可能导致资源的浪费。但是对性能有一定的影响，因为每次调用该方法都需要获取该类的锁，不能保证并行获取对象实例，而事实上，只有第一次初始化对象的时候才需要保证线程安全，后续直接调用函数获取对象的实例并不需要加锁，是可以并行获取对象实例的。程序如下

```java
class Singleton
{
    private static Singleton instance;
    private Singleton()
    {
        System.out.println(Thread.currentThread() + " Construct Singleton class!");
    }
    
    public static synchronized Singleton getInstance()
    {
        if (instance == null)
        {
            instance = new Singleton();
        }
        
        return instance;
    }
    
    // 其他函数，测试输出
    public void doSomething()
    {
        System.out.println(Thread.currentThread() + " do something!");
    }
}
```

程序输出如下（*注意：由于是多线程，每次输出结果可能不一致*）

```java
Thread[pool-1-thread-1,5,main] Construct Singleton class!
Thread[pool-1-thread-2,5,main] do something!
Thread[pool-1-thread-4,5,main] do something!
Thread[pool-1-thread-5,5,main] do something!
Thread[pool-1-thread-3,5,main] do something!
Thread[pool-1-thread-1,5,main] do something!
```

# 双重检查加锁方式

双重检查加锁（double-checked locking）方式是在简单写法的基础上，在`if (instance == null)`条件里增加同步代码块和空判断保证线程安全。同时，这种方法实现的单例也是懒加载的。程序如下

```java
class Singleton
{
    // 需要注意：必须使用volatile关键字保证可见性
    private volatile static Singleton instance;
    private Singleton()
    {
        System.out.println(Thread.currentThread() + " Construct Singleton class!");
    }
    
    public static Singleton getInstance()
    {
        if (instance == null)
        {
            synchronized (Singleton.class)
            {
                // 此处必须有一层判断，否则可能存在以下情况：线程A判断为空，未进入同步块，切换到线程B，
                // 此时对象未初始化，线程B进入同步块获取锁，并创建对象完毕，然后切换到线程A继续执行，
                // 线程A进入同步块，如果没有这一层空判断，此时线程A也会创建一个对象，
                // 导致两个线程的对象不一致，进而出现线程不安全的现象
                if (instance == null)
                {                    
                    instance = new Singleton();
                }
            }
        }
        
        return instance;
    }
    
    // 其他函数，验证输出
    public void doSomething()
    {
        System.out.println(Thread.currentThread() + " do something!");
    }
}
```

需要注意以下几点

* 静态变量`instance`必须有`volatile`关键字修饰，这是为了保证可见性和避免指令重排序，只有加上这个关键字，在一个线程中对`instance`的修改会立刻反映到其他线程上，从而让其他线程在判断`instance`是否为空时能获取到准确的值，避免产生多个对象
* 在同步代码块中需要增加`instance`是否为空的判断，这样才能保证线程安全。否则，可能出现以下的情况：线程A判断为空，未进入同步块，切换到线程B，此时对象未初始化，线程B进入同步块获取锁，并创建对象完毕，然后切换到线程A继续执行，线程A进入同步块，如果没有这一层空判断，此时线程A也会创建一个对象， 导致两个线程的对象不一致，进而出现线程不安全的现象
* `volatile`关键字是在JDK1.5之前的版本中，许多JVM的实现会导致双重检查加锁的失效，因此，此种方式只适合于JDK1.5及其之后的版本

程序输出如下（*注意：由于是多线程，每次输出结果可能不一致*）

```
Thread[pool-1-thread-4,5,main] Construct Singleton class!
Thread[pool-1-thread-1,5,main] do something!
Thread[pool-1-thread-2,5,main] do something!
Thread[pool-1-thread-4,5,main] do something!
Thread[pool-1-thread-5,5,main] do something!
Thread[pool-1-thread-3,5,main] do something!
```

# 静态成员变量初始化方式

此种方式是在类加载时就直接对整个对象进行实例化，优点是由JVM保证在任何线程访问静态变量前一定会创建该实例，从而保证线程安全。缺点是类加载时立刻进行对象实例化，可能导致整个类并没有使用却占用者资源。程序如下

```java
class Singleton
{
    private static Singleton instance = new Singleton();
    private Singleton()
    {
        System.out.println(Thread.currentThread() + " Construct Singleton class!");
    }
    
    public static Singleton getInstance()
    {   
        return instance;
    }
    
    // 其他函数，测试输出
    public void doSomething()
    {
        System.out.println(Thread.currentThread() + " do something!");
    }
}
```

程序输出如下（*注意：由于是多线程，每次输出结果可能不一致*）

```java
Thread[pool-1-thread-1,5,main] Construct Singleton class!
Thread[pool-1-thread-4,5,main] do something!
Thread[pool-1-thread-3,5,main] do something!
Thread[pool-1-thread-5,5,main] do something!
Thread[pool-1-thread-2,5,main] do something!
Thread[pool-1-thread-1,5,main] do something!
```

# 静态内部类方式

此种方式是参考静态成员变量初始化方式实现的，因为静态成员变量只有在类加载时才会进行初始化，且由JVM保证只有一个实例。当`Singleton.getInstance`方法没调用时，不会触发静态内部类`Holder`的加载，此时也不会进行`Singleton`的实例化，只有当`Singleton.getInstance`方法没调用时才会触发内部类`Holder`的加载，进而实例化`Singleton`对象，因此这是懒加载的。程序如下

```java
class Singleton
{
    private Singleton()
    {
        System.out.println(Thread.currentThread() + " Construct Singleton class!");
    }
    
    public static Singleton getInstance()
    {   
        return Holder.singleton;
    }
    
    // 静态内部类，只有Singleton.getInstance调用时才会加载此类
    // 在加载此类时初始化化Singleton对象，由于是静态变量，故只有一个
    private static class Holder
    {
        private static Singleton singleton = new Singleton();
    }
    
    // 其他函数，验证输出
    public void doSomething()
    {
        System.out.println(Thread.currentThread() + " do something!");
    }
}
```

程序输出如下（*注意：由于是多线程，每次输出结果可能不一致*）

```
Thread[pool-1-thread-2,5,main] Construct Singleton class!
Thread[pool-1-thread-2,5,main] do something!
Thread[pool-1-thread-4,5,main] do something!
Thread[pool-1-thread-3,5,main] do something!
Thread[pool-1-thread-5,5,main] do something!
Thread[pool-1-thread-1,5,main] do something!
```

# 枚举方式

枚举是JDK1.5引入的新的数据结构，其由JVM保证线程安全，同时由JVM保证在反射和反序列化的情况下仍然是单例的（其他的写法并不能保证，或需要自己手动在程序中保证）。另外，枚举类是在第一次访问时才被实例化，所以这也是懒加载的。程序如下 

```java
enum Singleton
{
    INSTANCE;
    
    // 此函数可以不提供，而直接使用 Singleton.INSTANCE 获取Singleton实例
    public static Singleton getInstance()
    {
        return INSTANCE;
    }
    
    // 其他函数，验证输出
    public void doSomething()
    {
        System.out.println(Thread.currentThread() + " do something!");
    }
}
```

程序输出如下（*注意：由于是多线程，每次输出结果可能不一致*）

```
Thread[pool-1-thread-3,5,main] do something!
Thread[pool-1-thread-5,5,main] do something!
Thread[pool-1-thread-2,5,main] do something!
Thread[pool-1-thread-1,5,main] do something!
Thread[pool-1-thread-4,5,main] do something!
```

由于枚举类不用自己写构造函数，所以不会输出`Construct Singleton class!`语句，只输出了调用的函数`doSomething`中的`do something`语句。另外，在使用枚举类获取单例时，可以直接用`Singleton.INSTANCE`，更进一步，获取单例后调用对象方法可以直接写`Singleton.INSTANCE.doSomething`，这样就不用写`getInstance`方法。

# 比较

| 简单方式     | 同步函数方式       | 双重检查加锁方式 | 静态成员变量初始化方式 | 静态内部类方式 | 枚举方式       |
| ------------ | ------------------ | ---------------- | ---------------------- | -------------- | -------------- |
| 非线程安全   | 线程安全           | 线程安全         | 线程安全               | 线程安全       | 线程安全       |
| 懒加载       | 懒加载，但影响性能 | 懒加载           | 非懒加载，可能浪费资源 | 懒加载         | 懒加载         |
| 适用于版本 >= JDK1.0 | 适用于版本 >= JDK1.0 | 适用于版本>= JDK1.5 | 适用于版本>= JDK1.0 | 适用于版本>= JDK1.0 | 适用于版本>= JDK1.5 |

# 扩展阅读

[https://www.cnblogs.com/bugly/p/6541983.html](https://www.cnblogs.com/bugly/p/6541983.html)

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007