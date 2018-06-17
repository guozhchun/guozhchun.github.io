---
title: 策略模式
date: 2018-06-13 20:12:05
tags: 设计模式
categories: 设计模式
---

# 定义

策略模式定义了算法族，分别分装起来，让他们之间可以互相替换，此模式让算法的变化独立与使用算法的客户。这是《Head Fisrst 设计模式》里面的定义，有点拗口不好理解。我的理解是策略模式是让一个类拥有某个接口类的变量，在调用某个函数时，调用接口类变量里的接口函数，当这个类拥有的接口类的变量所指向的具体的实现类不同时可以产生不一样的效果，这个过程中，类调用接口变量的接口函数，并不需要知道接口变量代表的是哪个具体实现，而且也可以改变接口变量所指向的具体实现类来达到不同的效果，可以很好的实现封装效果，也留下了很好的扩展性。

<!-- more -->

# 类图

![策略模式类图](/images/strategyPatternUML.png)

# 例子

此处以吃鸡游戏中的角色和武器为例。假设游戏中有两个玩家：Alice和Bob，有两种武器：枪和刀。Alice和Bob可以拿任何一种武器攻击对方，也可以在中途更换武器。

## 类图

![策略模式例子类图](/images/strategyPatternSampleUML.png)

## Weapon接口类

这个接口类主要是定义行为的，主要是让持有者类通过调用接口方法实现具体行为。此例中就是让Person类持有这个接口类的引用，通过引用调用接口方法实现不同武器的不同攻击效果。

```java
interface Weapon
{
    void attack();
}
```

## Gun实现类

这个类是接口类Weapon的一个实现类，用于模拟枪的攻击效果

```java
class Gun implements Weapon
{
    @Override
    public void attack()
    {
        System.out.println("Gun Attacking...");
    }   
}
```

## Knife实现类

这个类是接口类Weapon的一个实现类，用于模拟刀的攻击效果

```java
class Knife implements Weapon
{
    @Override
    public void attack()
    {
        System.out.println("Knife Attacking...");
    }
}
```

## Person类

Person类拥有Weapon接口，方法attackOthers调用接口对象weapon中的接口函数attack实现攻击别人的目的。changeWeapon函数用于改变weapon的行为。

```java
class Person
{
    private Weapon weapon;
    
    public Person(Weapon weapon)
    {
        this.weapon = weapon;
    }
    
    public void attackOthers()
    {
        weapon.attack();
    }
    
    public void changeWeapon(Weapon weapon)
    {
        this.weapon = weapon;
    }
}
```

## 验证程序

首先，初始化Gun和Knife两个武器类，然后初始化Alice和Bob两个玩家。在初始化Alice时给Alice一把刀，在初始化Bob时给Bob一把枪。

接着，Alice使用自己的武器（刀）攻击Bob。Bob受到攻击后，也使用自己的武器（枪）进行反击。

最后，Alice发现刀打不过枪，决定更换武器枪，然后使用现在的武器（枪）给Bob致命一击，成功吃鸡。

程序如下

```java
public class TestStrategy
{
    public static void main(String[] args)
    {
        // 初始化武器和角色
        System.out.println("----Game Begin!----");
        Weapon gun = new Gun();
        Weapon knife = new Knife();
        System.out.println("----Alice enter the game with a knife----");
        Person alice = new Person(knife);
        System.out.println("----Bob enter the game with a gun----");
        Person bob = new Person(gun);
        
        // Alice用刀偷袭了Bob
        System.out.println("----Alice attack Bob----");
        alice.attackOthers();
        
        // Bob发现有人偷袭，转身给对方一枪
        System.out.println("----Bob attack Alice----");
        bob.attackOthers();
        
        // Alice发现刀的杀伤力不够，更换枪
        System.out.println("----Alice change weapon to gun----");
        alice.changeWeapon(gun);
        
        // Alice更换枪后，给Bob致命一击
        System.out.println("----Alice attack Bob----");
        alice.attackOthers();
        
        System.out.println("----Game Over! Alice Win!----");
    }
}
```

输出如下

![策略模式例子输出结果](/images/strategyPatternSampleOutput.png)

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007