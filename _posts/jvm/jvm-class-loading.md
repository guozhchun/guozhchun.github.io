---
title: jvm 类加载过程
date: 2018-09-02 11:14:03
tags: [java, jvm]
categories: jvm
---

# 类生命周期

类从被加载到虚拟机内存开始，到卸载出内存为止，其需要经历包含以下 7 个阶段的生命周期：加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化化（Initialization）、使用（Using）、卸载（Unloading）。其中，验证、准备、解析三个部分统称为连接。其图示如下：

![类生命周期](/images/jvm/classLifecycle.png)

<!-- more -->

这些阶段是按部就班地开始，但是并不一定是需要等上一个阶段完成后才开始下一个阶段，有可能上一个步骤在进行时，下一个步骤也开始执行了。

# 加载

加载是类加载过程中的第一个阶段，在这个阶段中，虚拟机需要完成以下三件事

* 通过一个类的全限定名来获取定义此类的二进制字节流
* 将这个字节流所代表的静态存储结构转换为方法区的运行时的数据接结构
* 在内存中生成一个代表这个类的 `java.lang.Class` 对象，作为方法区这个类的各种数据的访问入口

而关于“通过一个类的全限定名来获取定义此类的二进制字节流”这一条中，并没有规定一定要从硬盘中的 class 文件中获取，也可以从 zip 包中读取（例如：jar包、war包），或是从网络中获取（例如：Applet），当然，也可以是运行时生成（如：动态代理）或从其他文件中读取（如，JSP）······

在加载阶段完成之后，虚拟机外部的二进制流就按照虚拟机所需的格式存储在方法区之中，而对于方法区中的数据存储格式，虚拟并没有规定，可以由虚拟机实现者自己决定。

# 验证

验证是连接阶段的第一步，一般情况下，在加载阶段进行的过程中，会启动验证阶段对二进制流进行验证。由于 class 文件并不一定是由 java 源码编译生成（可能是由其他编程语言产生，也可能是直接用编辑器输出 16 进制的文件），所以对读取的二进制字节流进行验证显得格外重要，否则可能会对整个系统产生严重破坏。

虚拟机中在验证阶段进行了很多项检验动作，这些动作主要可以归纳为四个方面：

* 文件格式验证：主要是对“魔数”、版本号等类型信息的验证，主要是要确保解析的二进制字节流在格式上符合描述一个 java 类型信息的要求，在这一阶段完成后，二进制字节流就被存储在方法区中了，后续的验证都是基于方法区中进行，不会再操作二进制字节流。
* 元数据验证：这个阶段主要是对字节码进行语义分析，确保描述的信息符合 java 语言规范的要求，主要包括验证类的继承关系、接口关系、数据类型等是否合法。
* 字节码验证：这个阶段是最复杂的一个阶段，主要是通过数据流和控制流分析，确保程序语义的合法性。主要是对类的方法体进行校验（校验执行指令），确保方法在运行时不会对虚拟机造成破坏。
* 符号引用验证：这个阶段一般与解析阶段并行，主要是确保所有的符号引用都能得到正确的解析。

# 准备

准备阶段是正式为类变量分配内存并设置类变量初始值的阶段。这些变量所使用的内存都在方法区中进行分配，初始值指“零值”，需要注意的是被 final 定义的变量，其值不是零值，而是指定的具体的值，因为在编译阶段就会将这些常量值生成 ConstantValue 属性，在准备阶段时会根据 ConstantValue 将对应的变量设置成相应的常量值。此过程只涉及类成员变量（static 变量），不涉及实例成员变量（这是在初始化阶段进行的，分配在堆中）。

# 解析

解析是虚拟机将常量池内的符号引用替换为直接引用的过程，类似于根据指针找到对应的内存区域的过程。解析动作主要包含以下 7 中类型

* 类或接口：对应常量池中的 CONSTANT_Class_info
* 字段：对应常量池中的 CONSTANT_Fieldref_info，解析过程中先搜索本类、再搜索接口、再搜索父类
* 类方法：对应常量池中的 CONSTANT_Methodref_info，解析过程中先搜索本类，再搜索父类、再搜索接口
* 接口类型：对应常量池中的 CONSTANT_InterfaceMethodref_info
* 方法类型：对应常量池中的 CONSTANT_MethodType_info
* 方法句柄：对应常量池中的 CONSTANT_MethodHandle_info
* 调用点：对应常量池中的 CONSTANT_InvokeDynamic_info

# 初始化

## 时机

虚拟机规范中严格规定了有且仅有以下 5 中情况会立即对类进行初始化：

* 遇到 new、getstatic（读取静态变量，但是非 fianl 修饰的变量，因为 final 修饰的变量在编译期已经被放入常量池中）、putstatic（设置静态变量）、invokestatic（调用静态方法）这四条字节码指令时，如果没有对类进行初始化，需要立即对类进行初始化
* 使用 `java.lang.reflect` 包的方法对类进行反射调用
* 当初始化一个类的时候，如果发现父类还没有进行过初始化，则需要先触发其父类的初始化
* 当虚拟机启动时，用户需要指定一个要执行的主类（包含 main 方法的类），虚拟机会先初始化这个主类
* 当使用动态语言支持时，如果一个 `java.lang.invoke.MethodHandle` 实例最后的解析结果 `REF_getStatic`、`REF_putStatic`、`REF_invokeStatic`的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化

## 过程

类初始化阶段是执行类构造器 <clinit\> 方法的过程，其主要有以下特点

* <clinit\> 是由编译器自动收集类中的所有类变量（static 变量）的赋值动作（单纯定义的不包含）和静态语句块（static 代码块）中的语句合并产生，其按照代码出现的先后顺序进行搜集。也就是说，初始化过程会执行类静态变量和静态代码块，同时按照出现的先后顺序从上到下执行
* 静态语句块中只能访问到定义在静态语句块之前的变量，定义在它后面的变量，在前面的的静态语句块中可以赋值，但是不能访问
* 父类中定义的静态变量和静态语句块的初始化先于子类完成。虚拟机会保证在子类的 <clinit\> 方法执行之前，父类的 <clinit\> 方法已经执行完毕
* <clinit\> 方法对于类或接口来说并不是必须的，如果一个类中没有静态语句块，也么有对变量的赋值操作，那么编译器可以不为这个类生成 <clinit\> 方法
* 接口中不能使用静态语句块，但仍然有变量初始化的复制操作，因此接口与类一样会生成 <clinit\> 方法，只是接口执行 <clinit\> 方法不需要先执行父接口的 <clinit\> 方法。只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的 <clinit\> 方法

# 例子

以下的例子是从[网上](https://blog.csdn.net/noaman_wgs/article/details/74489549)看到的，作为补充，能更好地理解类的加载过程。

```java
class Singleton1
{
    private static Singleton1 instance = new Singleton1();
    public static int value1;
    public static int value2 = 10;
    
    public Singleton1()
    {
        value1++;
        value2++;
        System.out.println("Singleton1() value1: " + value1 + ", value2: " + value2);
    }
    
    public static Singleton1 getInstance()
    {
        return instance;
    }
}

class Singleton2
{
    public static int value1;
    public static int value2 = 10;
    private static Singleton2 instance = new Singleton2();
    
    public Singleton2()
    {
        value1++;
        value2++;
        System.out.println("Singleton2() value1: " + value1 + ", value2: " + value2);
    }
    
    public static Singleton2 getInstance()
    {
        return instance;
    }
}

public class TestInit
{
    public static void main(String[] args)
    {
        Singleton1 singleton1 = Singleton1.getInstance();
        System.out.println("Singleton1 value1 is: " + singleton1.value1);
        System.out.println("Singleton1 value2 is: " + singleton1.value2);
        
        Singleton2 singleton2 = Singleton2.getInstance();
        System.out.println("Singleton2 value1 is: " + singleton2.value1);
        System.out.println("Singleton2 value2 is: " + singleton2.value2);
    }
}
```

程序输出如下

```
Singleton1() value1: 1, value2: 1
Singleton1 value1 is: 1
Singleton1 value2 is: 10
Singleton2() value1: 1, value2: 11
Singleton2 value1 is: 1
Singleton2 value2 is: 11
```

首先解析 Singleton1 类的三个输出语句。在程序执行到 `Singleton1 singleton1 = Singleton1.getInstance()` 时会触发 Singleton1 类的加载、验证、准备、解析、初始化阶段。在类的解析阶段，会为类的静态成员变量赋零值，此时 `instance = null, value1 = 0, value2 = 0`，然后执行初始化阶段，对静态成员变量进行赋值操作。首先是 instance 变量的赋值，这会 new 一个 Singleton1 对象，执行 Singlton1 的无参构造函数，执行后 `value1 = 1, value2 = 1`，同时输出 `Singleton1() value1: 1, value2: 1`，接着执行 value2 变量的赋值，执行后 `value2 = 10`，虽然 value1 也是静态变量，但是其没有显式的赋值操作，所以不会再次为其赋值。接下来程序执行对 value1 和 value2 的输出操作，所以输出 `Singleton1 value1 is: 1` 和 `Singleton1 value2 is: 10`。

现在解析 Singleton2 类的三个输出语句。在程序执行到 `Singleton2 singleton2 = Singleton2.getInstance()` 时会触发 Singleton2 类的加载、验证、准备、解析、初始化阶段。在类的解析阶段，会为类的静态成员变量赋零值，此时 `instance = null, value1 = 0, value2 = 0`，然后执行初始化阶段，对静态成员变量进行赋值操作。首先执行 value2 变量的赋值，执行后 `value2 = 10`，接着是 instance 变量的赋值，这会 new 一个 Singleton2 对象，执行 Singlton2 的无参构造函数，执行后 `value1 = 1, value2 = 11`，同时输出 `Singleton2() value1: 1, value2: 11`，虽然 value1 也是静态变量，但是其没有显式的赋值操作，所以不会再次为其赋值。接下来程序执行对 value1 和 value2 的输出操作，所以输出 `Singleton2 value1 is: 1` 和 `Singleton2 value2 is: 11`。

# 参考资料

[1] 周志明. 深入理解Java虚拟机：JVM高级特性与最佳实践[M]. 北京：机械工业出版社，2013
[2] 是Guava不是瓜娃. JVM(四)—一道面试题搞懂JVM类加载机制[J/OL]. https://blog.csdn.net/noaman_wgs/article/details/74489549，2017-07-05 