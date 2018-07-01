---
title: 工厂模式
date: 2018-06-18 15:10:15
tags: 设计模式
categories: 设计模式
---

# 概述

工厂模式主要是将创建对象这一个步骤进行封装，从而将客户程序从具体类中解耦。工厂模式主要有三种：简单工厂、工厂方法模式、抽象工程模式。

<!-- more -->

# 简单工厂

严格来说，简单工厂并不是真正的设计模式，但是由于其也封装了对象的创建，解耦了客户程序与具体实现类，且日常中也将此种情况作为工厂模式的一种，因此这里也将其归类为工厂模式并一起介绍比较。

## 类图

![简单工厂类图](/images/designPattern/factoryPattern/simpleFactoryUML.png)

## 例子

此处以生产服装为例，假设有两种类型的服装，一种是衬衣，一种是裤子。服装的工厂类根据接收的参数判断是生产一件衬衣还是生产一件裤子。

### 类图

![简单工厂例子类图](/images/designPattern/factoryPattern/simpleFactorySampleUML.png)

### Clothes抽象类

Clothes抽象类主要是定义了两个常量用于判断服装的类型，一个私有变量用于保存服装的类型，一个protected变量用于在子类中对其进行修改，同时定义了一个抽象方法用于子类的重写，最后重写了toString方法输出类的信息。

```java
abstract class Clothes
{
    public static final String CLOTHES_TYPE_TROUSERS = "trousers";
    public static final String CLOTHES_TYPE_SHIRT = "shirt";
    
    private String type;
    protected int price;
    
    protected abstract void initPrice();
    
    public Clothes(String type)
    {
        this.type = type;
    }

    @Override
    public String toString()
    {
        return "Clothes [type=" + type + ", price=" + price + "]";
    }
}
```

### Clothes子类

Clothes有两个子类，一个用于表示裤子的Trousers，另一个用于表示衬衣的Shirt。在子类的构造函数中，调用了父类构造函数对type变量进行赋值，然后调用重写过后的initPrice方法对price进行赋值，从而达到不同子类属性不同的效果。

```java
class Trousers extends Clothes
{
    public Trousers()
    {
        super(Clothes.CLOTHES_TYPE_TROUSERS);
        initPrice();
    }
    
    @Override
    public void initPrice()
    {
        price = 200;
    }
}

class Shirt extends Clothes
{
    public Shirt()
    {
        super(Clothes.CLOTHES_TYPE_SHIRT);
        initPrice();
    }
    
    @Override
    public void initPrice()
    {
        price = 100;
    }
}
```

### Clothes工厂类

Clothes工厂类只有一个方法`makeClothes`，这个方法接收一个参数，用于判断生产服装的类型。

```java
class ClothesFactory
{
    public Clothes makeClothes(String type)
    {
        // 如果新增了服装类型，需要在此增加条件判断分支
        if (Clothes.CLOTHES_TYPE_SHIRT.equals(type))
        {
            return new Shirt();
        }
        else if (Clothes.CLOTHES_TYPE_TROUSERS.equals(type))
        {
            return new Trousers();
        }
        else 
        {
            return null;
        }
    }
}
```

### 验证程序

首先，获取一个工厂类，然后用工厂类生产一件衬衣，再用工厂类生产一件裤子。

```java
public class TestFactory
{
    public static void main(String[] args)
    {
        // 如果要获取新增的服装类型，只需要将新的参数值传给makeClothes方法，其他保持不变
        System.out.println("----init clothesFactory----");
        ClothesFactory clothesFactory = new ClothesFactory();
        
        System.out.println("----making a shirt----");
        Clothes shirt = clothesFactory.makeClothes(Clothes.CLOTHES_TYPE_SHIRT);
        System.out.println(shirt);
        
        System.out.println("----making a trousers----");
        Clothes trousers = clothesFactory.makeClothes(Clothes.CLOTHES_TYPE_TROUSERS);
        System.out.println(trousers);
    }
}
```

输出如下

![简单工厂例子输出结果](/images/designPattern/factoryPattern/simpleFactorySampleOutput.png)

# 工厂方法模式

## 定义

> 工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法让类将实例化推迟到子类。
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Head First 设计模式》

相比简单工厂，工厂方法模式比简单工厂多了一个工厂抽象类，在工厂抽象类中定义了创建工厂的方法，在工厂子类中实现创建工厂的方法，在实现创建工厂的方法里面再实现创建具体对象的功能。

## 类图

![工厂方法模式类图](/images/designPattern/factoryPattern/factoryMethodUML.png)

> 说明：此图来源于《Head First 设计模式》

## 例子

此处仍以简单工厂中的例子进行说明，只不过生产服装的工厂定义成一个抽象的类，再增加生产衬衫的工厂子类和生产裤子的工厂子类。

### 类图

![工厂方法模式例子类图](/images/designPattern/factoryPattern/factoryMethodSampleUML.png)

### Clothes抽象类

此类与简单工厂中的例子一样

```java
abstract class Clothes
{
    public static final String CLOTHES_TYPE_TROUSERS = "trousers";
    public static final String CLOTHES_TYPE_SHIRT = "shirt";
    
    private String type;
    protected int price;
    
    protected abstract void initPrice();
    
    public Clothes(String type)
    {
        this.type = type;
    }

    @Override
    public String toString()
    {
        return "Clothes [type=" + type + ", price=" + price + "]";
    }
}
```

### Clothes子类

这两类与简单工厂中的例子一样

```java
class Trousers extends Clothes
{
    public Trousers()
    {
        super(Clothes.CLOTHES_TYPE_TROUSERS);
        initPrice();
    }
    
    @Override
    public void initPrice()
    {
        price = 200;
    }
}

class Shirt extends Clothes
{
    public Shirt()
    {
        super(Clothes.CLOTHES_TYPE_SHIRT);
        initPrice();
    }
    
    @Override
    public void initPrice()
    {
        price = 100;
    }
}
```

### Clothes工厂抽象类

与简单工厂的例子不同，Clothes工厂是一个抽象类，而不是一个具体的实现类，此抽象类中只有一个抽象方法用于表示创建一件服装，而具体的实现放在工厂子类中。

```java
// 当然，也可以使用interface
abstract class ClothesFactory
{
    // 如果新增了服装类型，需要增加新的子类工厂实现
    protected abstract Clothes makeClothes();
}
```

### Clothes工厂子类

ShirtFactory和TrousersFactory是ClothesFactory的子类，分别实现工厂抽象方法makeClothes产生不同类型的服装。

```java
class ShirtFactory extends ClothesFactory
{
    @Override
    public Clothes makeClothes()
    {
        return new Shirt();
    }
}

class TrousersFactory extends ClothesFactory
{
    @Override
    public Clothes makeClothes()
    {
        return new Trousers();
    }
}
```

### 验证程序

首先，获取衬衣工厂类，然后用衬衣工厂类生产一件衬衣；然后获取裤子工厂类，再用裤子工厂类生产一件裤子。

```java
public class TestFactory
{
    public static void main(String[] args)
    {
        // 如果要获取新的衣服类型，只需要创建新衣服类型的工厂，其他保持不变
        System.out.println("----init shirtFactory----");
        ClothesFactory shirtFactory = new ShirtFactory();
        System.out.println("---making a shirt----");
        Clothes shirt = shirtFactory.makeClothes();
        System.out.println(shirt);
        
        System.out.println("----init trousersFactory----");
        ClothesFactory trousersFactory = new TrousersFactory();
        System.out.println("----making a trousers----");
        Clothes trousers = trousersFactory.makeClothes();
        System.out.println(trousers);
    }
}
```

输出如下

![工厂方法模式例子输出结果](/images/designPattern/factoryPattern/factoryMethodSampleOutput.png)

# 抽象工厂模式

## 定义

> 抽象工厂模式提供一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类。
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Head First 设计模式》

与工厂方法不同的时，抽象工厂模式提供的是一个接口，里面包含了一组（多个）接口方法用于表示创建具体对象需要的一组方法（该对象需要由多个子对象组合而成），实现类需要同时实现这一组方法才能创建出具体的对象

## 类图

![抽象工厂模式类图](/images/designPattern/factoryPattern/abstractFactoryUML.png)

> 说明：此图来源于《Head First 设计模式》

## 例子

此处仍以生产服装为例，只是此时生产一件服装后会附加赠送一件礼物，衬衣赠送领带，裤子赠送皮带，只有将服装和礼物同时弄好后才算一件服装商品真正完成。

### 类图

![抽象工厂模式例子类图](/images/designPattern/factoryPattern/abstractFactorySampleUML.png)

### Clothes抽象类

此类与前面简单工厂和工厂方法模式的例子相同

```java
abstract class Clothes
{
    public static final String CLOTHES_TYPE_TROUSERS = "trousers";
    public static final String CLOTHES_TYPE_SHIRT = "shirt";
    
    private String type;
    protected int price;
    
    protected abstract void initPrice();
    
    public Clothes(String type)
    {
        this.type = type;
    }

    @Override
    public String toString()
    {
        return "Clothes [type=" + type + ", price=" + price + "]";
    }
}
```

### Clouthes子类

这两个类与前面简单工厂和工厂方法模式的例子相同

```java
class Trousers extends Clothes
{
    public Trousers()
    {
        super(Clothes.CLOTHES_TYPE_TROUSERS);
        initPrice();
    }
    
    @Override
    public void initPrice()
    {
        price = 200;
    }
}

class Shirt extends Clothes
{
    public Shirt()
    {
        super(Clothes.CLOTHES_TYPE_SHIRT);
        initPrice();
    }
    
    @Override
    public void initPrice()
    {
        price = 100;
    }
}
```

### Gift抽象类

这个类是本例新增的，用于表示随服装赠送的礼物。此类只有一个抽象方法，用于描述礼物，同时重写`toString`方法用于描述具体的礼物信息。

```java
abstract class Gift
{
    protected abstract String getDescription();

    @Override
    public String toString()
    {
        return "Gift [" + getDescription() + "]";
    }
}
```

### Gift子类

这两个子类也是本例新增的。Gift有两个子类，一个用于表示皮带的Belt，另一个用于表示领带的Tie。在子类中，分别实现了父类的抽象方法，描述自己的独特信息，供父类`toString`方法调用，从而输出具体的礼物信息。

```java
class Belt extends Gift
{
    @Override
    public String getDescription()
    {
        return "Belt";
    }
}

class Tie extends Gift
{
    @Override
    public String getDescription()
    {
        return "Tie";
    }
}
```

### Clothes工厂抽象类

与工厂方法模式相比，此例的工厂抽象类新增了`makeGift`抽象方法，因此此时需要在生产服装后也生产礼物，只有当这两者完成时才算一件服装商品完成。另外，也把`ClothesFactory`由抽象类变成了接口的方式（当然，interface也可以用工厂方法模式的abstract方式替换）。

```java
// 当然，此处也可以用抽象类
interface ClothesFactory
{
    // 如果新增了服装类型，需要增加新的子类工厂实现
    Clothes makeClothes();
    Gift makeGift();
}
```

### Clothes工厂子类

与工厂方法模式相比，这两个类只是新增了实现ClothesFactory的新抽象方法，其他并无差异

```java
class ShirtFactory implements ClothesFactory
{
    @Override
    public Clothes makeClothes()
    {
        return new Shirt();
    }

    @Override
    public Gift makeGift()
    {
        return new Tie();
    }
}

class TrousersFactory implements ClothesFactory
{
    @Override
    public Clothes makeClothes()
    {
        return new Trousers();
    }

    @Override
    public Gift makeGift()
    {
        return new Belt();
    }
}
```

### 验证程序

首先，获取一个衬衣工厂类，用衬衣工厂类生产一件衬衣及对应的礼物（领带）；然后获取一个裤子工厂类，用裤子工厂类生产一件裤子及对应的礼物（皮带）。

```java
public class TestFactory
{
    public static void main(String[] args)
    {
        // 如果要获取新的服装类型，只需要创建新服装类型的工厂，其他保持不变
        System.out.println("----init shirtFactory----");
        ClothesFactory shirtFactory = new ShirtFactory();
        System.out.println("----making a shirt and giving a tie as present----");
        System.out.println(shirtFactory.makeClothes());
        System.out.println(shirtFactory.makeGift());
        
        System.out.println("----init trousersFactory----");
        ClothesFactory trousersFactory = new TrousersFactory();
        System.out.println("----making a trousers and giving a belt as present----");
        System.out.println(trousersFactory.makeClothes());
        System.out.println(trousersFactory.makeGift());
    }
}
```

输出如下

![抽象工厂模式例子输出结果](/images/designPattern/factoryPattern/abstractFactorySampleOutput.png)

# 比较

| 简单工厂                                 | 工厂方法模式                                     | 抽象工厂模式                                                 |
| ---------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| 易于实现                                 | 实现起来相对繁琐                                 | 实现起来相对繁琐                                             |
| 不易扩展，需要在工厂类中增加条件判断语句 | 易于扩展，只需要继承抽象工厂，新实现工厂子类即可 | 易于扩展，只需要新建工厂子类，实现接口方法即可               |
| 无继承、无组合                           | 使用继承，单一接口                               | 使用组合，多个接口，将多个对象（对象家族）创建放在同一个接口中。对每个单独的接口，使用工厂方法模式实现 |

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007