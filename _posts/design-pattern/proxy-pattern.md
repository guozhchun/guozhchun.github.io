---
title: 代理模式
date: 2018-07-14 16:29:03
tags: 设计模式
categories: 设计模式
---

# 定义

> 代理模式为另一个对象提供一个替身或占位符以控制对这个对象的访问。
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Head First 设计模式》

其实代理对象就是一个“中介”的形式，代理对象封装了目标对象，提供跟目标对象相同的接口，目标对象不直接对外暴露接口，一切操作都是代理对象进行调用的。而客户端只会看到代理对象提供的接口。整个调用过程如下：客户端调用代理对象的方法---->代理对象调用目标对象的方法---->目标对象方法执行并将返回值传给代理对象---->代理对象将目标对象的返回值传给客户端。

<!-- more -->

# 类图

![代理模式类图](/images/designPattern/proxyPattern/UML.png)

> 说明：此图参考《Head First 设计模式》画出

# 例子

此处以“代购”iphone手机为例。假设要买一台iphone手机，因为香港比大陆便宜，所以想去香港买，但是自己又没时间，所以找人（代购者）替自己去香港把iphone买回来。这里代购者就是代理对象，商店就是目标对象。原先买家直接跟商店进行交互，商店直接将手机买给买家，现在买家跟代购者进行交互，代购者再跟商店进行交互，商店先将手机卖给代购者，代购者再将手机卖给买家。

由于代理主要分为静态代理和动态代理，所以此处也分别用静态代理和JDK动态代理方式实现此例子。

## 静态代理

### 类图

![代理模式类图](/images/designPattern/proxyPattern/sampleUML.png)

### Subject接口类

此处以SellAction表示Subject接口类，此接口类中只有一个方法，表示卖手机的行为。

```java
interface SellAction
{
    void sell();
}
```

### RealSubject实现类

此处以Store表示RealSubject实现类，实现其中的sell接口方法，表示商店卖出iphone手机。

```java
class Store implements SellAction
{
    @Override
    public void sell()
    {
        System.out.println("Sell a iphone!");
    }
}
```

### Proxy代理类

由于经常在代理类中创建目标对象，所以此处在Proxy中，直接new一个Store对象。当然，为了使程序有更好的扩展性，也可以通过构造函数将目标对象传进来，或者通过set函数更改目标对象。

```java
class Proxy implements SellAction
{
    SellAction target;
    
    public Proxy()
    {
        // 代理对象获得目标对象
        target = new Store();
    }
    
    @Override
    public void sell()
    {
        target.sell();
    }
}
```

### 验证程序

验证程序很简单，直接新建一个代理对象，然后调用代理对象中的方法输出结果。

```java
public class TestProxy
{
    public static void main(String[] args)
    {
        SellAction proxy = new Proxy();
        proxy.sell();
    }
}
```

程序输出如下

```
Sell a iphone!
```

## JDK动态代理

静态代理需要自己实现代理类，JDK动态代理则是有JDK帮忙生产代理类，自己只需要编写一个调用处理器类实现InvocationHandler接口即可。当然，获取代理类的方式也相应的发生改变。静态代理由于是自己实现的代理类，所以可以直接new一个代理类对象，而JDK动态代理由于是JDK根据InvocationHandler接口实现类动态生成代理类及其对象的，所以需要用配套的`Proxy.newProxyInstance`方法类获取代理对象实例。此方法是静态方法，有三个参数：

* `ClassLoader loader`：用于定义代理对象的类加载器，一般情况下与Subject接口类的加载器一致
* `Class<?>[] interfaces`：代理对象所需要实现的接口类（可以是多个），也就是Subject这个接口类
* `InvocationHandler h`：调用处理器，当代理方法调用时，会调用此类中的`invoke`方法，执行目标类中对应的方法。

### Subject接口类

此处与静态代理中的类Subject接口类相同

```java
interface SellAction
{
    void sell();
}
```

### RealSubject实现类

此处与静态代理中的类RealSubject实现类相同

```java
class Store implements SellAction
{
    @Override
    public void sell()
    {
        System.out.println("Sell a iphone!");
    }
}
```

### 调用处理器

调用处理器需要知道目标对象，由于经常在代理类中创建目标对象，所以此处在InvocationHandlerProxy中，直接new一个Store对象。当然，为了使程序有更好的扩展性，也可以通过构造函数将目标对象传进来，或者通过set函数更改目标对象。

```java
class InvocationHandlerProxy implements InvocationHandler
{
    SellAction target;
    
    public InvocationHandlerProxy()
    {
        target = new Store();
    }
    
    // 当代理对象调用代理方法时：proxy.sell()，会调用此方法
    // 此时传入此方法的参数对应关系如下:
    // proxy: 对应代理对象的调用实例，即proxy.sell()中的proxy
    // method：对应调用的代理方法，即proxy.sell()中的sell函数
    // args: 对应调用的代理方法的参数，即proxy.sell()中的sell函数的参数，此处为空
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable
    {
        return method.invoke(target, args);
    }
}
```

### 验证程序

在验证程序中，需要让JDK动态创建代理类，所以先new了一个调用处理器类，然后使用`Proxy.newProxyInstance`函数动态创建代理类并获取对象实例，最后调用代理类中的方法。

```java
public class TestProxy
{
    public static void main(String[] args)
    {
        InvocationHandlerProxy handlerProxy = new InvocationHandlerProxy();
        SellAction proxy = (SellAction)Proxy.newProxyInstance(SellAction.class.getClassLoader(), new Class[]{SellAction.class}, handlerProxy);
        proxy.sell();
    }
}
```

程序输出如下

```
Sell a iphone!
```

## 比较

从两个例子的输出结果来看，JDK动态代理与静态代理的效果是一样的，都是对目标对象的封装，让客户端通过代理对象间接的与目标对象进行通信。但是JDK动态代理不需要自己实现代理类，但是需要实现调用处理器，从上面的例子中看，甚至JDK动态代理比静态代理还要麻烦。但是这是一个接口方法的情况，如果目标对象有多个接口方法需要代理，那静态代理需要实现每个代理方法，而JDK动态代理的代码几乎不用改变，因为调用处理器类不用改变，而代理类是由JDK动态生成的，当增加方法时，动态生成的代理类也会包含此方法，并不需要人为干预。

# 与其他模式比较

| 代理模式                                                     | 装饰者模式                                                   | 适配器模式                                                   | 策略模式                                                     | 命令模式                                                     | 外观模式                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 封装目标对象，提供相同的接口供外部调用，一般不在代理方法中增加额外功能。可以提供访问控制权限等。接口函数名一定与目标对象相同 | 封装目标对象，为目标对象增加额外的功能，接口函数名一般与目标对象相同 | 封装目标对象，将外部对象想要的接口转换成目标对象的接口，接口函数名一般与外部对象相同，与目标对象不同 | 封装目标对象，将调用者的行为委托给目标对象进行，接口函数名一般与目标对象不同 | 封装目标对象，将调用者的行为委托给命令对象，再由命令对象委托给目标对象进行，命令对象提供统一的接口函数名，一般与目标对象不同 | 封装目标对象，将一群复杂的接口简化为简单的接口供外部调用，接口函数名称一般与目标对象不同 |

# 扩展阅读

1. Java的三种代理模式：[https://www.cnblogs.com/cenyu/p/6289209.html](https://www.cnblogs.com/cenyu/p/6289209.html)
2. Java设计模式——代理模式实现及原理：[https://blog.csdn.net/goskalrie/article/details/52458773](https://blog.csdn.net/goskalrie/article/details/52458773)

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007