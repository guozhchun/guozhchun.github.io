---
title: 迭代器模式
date: 2018-06-26 21:42:06
tags: 设计模式
categories: 设计模式
---

# 定义

> 迭代器模式提供一种方法顺序访问一个聚合对象中的各个元素，而不暴露其内部的表示。
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Head First 设计模式》

<!-- more -->

# 类图

![迭代器模式类图](/images/designPattern/iteratorPattern/UML.png)

> 说明：此图参考《Head First 设计模式》画出

# 例子

此处以衣服类中的衬衣和裤子进行遍历为例，分别用自己实现的迭代器和使用java库中提供的迭代器进行说明

## 自己定义迭代器

### 类图

![迭代器模式例子1类图](/images/designPattern/iteratorPattern/sampleUML.png)

### 迭代器接口

此处接口中有两个迭代器中的两个经典常用函数，`hasNext`用于判断是否还有下一个元素，`next`用于获取下一个元素

```java
interface Iterator
{
    boolean hasNext();
    Object next();
}
```

### 聚合类接口

聚合类接口主要提供一个函数用于获取迭代器对象

```java
interface Clothes
{
    Iterator createIterator();
}
```

### 接口具体实现类

此处将迭代器实现类放在聚合类的实现类中作为内部类使用，方便获取聚合类中的数据结构对象数据，能够有更好地封装效果，虽然这违反了单一责任原则。

在Shirt类中，使用的数据结构是数组，为简单起见，在数组中只设置了三个对象。在Trousers类中，使用的数据结构是ArrayList，为简单起见，也在List只设置了三个对象。

```java
class Shirt implements Clothes
{
    private String[] colors = new String[] {"red", "white", "black"};
    private int currentIndex = 0;
    private static final int MAX_NUM = 3;
    
    @Override
    public Iterator createIterator()
    {
        return new ShirtIterator();
    }
    
    // 内部类，实现Iterator接口
    private class ShirtIterator implements Iterator
    {
        @Override
        public boolean hasNext()
        {
            if (currentIndex >= 0 && currentIndex < MAX_NUM)
            {
                return true;
            }
            
            return false;
        }
        
        @Override
        public Object next()
        {
            String color = colors[currentIndex];
            currentIndex++;
            return color;
        }
    }
}

class Trousers implements Clothes
{
    private List<String> colors = new ArrayList<>();
    private int currentIndex = 0;
    
    public Trousers()
    {
        colors.add("white");
        colors.add("blue");
        colors.add("green");
    }
    
    @Override
    public Iterator createIterator()
    {
        return new TrousersIterator();
    }
    
    // 内部类，实现Iterator接口
    private class TrousersIterator implements Iterator
    {
        @Override
        public boolean hasNext()
        {
            if (currentIndex >= 0 && currentIndex < colors.size())
            {
                return true;
            }
            
            return false;
        }

        @Override
        public Object next()
        {
            String color = colors.get(currentIndex);
            currentIndex++;
            return color;
        }
        
    }
}
```

### 验证程序

```java
public class TestIterator
{
    // 遍历输出迭代器每一个对象
    private static void printIteratorObject(Iterator iterator)
    {
        while (iterator.hasNext())
        {
            System.out.print(iterator.next() + " ");
        }
        System.out.println();
    }
    
    public static void main(String[] args)
    {
        Clothes shirt = new Shirt();
        Clothes trousers = new Trousers();
        System.out.println("----traversal shirt----");
        printIteratorObject(shirt.createIterator());
        System.out.println("----traversal trousers----");
        printIteratorObject(trousers.createIterator());
    }
}
```

输出如下

![迭代器模式例子1输出结果](/images/designPattern/iteratorPattern/sampleOutput.png)

## 使用java类库迭代器

仍然以上面的例子，使用java类库迭代器进行实现。此时不需要自己定义迭代器接口。在java类库中，List已经实现了迭代器，直接调用`iterator()`函数即可；而数组没有实现迭代器，所以仍然需要自己手动实现。改动后的Clothes接口类、Shirt类和Trousers类如下

```java
// 使用java类库中的迭代器
import java.util.Iterator;

interface Clothes
{
    Iterator<?> createIterator();
}

class Shirt implements Clothes
{
    private String[] colors = new String[] {"red", "white", "black"};
    private int currentIndex = 0;
    private static final int MAX_NUM = 3;
    
    @Override
    public Iterator<?> createIterator()
    {
        return new ShirtIterator();
    }
    
    // 内部类，实现Iterator接口，java类库中数组没有实现迭代器，需要自己手动实现
    private class ShirtIterator implements Iterator<Object>
    {
        @Override
        public boolean hasNext()
        {
            if (currentIndex >= 0 && currentIndex < MAX_NUM)
            {
                return true;
            }
            
            return false;
        }
        
        @Override
        public Object next()
        {
            String color = colors[currentIndex];
            currentIndex++;
            return color;
        }
    }
}

class Trousers implements Clothes
{
    private List<String> colors = new ArrayList<>();
    
    public Trousers()
    {
        colors.add("white");
        colors.add("blue");
        colors.add("green");
    }
    
    @Override
    public Iterator<?> createIterator()
    {
    	// 直接调用java中list的迭代器接口
        return colors.iterator();
    }
}
```

验证程序跟上面的例子相同

```java
public class TestIterator
{
    // 遍历输出迭代器每一个对象
    private static void printIteratorObject(Iterator<?> iterator)
    {
        while (iterator.hasNext())
        {
            System.out.print(iterator.next() + " ");
        }
        System.out.println();
    }
    
    public static void main(String[] args)
    {
        Clothes shirt = new Shirt();
        Clothes trousers = new Trousers();
        System.out.println("----traversal shirt----");
        printIteratorObject(shirt.createIterator());
        System.out.println("----traversal trousers----");
        printIteratorObject(trousers.createIterator());
    }
}
```

输出如下

![迭代器模式例子2输出结果](/images/designPattern/iteratorPattern/sample2Output.png)

# 其他

迭代器设计模式把在元素之间游走的责任交给迭代器而不是聚合对象，这不仅让聚合的接口和实现变得更简洁，也可以让聚合更专注在它所应该专注的事情上面，而不必去理会遍历的事情。这虽然对聚合类的数据结构进行了封装，但是由于java类库中已经对集合类提供了迭代器的实现，在现实中更多的是直接调用集合类提供的迭代器方法，自己实现迭代器的场景还是比较少的。

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007