---
title: 命令模式
date: 2018-06-21 20:48:18
tags: 设计模式
categories: 设计模式
---

# 定义

> 命令模式将“请求”封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。命令模式也支持可撤销的操作。
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Head First 设计模式》

也就是说，命令者模式是让命令对象充当调用者（请求命令方）和接收者（执行命令方）中间的桥梁，调用者（请求命令方）拥有一个命令对象，命令对象拥有一个接收者（执行命令方），当调用者需要调用接收者的命令时，不是直接调用接收者，而是先通过调用命令对象的接口，由命令对象去内部的逻辑去调用接收方相应的操作，从而完成整个过程。因为命令对象的存在，调用者不需要知道接收者提供的接口，甚至可以无视接收者，因为调用者只和命令对象打交道。通过命令对象的隔离，可以使调用者和接收者进行解耦，后续接收者发生相应的变动也对调用者无感知。

<!-- more -->

# 类图

![命令模式类图](/images/designPattern/commandPattern/commandPatternUML.png)

> 说明：此图参考《Head First 设计模式》画出

# 例子

此处以射击类游戏为例，假设有一个角色，两种武器：刀和枪，角色使用武器进行攻击。在使用枪时需要先装子弹，然后才能攻击，而使用刀可以直接攻击。

## 类图

![命令模式例子类图](/images/designPattern/commandPattern/commandPatternSampleUML.png)

## 接收方类

接收方类有两个（模拟武器）：Knife和Gun，Knife中只有有一个方法stab，用于模拟刀的攻击，Gun中有两个方法，其中loadBullets用于模拟装子弹，fire用于模拟开枪，只有两个方法连起来才能模拟一次枪的攻击。

```java
class Knife
{
    public void stab()
    {
        System.out.println("Knife stabbing...");
    }
}

class Gun
{
    public void loadBullets()
    {
        System.out.println("Loading bullets...");
    }
    
    public void fire()
    {
        System.out.println("Gun firing...");
    }
}
```

## Command接口类

Command接口类很简单，只有一个execute方法，供调用者调用，执行命令

```java
interface Command
{
    void execute();
}
```

## Command接口实现类

Command接口实现类有两个：KnifeAttackCommand和GunAttackCommand，分别实现execute方法，模拟对应的攻击行为。

```java
class KnifeAttackCommand implements Command
{
    private Knife knife;
    
    public KnifeAttackCommand(Knife knife)
    {
        this.knife = knife;
    }
    
    @Override
    public void execute()
    {
        knife.stab();
    }
}

class GunAttackCommand implements Command
{
    private Gun gun;
    
    public GunAttackCommand(Gun gun)
    {
        this.gun = gun;
    }
    
    @Override
    public void execute()
    {
        gun.loadBullets();
        gun.fire();
    }
}
```

## 调用方类

Person类是调用方类，在这个类中持有一个Command对象，提供一个setCommond方法用于改变命令对象，在doAttack方法中调用命令对象的execute方法完成命令调用。

```java
class Person
{
    private Command command;
    
    public Person(Command command)
    {
        this.command = command;
    }
    
    public void setCommond(Command command)
    {
        this.command = command;
    }
    
    public void doAttack()
    {
        command.execute();
    }
}
```

## 验证程序

```java
public class TestCommand
{
    public static void main(String[] args)
    {
        // 初始化相关对象
        Knife knife = new Knife();
        Gun gun = new Gun();
        Command knifeAttackCommand = new KnifeAttackCommand(knife);
        Command gunAttackCommand = new GunAttackCommand(gun);
        
        // 用knifeAttackCommand初始化person对象
        System.out.println("----init person with knifeAttackCommand----");
        Person person = new Person(knifeAttackCommand);
        person.doAttack();
        
        System.out.println("----change command object to gunAttackCommand----");
        person.setCommond(gunAttackCommand);
        person.doAttack();
    }
}
```

输出如下

![命令模式例子输出结果](/images/designPattern/commandPattern/commandPatternSampleOutput.png)

# 与策略模式比较

命令模式和策略模式都是对行为进行封装，接收者及接收者的变化对调用者均无感知。这两种模式看起来似乎是相同的，甚至命令者模式也可以用策略模式实现（文章后面会将本文的例子用策略模式实现），策略模式也可以用命令者模式实现（本文同一个例子，用命令模式和策略模式分别实现，说明两者是可以转换的），那这两种模式是不是没有区别呢。事实上，命令模式与策略模式相比，多了一个“中介”，就是命令对象。命令模式对行为的封装是在命令对象中进行的，而策略模式对行为的封装是在接收者自己对象内进行的（参考对比本文的两个例子）。另一方面，相比而言，用命令模式实现相对比较复杂，而用策略模式实现则比较简单。但是，如果要对外统一提供接口，对行为进行封装，而又不能改变接收者类的时候（这种情况下一般发生在新增统一接口而原有类没实现该接口的场景下），此时命令模式就能派上用场了；而如果允许对接收这类进行修改，让接收者类实现统一的接口（这种情况下一般发生在接收者类和统一接口都是新增的或接收者类已经实现该接口的场景下），此时选用策略模式比较好（因为简单）。

接下来用策略模式实现上文中的例子。

## 类图

![策略模式实现命令模式例子类图](/images/designPattern/commandPattern/strategyPatternSampleUML.png)

对比这两个类图，可以发现，策略模式相比命令模式，少了命令对象，将命令对象进行的行为封装放在接收方类中，其他不变

## Command接口类

Command接口类与命令模式的Command接口类一样，只提供一个execute接口

```java
interface Command
{
    void execute();
}
```

## Command接口实现类（接收方类）

在策略模式中，接收方类与Command接口实现类是同一个，在接收方类中实现execute方法，完成对行为的封装。而在命令模式中，对行为的封装是在Command接口实现类中，在实现类中调用了接收方的相应方法。

```java
class Gun implements Command
{
    public void loadBullets()
    {
        System.out.println("Loading bullets...");
    }
    
    public void fire()
    {
        System.out.println("Gun firing...");
    }

    @Override
    public void execute()
    {
        loadBullets();
        fire();
    }
}

class Knife implements Command
{
    public void stab()
    {
        System.out.println("Knife stabbing...");
    }

    @Override
    public void execute()
    {
        stab();
    }
}
```

## 调用方类

调用方类Person与命令模式的调用方类Person一样

```java
class Person
{
    private Command command;
    
    public Person(Command command)
    {
        this.command = command;
    }
    
    public void setCommond(Command command)
    {
        this.command = command;
    }
    
    public void doAttack()
    {
        command.execute();
    }
}
```

## 验证程序

策略模式中，因为少了命令模式中的“中介”命令对象，所以初始化对象也会比较少。

```java
public class TestStrategy
{
    public static void main(String[] args)
    {
        // 初始化
        Command gun = new Gun();
        Command knife = new Knife();
        
        System.out.println("----init person with knife----");
        Person person = new Person(knife);
        person.doAttack();
        
        System.out.println("----change command object to gun----");
        person.setCommond(gun);
        person.doAttack();
    }
}
```

输出如下

![策略模式实现命令模式例子输出结果](/images/designPattern/commandPattern/strategyPatternSampleOutput.png)

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007