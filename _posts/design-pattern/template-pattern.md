---
title: 模板方法模式
date: 2018-06-23 16:26:33
tags: 设计模式
categories: 设计模式
---

# 定义

> 模板方法模式在一个方法中定义一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Head First 设计模式》

也就是说，模板方法是在一个方法中定义了整个流程，其中部分流程是在本类实现的，部分流程是需要由具体的子类实现的。

<!-- more -->

# 类图

![模板方法模式类图](/images/designPattern/templatePattern/UML.png)

> 说明：此图参考《Head First 设计模式》画出

# 例子

此处以竞技类游戏为例。假设游戏中有两种武器：刀和枪。用刀完成一次攻击需要完成以下流程：取出武器，刺杀，收回武器。用枪完成一次攻击需要完成以下流程：取出武器，装子弹，开枪，收回武器。

## 类图

![模板方法模式例子类图](/images/designPattern/templatePattern/sampleUML.png)

## 模板抽象类

Weapon是模板抽象类，里面有一个模板方法doAttack，该方法定义了攻击的整个流程：取武器takeWeapon、攻击attack、收回武器putBackWeapon。其中取武器takeWeapon和收回武器putBackWeapon是武器攻击的共有特征，放在抽象类中实现，而攻击attack根据武器类别的不同而不同，故定义成抽象方法供子类实现。

需要另外注意的是，为了避免模板方法被子类覆盖重写，此处将其定义成final

```java
abstract class Weapon
{
    // 这是模板方法，定义了整个攻击的流程
    // 将其定义为final，可以避免子类覆盖重写这个方法
    public final void doAttack()
    {
        takeWeapon();
        attack();
        putBackWeapon();
    }
    
    // 抽象方法，供子类实现，从而实现不同的行为
    protected abstract void attack();
    
    private void takeWeapon()
    {
        System.out.println("taking weapon...");
    }
    
    private void putBackWeapon()
    {
        System.out.println("putting bakc weapon....");
    }
}
```

## 模板具体实现类

模板具体实现类有两个Knife和Gun。其中Knife用于模拟刀，它有一个stab方法用于表示刀的特殊攻击行为。Gun用于模拟枪，它有loadBullets和fire方法用于表示枪的特殊攻击行为。这两个实现类分别继承了Weapon抽象类，实现attack抽象方法，在attack中实现自己独特的攻击行为。

```java
class Knife extends Weapon
{
    private void stab()
    {
        System.out.println("Knife stabbing...");
    }
    
    @Override
    public void attack()
    {
        stab();
    }
}

class Gun extends Weapon
{
    private void loadBullets()
    {
        System.out.println("Loading bullets...");
    }
    
    private void fire()
    {
        System.out.println("Gun firing...");
    }
    
    @Override
    public void attack()
    {
        loadBullets();
        fire();
    }
}
```

## 验证程序

验证程序很简单，直接初始化Knife和Gun，并分别调用模板方法doAttack实现一次攻击

```java
public class TestTemplate
{
    public static void main(String[] args)
    {
        Weapon knife = new Knife();
        Weapon gun = new Gun();
        
        System.out.println("----Knife do attack----");
        knife.doAttack();
        
        System.out.println("----Gun do attack----");
        gun.doAttack();
    }
}
```

输出如下，可以看出，出了attack方法中输出不同，其他（takeWeapon和putBackWeapon方法）输出是相同的

![模板方法模式例子输出结果](/images/designPattern/templatePattern/sampleOutput.png)

# 其他

模板方法模式是一种比较常用的设计，也是一种比较简单的设计模式。对于需要提供用户自定义行为的框架而言，模板方法设计模式是一种常用的也是一种合适的设计模式，因为其既保证了框架整体的流程不变性，又提供了部分子流程的可变性供客户自定义完成独特的要求。在项目中，涉及到提供定制点的功能实现，也大部分是采用模板方法设计模式来实现的。

## 与策略模式比较

模板方法模式与策略模式都是对“算法”进行封装，但是模板方法模式侧重的是对算法整理流程的定义和不变性，暴露部分子流程接口供子类实现，从而实现不同的功能；而策略模式则注重对整套算法的封装实现，其关注的是整个算法的互换性，当对某个算法进行更改时，是更改整套算法流程而不是其中的某个子流程。另一方面，从实现的角度而言，模板方法模式使用的是继承，而策略模式使用的是组合。

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007