---
title: java中foreach实现原理
date: 2018-07-22 14:35:33
tags: java
categories: java
---

# 概述

循环作为程序中经常使用的语句，在java5之后推出了新的for/in(foreach)循环方式以方便程序员编写（阅读）代码。这种方式并不是新的语法，只是语法糖。即编写的foreach循环的代码并不是直接转成字节码，而是由编译器先转成对应的语法，然后再转成字节码，可以理解成是编译器对一些语法的封装提供的另一种方便阅读编写功能代码的实现方式。java中提供的foreach语法糖其底层实现方式主要有两种：对于集合类或实现迭代器的集合使用迭代器的遍历方式，对于数组集合使用数组的遍历方法。

<!-- more -->

# 迭代器遍历模式

对于实现Iterator接口的集合，使用foreach实现循环功能的代码会被编译器转换成使用迭代器遍历集合的代码，然后再转成字节码。例如以下的程序，使用foreach循环遍历ArrayList集合，使用`javac TestForEach.java`生成字节码后，再使用`javap -verbose TestForEach`进行反编译，从反编译的结果来看，可以看出其底层是用迭代器模式进行遍历的。

```java
import java.util.ArrayList;
import java.util.List;

public class TestForEach
{
    public static void main(String args[])
    {
        List<Integer> nums = new ArrayList<>();
        nums.add(11);
        nums.add(22);
        nums.add(33);

        for (Integer num : nums)
        {
            System.out.println(num);
        }
    }
}
```

反编译结果如下，从中可以看出，在106~118这十几行中是对集合进行遍历输出，在106行先使用`List.iterator()`接口生成迭代器，然后在109~118中不断使用`Iterator.hasNext()`判断是否有下个元素，有则使用`Iterator.next()`接口获取下个元素并进行输出。

```java
Classfile /C:/Users/zhchun/Desktop/TestForEach.class
  Last modified 2018-7-22; size 842 bytes
  MD5 checksum 45751115d8755b894835c52451125338
  Compiled from "TestForEach.java"
public class TestForEach
  SourceFile: "TestForEach.java"
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #13.#25        //  java/lang/Object."<init>":()V
   #2 = Class              #26            //  java/util/ArrayList
   #3 = Methodref          #2.#25         //  java/util/ArrayList."<init>":()V
   #4 = Methodref          #9.#27         //  java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
   #5 = InterfaceMethodref #28.#29        //  java/util/List.add:(Ljava/lang/Object;)Z
   #6 = InterfaceMethodref #28.#30        //  java/util/List.iterator:()Ljava/util/Iterator;
   #7 = InterfaceMethodref #31.#32        //  java/util/Iterator.hasNext:()Z
   #8 = InterfaceMethodref #31.#33        //  java/util/Iterator.next:()Ljava/lang/Object;
   #9 = Class              #34            //  java/lang/Integer
  #10 = Fieldref           #35.#36        //  java/lang/System.out:Ljava/io/PrintStream;
  #11 = Methodref          #37.#38        //  java/io/PrintStream.println:(Ljava/lang/Object;)V
  #12 = Class              #39            //  TestForEach
  #13 = Class              #40            //  java/lang/Object
  #14 = Utf8               <init>
  #15 = Utf8               ()V
  #16 = Utf8               Code
  #17 = Utf8               LineNumberTable
  #18 = Utf8               main
  #19 = Utf8               ([Ljava/lang/String;)V
  #20 = Utf8               StackMapTable
  #21 = Class              #41            //  java/util/List
  #22 = Class              #42            //  java/util/Iterator
  #23 = Utf8               SourceFile
  #24 = Utf8               TestForEach.java
  #25 = NameAndType        #14:#15        //  "<init>":()V
  #26 = Utf8               java/util/ArrayList
  #27 = NameAndType        #43:#44        //  valueOf:(I)Ljava/lang/Integer;
  #28 = Class              #41            //  java/util/List
  #29 = NameAndType        #45:#46        //  add:(Ljava/lang/Object;)Z
  #30 = NameAndType        #47:#48        //  iterator:()Ljava/util/Iterator;
  #31 = Class              #42            //  java/util/Iterator
  #32 = NameAndType        #49:#50        //  hasNext:()Z
  #33 = NameAndType        #51:#52        //  next:()Ljava/lang/Object;
  #34 = Utf8               java/lang/Integer
  #35 = Class              #53            //  java/lang/System
  #36 = NameAndType        #54:#55        //  out:Ljava/io/PrintStream;
  #37 = Class              #56            //  java/io/PrintStream
  #38 = NameAndType        #57:#58        //  println:(Ljava/lang/Object;)V
  #39 = Utf8               TestForEach
  #40 = Utf8               java/lang/Object
  #41 = Utf8               java/util/List
  #42 = Utf8               java/util/Iterator
  #43 = Utf8               valueOf
  #44 = Utf8               (I)Ljava/lang/Integer;
  #45 = Utf8               add
  #46 = Utf8               (Ljava/lang/Object;)Z
  #47 = Utf8               iterator
  #48 = Utf8               ()Ljava/util/Iterator;
  #49 = Utf8               hasNext
  #50 = Utf8               ()Z
  #51 = Utf8               next
  #52 = Utf8               ()Ljava/lang/Object;
  #53 = Utf8               java/lang/System
  #54 = Utf8               out
  #55 = Utf8               Ljava/io/PrintStream;
  #56 = Utf8               java/io/PrintStream
  #57 = Utf8               println
  #58 = Utf8               (Ljava/lang/Object;)V
{
  public TestForEach();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0       
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return        
      LineNumberTable:
        line 4: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
         0: new           #2                  // class java/util/ArrayList
         3: dup           
         4: invokespecial #3                  // Method java/util/ArrayList."<init>":()V
         7: astore_1      
         8: aload_1       
         9: bipush        11
        11: invokestatic  #4                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        14: invokeinterface #5,  2            // InterfaceMethod java/util/List.add:(Ljava/lang/Object;)Z
        19: pop           
        20: aload_1       
        21: bipush        22
        23: invokestatic  #4                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        26: invokeinterface #5,  2            // InterfaceMethod java/util/List.add:(Ljava/lang/Object;)Z
        31: pop           
        32: aload_1       
        33: bipush        33
        35: invokestatic  #4                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        38: invokeinterface #5,  2            // InterfaceMethod java/util/List.add:(Ljava/lang/Object;)Z
        43: pop           
        44: aload_1       
        45: invokeinterface #6,  1            // InterfaceMethod java/util/List.iterator:()Ljava/util/Iterator;
        50: astore_2      
        51: aload_2       
        52: invokeinterface #7,  1            // InterfaceMethod java/util/Iterator.hasNext:()Z
        57: ifeq          80
        60: aload_2       
        61: invokeinterface #8,  1            // InterfaceMethod java/util/Iterator.next:()Ljava/lang/Object;
        66: checkcast     #9                  // class java/lang/Integer
        69: astore_3      
        70: getstatic     #10                 // Field java/lang/System.out:Ljava/io/PrintStream;
        73: aload_3       
        74: invokevirtual #11                 // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
        77: goto          51
        80: return        
      LineNumberTable:
        line 8: 0
        line 9: 8
        line 10: 20
        line 11: 32
        line 13: 44
        line 15: 70
        line 16: 77
        line 17: 80
      StackMapTable: number_of_entries = 2
           frame_type = 253 /* append */
          offset_delta = 51
          locals = [ class java/util/List, class java/util/Iterator ]
             frame_type = 250 /* chop */
            offset_delta = 28

  }
```

因此，上面使用foreach方式遍历集合的程序与下面使用迭代器模式进行遍历的程序是一样的

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Iterator;

public class TestForEach
{
    public static void main(String args[])
    {
        List<Integer> nums = new ArrayList<>();
        nums.add(11);
        nums.add(22);
        nums.add(33);

        // 此处没有使用泛型，因为泛型在java中也是一种语法糖，只是编译器提供的一种检查，在运行期会擦除类型信息，其并不像C++那样在语法层面真正的支持泛型
        // 当然，为了良好的编码习惯，在平时的编码中应该使用泛型，即Iterator<Integer> iter = nums.iterator();
        Iterator iter = nums.iterator();
        while (iter.hasNext()) 
        {
            Integer num = (Integer)iter.next();
            System.out.println(num);    
        }
    }
}
```

# 数组依次遍历模式

数组没有实现Iterator接口，但是又要支持foreach语法糖，所以就用了最原始的最基本的依次遍历数组中的每个元素的方式来实现。如下代码是数组用foreach方式实现的遍历。

```java
public class TestForEach
{
    public static void main(String args[])
    {
        int[] nums = {11, 22, 33};
        for (int num : nums)
        {
            System.out.println(num);
        }
    }
}
```

同样使用`javac TestForEach.java`生成字节码后，再使用`javap -verbose TestForEach`进行反编译，输出结果如下。从中可以看出，从80~92这十几行是对数组进行遍历输出，这个过程没有使用迭代器，只是不断的对数进行出栈、比较、入栈、输出结果的操作。

```java
Classfile /C:/Users/zhchun/Desktop/TestForEach.class
  Last modified 2018-7-22; size 528 bytes
  MD5 checksum 874d6164dd77ec1874a96f4adb7d884b
  Compiled from "TestForEach.java"
public class TestForEach
  SourceFile: "TestForEach.java"
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #5.#17         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #18.#19        //  java/lang/System.out:Ljava/io/PrintStream;
   #3 = Methodref          #20.#21        //  java/io/PrintStream.println:(I)V
   #4 = Class              #22            //  TestForEach
   #5 = Class              #23            //  java/lang/Object
   #6 = Utf8               <init>
   #7 = Utf8               ()V
   #8 = Utf8               Code
   #9 = Utf8               LineNumberTable
  #10 = Utf8               main
  #11 = Utf8               ([Ljava/lang/String;)V
  #12 = Utf8               StackMapTable
  #13 = Class              #24            //  "[Ljava/lang/String;"
  #14 = Class              #25            //  "[I"
  #15 = Utf8               SourceFile
  #16 = Utf8               TestForEach.java
  #17 = NameAndType        #6:#7          //  "<init>":()V
  #18 = Class              #26            //  java/lang/System
  #19 = NameAndType        #27:#28        //  out:Ljava/io/PrintStream;
  #20 = Class              #29            //  java/io/PrintStream
  #21 = NameAndType        #30:#31        //  println:(I)V
  #22 = Utf8               TestForEach
  #23 = Utf8               java/lang/Object
  #24 = Utf8               [Ljava/lang/String;
  #25 = Utf8               [I
  #26 = Utf8               java/lang/System
  #27 = Utf8               out
  #28 = Utf8               Ljava/io/PrintStream;
  #29 = Utf8               java/io/PrintStream
  #30 = Utf8               println
  #31 = Utf8               (I)V
{
  public TestForEach();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0       
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return        
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=4, locals=6, args_size=1
         0: iconst_3      
         1: newarray       int
         3: dup           
         4: iconst_0      
         5: bipush        11
         7: iastore       
         8: dup           
         9: iconst_1      
        10: bipush        22
        12: iastore       
        13: dup           
        14: iconst_2      
        15: bipush        33
        17: iastore       
        18: astore_1      
        19: aload_1       
        20: astore_2      
        21: aload_2       
        22: arraylength   
        23: istore_3      
        24: iconst_0      
        25: istore        4
        27: iload         4
        29: iload_3       
        30: if_icmpge     53
        33: aload_2       
        34: iload         4
        36: iaload        
        37: istore        5
        39: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        42: iload         5
        44: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
        47: iinc          4, 1
        50: goto          27
        53: return        
      LineNumberTable:
        line 5: 0
        line 6: 19
        line 8: 39
        line 6: 47
        line 10: 53
      StackMapTable: number_of_entries = 2
           frame_type = 255 /* full_frame */
          offset_delta = 27
          locals = [ class "[Ljava/lang/String;", class "[I", class "[I", int, int ]
          stack = []
           frame_type = 248 /* chop */
          offset_delta = 25

}

```

对于用foreach方式实现的数组遍历方式，与下面的依次遍历数组中每个元素的方式是一样的

```java
public class TestForEach
{
    public static void main(String args[])
    {
        int[] nums = {11, 22, 33};
        for (int i = 0; i < nums.length; i++)
        {
            int num = nums[i];
            System.out.println(num);
        }
    }
}
```

# 其他

虽然foreach方便了程序的编写和阅读，是遍历集合和数组的一种好方式，但是使用foreach进行集合遍历时需要额外注意不能对集合长度进行修改，也就是不能对集合进行增删操作，否则会抛出`ConcurrentModificationException`异常。例如，下面程序会在执行第13行时抛出`ConcurrentModificationException`异常

```java
import java.util.ArrayList;
import java.util.List;

public class TestForEach
{
    public static void main(String args[])
    {
        List<Integer> nums = new ArrayList<>();
        nums.add(11);
        nums.add(22);
        nums.add(33);

        for (Integer num : nums)
        {
            if (num == 11)
            {
                // 此处使用集合中的remove操作，而不是迭代器中的remove操作，会导致迭代器中的expectedModCount和集合中的modCount变量不相等，从而导致在执行next()函数时抛出异常
                nums.remove((Integer)num);
            }
            else
            {
                System.out.println(num);
            }
        }
    }
}
```

虽然ArrayList的foreach底层用迭代器实现，迭代器也支持在遍历集合的过程中进行删除元素的操作，但是删除的函数必须是迭代器的函数，而不是集合自有的函数。至于上述代码为什么会抛出`ConcurrentModificationException`异常，可以从ArrayList中的迭代器类找到答案。ArrayList中部分源码如下所示

```java
// ArrayList中删除函数源码
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

// ArrayList中删除函数源码
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}

// ArrayList中迭代器函数源码
public Iterator<E> iterator() {
    return new Itr();
}

// ArrayList中迭代器类源码
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            // 此处重新赋值，避免跳过下一个元素
            cursor = lastRet;
            lastRet = -1;
            // 此处重新赋值，避免下次调用next()函数时校验不通过抛出异常
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

当执行`for (Integer num : nums)`语句时，会先调用ArrayList中的iterator()接口生成迭代器，而在初始化`Itr`类时会先将ArrayList对象中的`modCount`变量赋给Itr对象中的`expectedModCount`变量，在调用迭代器的`next`函数时会先调用`checkForComodification`函数进行校验，如果`expectedModCount`和`modCount`不相等则会抛出`ConcurrentModificationException`异常。在正常的集合遍历中，一般情况下，我们只使用迭代器中`hasNext`和`next`函数，并不会改变`expectedModCount`或者`modCount`的值，所以不会有问题，但是如果在遍历中调用了集合中自有的删除函数操作，则会改变`modCount`的值，从而导致`expectedModCount`与`modCount`不相等，进而在调用迭代器的`next`函数时进行校验不通过产生`ConcurrentModificationException`异常。而在遍历中调用迭代器的删除函数操作，由于其内部会在删除元素后对`expectedModCount`重新赋值，使其与`modCount`值相等，所以在遍历集合的过程中使用迭代器的删除函数操作不会有问题。

正确的在遍历集合过程中进行删除操作的方式如下

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Iterator;

public class TestForEach
{
    public static void main(String args[])
    {
        List<Integer> nums = new ArrayList<>();
        nums.add(11);
        nums.add(22);
        nums.add(33);

        Iterator<Integer> iter = nums.iterator();
        while (iter.hasNext()) 
        {
            Integer num = iter.next();
            if (num == 11)
            {
                // 在迭代器遍历中，不能使用集合自有的删除操作，只能使用迭代器中的删除操作，否则会导致迭代器中的expectedModCount和集合中的modCount变量不相等，从而导致在执行next()函数时抛出异常
                //nums.remove((Integer)num);
                iter.remove();
            }
            else
            {
                System.out.println(num);   
            } 
        }
        
    }
}
```

# 参考资料

[1] 朱小厮. Java语法糖之foreach[J/OL]. https://blog.csdn.net/u013256816/article/details/50736498 , 2016-02-25 