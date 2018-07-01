---
title: 组合模式
date: 2018-06-27 20:14:36
tags: 设计模式
categories: 设计模式
---

# 定义

> 组合模式允许你将对象组合成树形结构来表现“整体/部分”层次结构。组合能让客户以一致的方式处理个别对象以及对象组合。
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Head First 设计模式》

也就是说，组合模式是用来呈现树状结构的，利用组合模式，让叶子节点和非叶子节点继承同一个基础类，可以让叶子节点对象和组件（非叶子）节点对象之间的差异得以忽略，从而为客户程序提供更简单统一的调用方式。

<!-- more -->

# 类图

![组合模式类图](/images/designPattern/compositePattern/UML.png)

> 说明：此图参考《Head First 设计模式》画出

从类图中可以看出，组件（非叶子节点）既继承了基础类，也拥有基础类作为一个属性。继承基础类是为了和叶子节点有同一个基础类，从而对客户端屏蔽组件和叶子节点的差异，让客户端有统一接口可以调用，方便客户端程序。拥有基础类作为一个属性是因为组件类不是叶子节点，其叶子节点的数据放在这个属性中。

# 例子

此处以部门为例子进行说明。假设A部门底下有B、C、D三个部门，C部门底下有E、F部门，B、D、E、F部门都没有子部门。部门之间的层次结构如下图所示

![组合模式例子部门层次结构图](/images/designPattern/compositePattern/dept.png)

## 类图

![组合模式例子类图](/images/designPattern/compositePattern/sampleUML.png)

## 抽象基础类

抽象类Dept中有一个抽象函数description，用于描述本部门的信息。另外三个函数add、remove、getChild在抽象基础类中提供默认实现，主要是考虑到叶子类的情况，这样叶子类继承后只需要实现description函数即可。而非叶子类需要重写这四个函数。因为add、remove、getChild这三个函数对非叶子类有意义而对叶子类无意义。

当然，对于add、remove、getChild这三个函数，也可以定义成抽象的，然后再在叶子类和非叶子类中分别对其进行特殊的处理。

```java
abstract class Dept
{   
    public abstract void description();
    
    // 抽象类中提供默认实现，叶子类可以直接继承不用重写，非叶子类进行重写
    public void add(Dept dept)
    {
        System.out.println("No support method.");
    }
    
    // 抽象类中提供默认实现，叶子类可以直接继承不用重写，非叶子类进行重写
    public void remove(Dept dept)
    {
        System.out.println("No support method.");
    }
    
    // 抽象类中提供默认实现，叶子类可以直接继承不用重写，非叶子类进行重写
    public Dept getChild(int i)
    {
        System.out.println("No support method.");
        return null;
    }
}
```

## 叶子类

叶子类DeptLeaf中有一个属性name用于表示部门的名字，重写了description函数用于输出本部门的信息

```java
class DeptLeaf extends Dept
{
    private String name;
    
    public DeptLeaf(String name)
    {
        this.name = name;
    }
    
    @Override
    public void description()
    {
        System.out.println(name);
    }
}
```

## 非叶子类

非叶子类DeptComposite有一个属性name用于表示本部门的名字，有一个属性deptList用于存储子部门，并重写了抽象类中的所有方法。对于description方法，主要是依次调用deptList属性中元素的description方法，这样就形成了递归调用，从而能够将本类及本类的所有子类遍历到。

```java
class DeptComposite extends Dept
{
    private String name;
    private List<Dept> deptList = new ArrayList<>();

    public DeptComposite(String name)
    {
        this.name = name;
    }
    
    // 非叶子类，依次递归调用叶子类对应方法完成遍历
    @Override
    public void description()
    {
        System.out.println("DeptComposite " + name + " begin");
        for (Dept dept : deptList)
        {
            dept.description();
        }
        System.out.println("DeptComposite " + name + " end");
    }
    
    // 非叶子类，需要对此方法进行重写
    @Override
    public void add(Dept dept)
    {
        deptList.add(dept);
    }
    
    // 非叶子类，需要对此方法进行重写
    @Override
    public void remove(Dept dept)
    {
        deptList.remove(dept);
    }
    
    // 非叶子类，需要对此方法进行重写
    @Override
    public Dept getChild(int i)
    {
        return deptList.get(i);
    }
}
```

## 验证程序

在此程序中，先构造了部门B、部门D、部门E、部门F、然后用部门E、部门F组成部门C、再用部门B、部门C、部门D组成部门A，从而完成部门的层次结构构造。然后遍历输出部门A的组织结构，再遍历输出部门B的组织结构，最后遍历输出部门C的组织结构

```java
public class TestComposite
{
    public static void main(String[] args)
    {
        // 初始化叶子类
        Dept deptB = new DeptLeaf("Deptartment B");
        Dept deptD = new DeptLeaf("Deptartment D");
        Dept deptE = new DeptLeaf("Deptartment E");
        Dept deptF = new DeptLeaf("Deptartment F");
        
        // 初始化非叶子类
        Dept deptC = new DeptComposite("Deptartment C");
        deptC.add(deptE);
        deptC.add(deptF);
        
        Dept deptA = new DeptComposite("Deptartment A");
        deptA.add(deptB);
        deptA.add(deptC);
        deptA.add(deptD);
        
        // 对非叶子类A、叶子类B、非叶子类C进行遍历。可以看出，调用的是同一个接口
        System.out.println("----traversal Department A----");
        deptA.description();
        System.out.println("----traversal Department B----");
        deptB.description();
        System.out.println("----traversal Department C----");
        deptC.description();
    }
}
```

输出如下

![组合模式例子输出结果](/images/designPattern/compositePattern/sampleOutput.png)

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007