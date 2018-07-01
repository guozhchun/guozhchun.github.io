---
title: 适配器模式
date: 2018-06-22 20:58:48
tags: 设计模式
categories: 设计模式
---

# 定义

> 适配器模式将一个类的接口，转换成客户期望的另一个接口。适配器让原本接口不兼容的类可以合作无间。
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Head First 设计模式》

也就是说，适配器模式是让适配类在目标类和被适配者类中间做一个桥梁的转换，在不改变适目标类和被适配者类代码的情况下让两者可以互相调用。和命令模式的“中介”作用有点类似，甚至在实现上可以说命令模式的一部分功能是用适配器模式实现的（后续分析），但是从目的上来讲两者还是有差别的，命令模式是对“请求”的封装，让调用者对实现者无感知，而适配器模式是对接口的转换，是为了在让接口不变的情况下能够通信。

适配器有两种，一种是对象适配器，一种是类适配器。

<!-- more -->

# 对象适配器

## 类图

![适配器模式类图](/images/designPattern/adapterPattern/adapterPatternUML.png)

> 说明：此图参考《Head First 设计模式》画出

## 例子

此处以射击类游戏为例。假设有两种武器：刀和枪，刀有刺杀（stab）功能，枪有开枪（fire）的功能，现在用枪的开枪功能模拟刀的刺杀功能，也就是在调用stab函数时实际上是调用的fire函数。

### 类图

![适配器模式例子类图](/images/designPattern/adapterPattern/adapterPatternSampleUML.png)

### 目标接口

此处是用枪来模拟刀，所以目标接口是刀的接口。此处用StabAction接口类来表示，接口中只有一个stab抽象函数，用来表示刺杀行为。

```java
interface StabAction
{
    void stab();
}
```

### 目标接口实现类

目标接口实现类是Knife类，实现stab接口，表示刀的刺杀行为

```java
class Knife implements StabAction
{
    @Override
    public void stab()
    {
        System.out.println("Knife stabbing...");
    }
}
```

### 被适配者接口类和实现类

被适配者是枪的开枪行为接口类FireAction，该类的一个实现是Gun

```java
interface FireAction
{
    void fire();
}

class Gun implements FireAction
{    
    @Override
    public void fire()
    {
        System.out.println("Gun firing...");
    }
}
```

### 适配器

适配器要求实现目标接口，同时要求拥有一个被适配者对象属性。在stab方法中，通过调用被适配者gun的fire方法从而完成stab方法到fire方法的转换。

```java
class GunAdapter implements StabAction
{
    private FireAction gun;
    public GunAdapter(FireAction gun)
    {
        this.gun = gun;
    }
    
    @Override
    public void stab()
    {
        gun.fire();
    }
}
```

### 验证程序

```java
public class TestAdapter
{
    public static void main(String[] args)
    {
        // 初始化类对象
        StabAction knife = new Knife();
        FireAction gun = new Gun();
        StabAction gunAdapter = new GunAdapter(gun);
        
        System.out.println("----Knife stab----");
        knife.stab();
        
        System.out.println("----Gun fire----");
        gun.fire();
        
        System.out.println("----GunAdapter stab: it call stab method and print 'Gun firing...'----");
        gunAdapter.stab();
    }
}
```

输出如下

![适配器模式例子输出结果](/images/designPattern/adapterPattern/adapterPatternSampleOutput.png)

# 类适配器

## 类图

![类适配器模式类图](/images/designPattern/adapterPattern/classAdapterPatternUML.png)

> 说明：此图参考《Head First 设计模式》画出

对比类图可以发现，类适配器主要是使用继承实现的，即让适配器类继承被适配器类；而对象适配器主要是使用组合实现的，即让适配器拥有被适配者类。

## 例子

此处仍以对象适配器的例子进行说明比较。

### 类图

![类适配器模式例子类图](/images/designPattern/adapterPattern/classAdapterPatternSampleUML.png)

### 目标接口

与对象适配器中例子的目标接口相同

```java
interface StabAction
{
    void stab();
}
```

### 目标接口实现类

与对象适配器中例子的目标接口实现类相同

```java
class Knife implements StabAction
{
    @Override
    public void stab()
    {
        System.out.println("Knife stabbing...");
    }
}
```

### 被适配者接口类和实现类

与对象适配器中例子的被适配者接口类和实现类相同

```java
interface FireAction
{
    void fire();
}

class Gun implements FireAction
{    
    @Override
    public void fire()
    {
        System.out.println("Gun firing...");
    }
}
```

### 适配器

此处使用继承被适配器的方法完成适配器类，在stab方法中直接调用继承过来Gun对象中的fire方法，从而完成stab方法到fire方法的转换。

```java
class GunAdapter extends Gun implements StabAction
{
    @Override
    public void stab()
    {
        fire();
    }
}
```

### 验证程序

```java
public class TestAdapter
{
    public static void main(String[] args)
    {
        // 初始化类对象
        StabAction knife = new Knife();
        FireAction gun = new Gun();
        StabAction gunAdapter = new GunAdapter();
        
        System.out.println("----Knife stab----");
        knife.stab();
        
        System.out.println("----Gun fire----");
        gun.fire();
        
        System.out.println("----GunAdapter stab: it call stab method and print 'Gun firing...'----");
        gunAdapter.stab();
    }
}

```

输出如下

![类适配器模式例子输出结果](/images/designPattern/adapterPattern/classAdapterPatternSampleOutput.png)

可以看出，类适配器和对象适配器的输出结果是相同的

# 比较

对象适配器和类适配器都能在不改变原有接口的情况下实现接口的通信，完成一个接口到另一个接口的转换。但是对象适配器使用的是组合，而类适配器使用的是继承。使用组合不仅可以适配某个类，也可以适配该类的任何子类，而使用继承则可以覆盖被适配者的行为。然而，由于java不支持多继承，且本着“组合优于继承”的原则，还是建议多使用对象适配器的方式，少使用类适配器的方式。

# 其他

## 与命令模式比较

命令模式是将“请求”封装成一个命令对象，让调用者持有这个命令对象，调用者通过调用命令对象提供的接口让命令对象持有的实现者完成特定的操作。这个过程中，命令对象就相当于一个适配器的作用，对比命令模式的类图和适配器的类图也可以看出，调用者 -> 命令对象 -> 实现者 这三者就是一个适配器模式的类图，如下图所示

![命令模式与适配器模式类图比较](/images/designPattern/adapterPattern/adapterPatternCompareWithCommandPatternUML.png)

## 与装饰者模式比较

装饰者模式是对一个装饰对象新增功能，用来装饰的对象不但继承了装饰者对象，同时也拥有装饰者对象作为一个属性。对于适配者模式而言，适配器实现目标接口，同时继承或组合被装饰者。即装饰者模式对同一个对象同时应用了继承和组合，而适配器模式对同一个对象只会运用继承或组合中的一种。另外，从功能角度而言，装饰者模式是为对象附上新的功能，不会进行接口的转换，而适配器模式一定会对接口进行转换，不会为对象附上新的功能。

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007