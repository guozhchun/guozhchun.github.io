---
title: 类加载器
date: 2018-09-09 14:37:19
tags: [java, jvm]
categories: jvm
---

# 概述

类加载器主要是用来将 java 字节码文件（class 文件）加载到虚拟机中，也可以说是将 java 类加载到 java 虚拟机中。在 java 中，每一个类加载器都拥有一个独立的类名称空间，对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立起在 java 虚拟机中的唯一性。也就是说，要比较两个类是否相等，只有在两个类是由同一个类加载器加载的前提下才有效，否则即使两个类是同一个 class 文件加载而成，但是是由不同的类加载器进行加载的，那么两者进行比较也是不相等的。

<!-- more -->

# 双亲委派模型

java 中的类加载器基本上可以分成两类，一类是系统提供的，另一类是开发人员自己编写的。系统提供的类加载器主要有以下三类：

* 启动类加载器（Bootstrap ClassLoader）：它用来加载 java 的核心库（`${JRE_HOME}/lib/rt.jar`），是用原生代码来实现的，并不继承自 `java.lang.ClassLoader` 
* 扩展类加载器（Extension ClassLoader）：由 `sun.misc.Launcher$ExtClassLoader` 实现，负责加载 Java 的扩展库 ，如 `${JRE_HOME}/lib/ext` 目录下的类库
* 应用程序类加载器（Application ClassLoader）：由 `sun.misc.Launcher$AppClassLoader` 实现，负责加载用户类路径（classpath）上所指定的类库。一般而言，java 应用的类都是由它来完成加载的，可以通过 `ClassLoader.getSystemClassLoader()`来获取它 ，所以也可以称其为系统类加载器

其结构图大致如下所示

![类加载器双亲委派模型图](/images/jvm/classloader.png)

双亲委派模型的工作过程是：如果一个类加载器收到了类加载的请求，它首先不是自己尝试去加载这个类，而是交给其父类加载器进行尝试加载（注意：启动类加载器没有父类），如果父类加载器没加载成功，再交由自己的类加载器进行加载，如果仍然加载不成功，则抛出异常。

# 类加载器的主要函数

使用类加载器进行加载，主要会用到以下的函数

| 方法                                                 | 说明                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| getParent()                                          | 返回该类加载器的父类加载器。                                 |
| loadClass(String name)                               | 加载名称为 name 的类，返回的结果是  `java.lang.Class ` 类的实例。 |
| findClass(String name)                               | 查找名称为 name 的类，返回的结果是 `java.lang.Class` 类的实例。 |
| findLoadedClass(String name)                         | 查找名称为 name 的已经被加载过的类，返回的结果是 `java.lang.Class` 类的实例。 |
| defineClass(String name, byte[] b, int off, int len) | 把字节数组 `b`中的内容转换成 java 类，返回的结果是 `java.lang.Class`类的实例。这个方法被声明为 `final`的。 |
| resolveClass(Class<?> c)                             | 链接指定的 java 类。                                         |

一般情况下，调用类加载器加载某个类主要是用 loadClass 函数完成，在 loadClass 函数中，先通过 findLoadedClass 函数判断是否已经加载了某个类，如果没有加载，则调用父类加载器进行加载，如果父类加载器加载失败，则调用 findClass 函数尝试获取对应的 class 实例进行加载，而在 findClass 函数中，一般也会通过调用 defineClass 函数来将字节码数据转成对应的 class 实例。

# 自定义类加载器

自定义类加载器有两种模式：一种是通过继承 `ClassLoader` 类并重写 `loadClass` 方法，另一种是通过继承 `ClassLoader` 类并重写 `findClass` 方法。由于双亲委派模型在 `loadClass` 方法中已经保证了在调用父类的加载器加载失败后会调用本类的 `findClass` 方法进行类加载，因此建议通过重写 `findClass` 方法的方式进行自定义类加载器的编写。本例以重写 `findClass` 方法的方式进行自定义类加载器的编写。

## 要被加载器加载的类

这个类很简单，主要有两个函数：`setSample` 和 `sayHello`。其中 `sayHello` 主要用于验证类被自定义加载器正确加载，`setSample` 主要用于验证只有类加载器和加载的类都相同的情况下，两个类才是相等的。

需要注意的是，此类生成的 class 文件需要放在 main 函数加载不到的地方，否则此类会被应用加载器加载而不会触发自定义类加载器的加载。

```java
package jvm;

public class Sample
{
    private Sample sample;
    public void setSample(Object obj)
    {
        this.sample = (Sample) obj;
    }
    
    public void sayHello()
    {
        System.out.println("Hello World!");
    }
}
```

## 自定义类加载器类

```java
class FileClassLoader extends ClassLoader
{
    private static final String CLASS_SUFFIX = ".class";
    private String path;
    public FileClassLoader(String path)
    {
        this.path = path;
    }
    
    // 重写 findClass，用于加载类
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException
    {
        // 获取 class 文件的 byte 数据
        byte[] classBytes = getClassBytes(name);
        if (classBytes == null)
        {
            throw new ClassNotFoundException(name);
        }
        
        // 调用 defineClass 函数将 byte 数据转成对应的 class 类实例
        return defineClass(name, classBytes, 0, classBytes.length);
    }
    
    /**
     * 根据类的全限定名，找到对应的 class 文件，然后文件内的数据读取出来放入一个 byte 数组返回
     * @param className 类的全限定名
     * @return byte 数组，是类的 class 文件的数据。
     */
    private byte[] getClassBytes(String className)
    {
        String filePath = path + File.separator + className.replace(".", File.separator) + CLASS_SUFFIX;
        try
        {
            FileInputStream fileInputStream = new FileInputStream(filePath);
            ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
            byte[] data = new byte[1024 * 4];
            int length = 0;
            while ((length = fileInputStream.read(data)) != -1)
            {
                byteArrayOutputStream.write(data, 0, length);
            }
            fileInputStream.close();
            return byteArrayOutputStream.toByteArray();
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
        return null;
    }
}
```

## 验证程序

首先定义两个类加载器，用这两个类加载器加载 Sample 类并生成对应的实例，然后用反射方法分别调用实例的 `sayHello` 函数，用于验证类加载成功，再用反射方法调用实例的 `setSample` 方法用于验证在 java 中每个类都需要由加载它的类加载器和这个类本身一同确立起在 java 虚拟机中的唯一性。

需要注意的是，由于 Sample 类是通过自定义类加载器加载完成的，所以在调用 Sample 类中的函数时，不能直接调用，因为获取不到。有两种方法可以完成其调用：一是通过反射进行调用，二是通过接口的方式进行调用，此种方法需要调用方拥有与被加载类同样的接口并且调用方能够加载接口类。此处主要用反射的方式进行被加载类方法的调用。

```java
public class TestClassLoader
{
    public static void main(String[] args)
    {
        String currentPath = System.getProperty("user.dir");
        FileClassLoader fileClassLoader1 = new FileClassLoader(currentPath);
        FileClassLoader fileClassLoader2 = new FileClassLoader(currentPath);
        try
        {
            String className = "jvm.Sample";
            Class<?> class1 = fileClassLoader1.loadClass(className);
            Class<?> class2 = fileClassLoader2.loadClass(className);
            Object instance1 =  class1.newInstance();
            Object instance2 =  class2.newInstance();

            System.out.println("----class loader 1 indress: " + class1.getClassLoader() + "----");
            System.out.println("----class loader 2 indress: " + class2.getClassLoader() + "----");

            System.out.println("----class loader 1, instance sayHello----");
            Method method = class1.getDeclaredMethod("sayHello");
            method.invoke(instance1);
            
            System.out.println("----class loader 2, instance sayHello----");
            Method method2 = class2.getDeclaredMethod("sayHello");
            method2.invoke(instance2);
            
            System.out.println("----class loader 1, instance setSample----");
            Method method3 = class1.getDeclaredMethod("setSample", Object.class);
            method3.invoke(instance1, instance2);
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
        
    }
}
```

程序输出结果如下所示

```
----class loader 1 indress: jvm.FileClassLoader@6d06d69c----
----class loader 2 indress: jvm.FileClassLoader@4e25154f----
----class loader 1, instance sayHello----
Hello World!
----class loader 2, instance sayHello----
Hello World!
----class loader 1, instance setSample----
java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
	at java.lang.reflect.Method.invoke(Unknown Source)
	at jvm.TestClassLoader.main(TestClassLoader.java:96)
Caused by: java.lang.ClassCastException: jvm.Sample cannot be cast to jvm.Sample
	at jvm.Sample.setSample(Sample.java:8)
	... 5 more
```

从输出结果中可以看出，`fileClassLoader1` 和 `fileClassLoader2` 是两个不同的类加载器，虽然他们都成功地加载了同一个 class 文件（分别调用对应的 `sayHello` 函数都能正确输出结果），但是他们并不是同一个类，这通过 instance1（通过 fileClassLoader1 类加载器加载的 Sample 类生成的实例）调用 `setSample` 函数要将 instance2（通过 fileClassLoader2 类加载器加载的 Sample 类生成的实例）强制转换为 instance1 所对应的类时抛出 `ClassCastException` 异常可以证明。

# 其他

虽然在 java 世界中大部分的类加载器都使用双亲委派模型进行类的加载，而且这也是 java 设计者推荐给开发者的类加载器的实现方式，但是也例外的情况，比如 JNDI 和 OSGi，其并不能通过双亲委派模型解决问题，而是通过线程上下文类加载器或自定义其他类加载器模型完成。

# 参考资料

[1] 周志明. 深入理解Java虚拟机：JVM高级特性与最佳实践[M]. 北京：机械工业出版社，2013
[2] 成富. 深入探讨 Java 类加载器[J/OL]. https://www.ibm.com/developerworks/cn/java/j-lo-classloader/ ，2010-03-01