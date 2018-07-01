---
title: 外观模式
date: 2018-06-23 10:35:06
tags: 设计模式
categories: 设计模式
---

# 定义

> 外观模式提供了一个统一的接口，用来访问子系统中的一群接口。外观定义了一个高层接口，让子系统更容易使用。
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Head First 设计模式》

外观模式并没有“封装”子系统，只是额外增加一个类，在类中提供简化的接口完成对子系统中某些功能的实现，在提供简化接口的同时，并没有限制对子系统的访问，其依然将系统完整的功能暴露出来，供有需要的程序采用比较负载的方式访问子系统。

<!-- more -->

# 类图

![外观模式类图](/images/designPattern/facadePattern/UML.png)

> 说明：此图来源于《Head First 设计模式》

# 例子

此处以竞技类游戏为例，假设游戏中有两种武器：刀和枪。刀有刺杀行为，枪有装弹和开枪行为。假设要杀死一个人需要先用刀刺杀对方，然后装弹开枪，最后再补上一刀。

## 类图

![外观模式例子类图](/images/designPattern/facadePattern/facadePatternSampleUML.png)

## 子系统中的类

子系统中有两个类：Knife和Gun，分别表示刀和枪两种武器。在Knife类中有一个stab方法用来表示刺杀行为，在Gun类中有loadBullets表示装子弹行为，用fire表示开枪行为。

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

## 外观类

外观类提供了一个attack的接口供外部调用，该接口实现了杀死一个人的功能：先用刀刺杀，然后装弹开枪，最后再补上一刀。

```java
class MurderFacade
{
    private Knife knife;
    private Gun gun;
    
    public MurderFacade(Knife knife, Gun gun)
    {
        this.knife = knife;
        this.gun = gun;
    }
    
    public void attack()
    {
        knife.stab();
        gun.loadBullets();
        gun.fire();
        knife.stab();
    }
}
```

## 验证程序

在不使用外观模式的情况下，要完整杀死一个人需要四个步骤：knife.stab()、gun.loadBullets()、gun.fire()、knife.stab()，涉及两个类：Gun和Knife。而使用外观模式则只需要一个步骤：murderFacade.attack()，涉及一个类MurderFacade。显然，使用外观模式会让调用者更方便。另外，即使提供了外观模式，调用者也可以不使用外观类提供的接口，直接调用子系统的相应的方法进行组合完成目的（虽然有点繁琐）。

```java
public class TestFacade
{
    public static void main(String[] args)
    {
        Knife knife = new Knife();
        Gun gun = new Gun();
        
        System.out.println("----without using facade pattern----");
        knife.stab();
        gun.loadBullets();
        gun.fire();
        knife.stab();
        
        System.out.println("----using facade pattern----");
        MurderFacade murderFacade = new MurderFacade(knife, gun);
        murderFacade.attack();
    }
}
```

输出如下

![外观模式例子输出结果](/images/designPattern/facadePattern/sampleOutput.png)

# 其他

外观模式虽然在提供简化接口的同时并不限制外界对子系统的直接访问，但是如果外界直接调用的是外观类提供的接口而不是直接访问的子系统，这样就能将外部程序与子系统的实现进行解耦，当子系统发生变化时，只需要改变外观类接口的实现，不用让外部调用程序也相应发生改变。

更进一步，如果提供了外观类接口，同时让外部调用程序只调用外观类的接口而不直接调用子系统中的接口，这样就能很好地将系统进行隔离。这很适合用于将restful接口封装成一个外观类提供给外部调用，从而隔离本系统，达到系统（模块）之间的解耦。

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007