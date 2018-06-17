---
title: 装饰者模式
date: 2018-06-17 17:06:57
tags: 设计模式
categories: 设计模式
---

# 定义

> 装饰者模式动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更有弹性的替代方法。
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Head First 设计模式》

也就是说，装饰者模式是对原有功能类增加一些新的功能（行为）。但是为原有功能类增加新的功能行为，继承或组合也能做法，为什么要使用装饰者模式呢？这是因为要对原有功能类动态地同时增加多个功能行为（包括相同行为或不同行为），单独使用继承或组合实现将产生大量的子类（“类爆炸”），而使用装饰者模式则可以在保持良好的扩展性的同时避免“类爆炸”情况的出现。

<!-- more -->

# 特点

* 装饰者和被装饰者继承同一个基类。因为装饰者必须能够取代被装饰者，这里利用继承达到“类型匹配”，而不是利用继承获取“行为”
* 装饰者拥有(has a)一个与被装饰者相同的基类类型属性
* 可以用一个或多个装饰者类包装同一个对象
* 装饰者可以在所委托被装饰者的行为之前或行为之后或行为之前与之后加上自己的行为，以达到特定的目的

# 类图

![装饰者模式类图](/images/decoratorPatternUML.png)

> 说明：此图来源于《Head First 设计模式》

为什么装饰者类和被装饰者需要继承同一个基础类呢？主要有两个方面的原因。第一，装饰者必须能够取代被装饰者；第二，如果需要多个装饰对象对被装饰对象进行装饰（假设A是被装饰者，B和C用来装饰A），如果不使用继承，则先用B装饰A再用C装饰B时，需要B拥有A的对象，C拥有B的对象；而先用C装饰A再用B装饰C时，需要C拥有A的对象，B拥有C的对象，这使得装饰的顺序依赖于类的实现。而如果使用了继承，则只需要B和C同时拥有A的对象，然后在使用的地方用调用的顺序来完成装饰的顺序，这样不用破坏原有的类，有良好的扩展性。

为什么装饰者需要拥有(has a)一个和被装饰者相同的基础类类型作为自己属性呢？主要是考虑多个不同的被装饰者的情况，如果使用继承，则装饰者必须继承自被装饰者，此时只能装饰一个类，而使用组合，则装饰者可以同时装饰多个类。这里也算是应用了策略模式吧。

# 例子

此处以射击类游戏为例。假设游戏中有Alice和Bob两名游戏玩家，有刀和枪两种武器，一个游戏玩家可以装备相同或不同的武器，进而攻击其他人。

## 类图

![装饰者模式例子类图](/images/decoratorPatternSampleUML.png)

## 抽象基类

Person类是所有类的基类，主要提供两个抽象方法，供子类具体实现。当然，这个抽象类也可以用接口表示，但是装饰者模式中一般用抽象类当基类，所以此处也用抽象基类表示。

```java
abstract class Person
{
    protected abstract String getDescription();   // 描述装备信息
    protected abstract void attack();             // 攻击行为
}
```

## 角色类

有两个角色，Alice和Bob，分别继承抽象基类Person，实现自己的独特行为

```java
class Alice extends Person
{
    public String getDescription()
    {
        return "Alice: ";
    }
    
    public void attack()
    {
        System.out.print("Alice attacking with: ");
    }
}

class Bob extends Person
{
    public String getDescription()
    {
        return "Bob: ";
    }
    
    public void attack()
    {
        System.out.print("Bob attacking with: ");
    }
}
```

## 装饰者基类

WeaponDecorator是装饰者基类，其他的装饰者类继承此类。此类拥有Person类作为一个属性，这样就能对具体的person子类进行额外的封装， 加上额外的功能，以达到目的。

当然，也可以不使用此类，直接让装饰者子类继承自Person，并在每个装饰者子类中独自维护Person属性，这样也能达到同样的目的。

```java
class WeaponDecorator extends Person
{
    protected Person person;
    
    public WeaponDecorator(Person person)
    {
        this.person = person;
    }
    
    public String getDescription()
    {
        return person.getDescription() + "[no weapons!] ";
    }
    
    public void attack()
    {
        person.attack();
        System.out.print("[bare arms]");
    }
}
```

## 武器类

武器类有两个，Gun和Knife，这两个类都继承了装饰者基类WeaponDecorator，并使用从WeaponDecorator继承的person属性得到要装饰的具体对象，从而获取要装饰的具体对象的具体行为，并在此基础上增加额外的操作达到扩展不同功能的目的。

当然，也可以不继承装饰者基类WeaponDecorator，直接继承自Person类，并在各自的类中维护person属性，这样也能得到同样的效果。

```java
class Gun extends WeaponDecorator
{
    public Gun(Person person)
    {
        super(person);
    }
    
    public String getDescription()
    {
        return person.getDescription() + "[gun] ";
    }
    
    public void attack()
    {
        person.attack();
        System.out.print("[gun] ");
    }
}

class Knife extends WeaponDecorator
{
    public Knife(Person person)
    {
        super(person);
    }
    
    public String getDescription()
    {
        return person.getDescription() + "[knife] ";
    }
    
    public void attack()
    {
        person.attack();
        System.out.print("[knife] ");
    }
}
```

## 验证程序

```java
public class TestDecorator
{
    public static void main(String[] args)
    {
        // 初始化角色
        System.out.println("----Game Begin!----");
        Person alice = new Alice();
        Person bob = new Bob();
        
        // Alice装备了一把刀
        System.out.println("----Alice take a knife----");
        alice = new Knife(alice);
        System.out.println(alice.getDescription());
        
        // Bob装备了一支枪
        System.out.println("----Bob take a gun----");
        bob = new Gun(bob);
        System.out.println(bob.getDescription());
        
        // Alice用现有的装备攻击了Bob
        alice.attack();
        System.out.println();
        
        // Bob用现有装备对Alice进行反击
        bob.attack();
        System.out.println();
        
        // Alice发现地上有一把枪，拿起枪，和刀同时攻击Bob
        System.out.println("----Alice take a gun----");
        alice = new Gun(alice);
        System.out.println(alice.getDescription());
        alice.attack();
        System.out.println();
        
        // Bob发现地上还有一把枪，也拿起了枪，用两把枪同时攻击Alice，给Alice致命一击
        System.out.println("----Bob take a gun----");
        bob = new Gun(bob);
        System.out.println(bob.getDescription());
        bob.attack();
        System.out.println();
        
        System.out.println("----Game Over! Bob Win!----");
    }
}
```

输出如下

![装饰者模式例子输出结果](/images/decoratorPatternSampleOutput.png)

# 其他

装饰者模式在避免“类爆炸”的同时保持扩展性，但是也有可能产生多个小的装饰者类，在对被装饰者运用多个装饰者对象进行连续组合时，不能很清晰地看出这是运用了装饰者模式。即便装饰者模式存在者一些不足，但是其仍然是软件设计开发中的一种良好的实践。java类库中的IO流也使用了装饰者模式进行流功能的扩展，如下图所示

![java类库中IO流的装饰者模式](/images/decoratorPatternIOUML.png)

> 说明：此图来源于《Head First 设计模式》

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007