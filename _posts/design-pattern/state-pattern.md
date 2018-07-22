---
title: 状态模式
date: 2018-07-09 22:03:36
tags: 设计模式
categories: 设计模式
---

# 定义

> 状态模式允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它的类。
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;——《Head First 设计模式》

这个定义看起来晦涩难懂，其实就是把有限状态自动机中各个状态定义由一个基本数据类型变成状态类，再把对外提供的改变状态的方法中的一堆 if 条件分解到状态类中实现。

<!-- more -->

# 类图

![状态模式类图](/images/designPattern/statePattern/UML.png)

> 说明：此图参考《Head First 设计模式》画出

# 例子

此处以枪的状态转移为例子进行说明。假设枪有两个状态：已装弹、未装弹，初始化时枪是未装弹的，枪只有在未装弹的情况下才能完成进行装弹，从而进入到已装弹状态，枪只有在已装弹的情况下才能开枪从而进入到未装弹的情况。其状态转移图如下所示

![状态模式例子状态图](/images/designPattern/statePattern/state.png)

为了更加清楚地对比有限状态自动机到状态模式之间的转换，本文用有限状态机的方式实现此例，再用状态模式实现此例。

## 有限状态自动机

### Gun类

Gun类作为状态机，在对外暴露的接口`loadBullet`、`fire`中都对状态进行了判断，因此，存在这大量的 if 判断代码

```java
class Gun
{
    /**
     * 枪的两个状态
     * UNLOAD: 未装弹
     * LOADED: 已装弹
     * 
     * 状态转移：UNLOAD（loadBullet）-->LOADED(fire)-->UNLOAD
     */
    private static final int UNLOAD = 0;
    private static final int LOADED = 1;
    
    // 对象变量，用于记录对象的状态
    private int state = UNLOAD;
    
    public void loadBullet()
    {
        if (state == UNLOAD)
        {
            System.out.println("Load bullets finish!");
            state = LOADED;
        }
        else if (state == LOADED)
        {
            System.out.println("You have loaded the bullets.Don't need to load again!");
        }
        else 
        {
            System.out.println("Unknow state error!");
        }
    }
    
    public void fire()
    {
        if (state == UNLOAD)
        {
            System.out.println("You can't fire before the bullets are loaded!");
        }
        else if (state == LOADED)
        {
            System.out.println("Fire!");
            state = UNLOAD;
        }
        else 
        {
            System.out.println("Unknow state error!");
        }
    }
}
```

### 验证程序

验证程序很简单，先实例化一个状态机（Gun），然后模拟状态机正常的走向和异常的走向

```java
public class TestState
{
    public static void main(String[] args)
    {
        Gun gun = new Gun();
        
        // 正常情况
        System.out.println("----normal state----");
        gun.loadBullet();
        gun.fire();
        
        // 不正常情况
        System.out.println("----abnormal state----");
        gun.fire();
        gun.loadBullet();
        gun.loadBullet();
    }
}
```

输出如下

![状态模式-有限状态自动机例子输出结果](/images/designPattern/statePattern/sampleStateOutput.png)

## 状态模式

### 状态接口类

状态接口类定义了状态之间转移的动作。虽然类图中状态接口类是用抽象类表示的，但是在具体实现中也可以用接口实现。此处就是用接口进行实现。

```java
interface State
{
    void loadBullet();
    void fire();
}
```

### 状态类

本例中有两个状态（已装弹和未装弹），所以有两个状态类，分别是Unload（表示未装弹）、Loaded（表示已装弹）。这两个状态类都实现状态接口类中的接口，由于具体状态类对应哪个状态是明确的，所以在相应的接口中不需要再次判断状态信息，只需要执行对应的动作即可。

```java
class Unload implements State
{
    private Gun gun;
    
    public Unload(Gun gun)
    {
        this.gun = gun;
    }
    
    @Override
    public void loadBullet()
    {
        System.out.println("Load bullets finish!");
        // 由于gun中已经有明确的状态定义，所以此处直接调用即可
        // 如果gun中没有各个状态的明确定义，此处需要new一个新的下一个状态类对象
        // 如gun.setState(new Loaded(gun))
        gun.setState(gun.getLoadedState());
    }
    
    @Override
    public void fire()
    {
        System.out.println("You can't fire before the bullets are loaded!");
    }
}

class Loaded implements State
{
    private Gun gun;
    
    public Loaded(Gun gun)
    {
        this.gun = gun;
    }
    
    @Override
    public void loadBullet()
    {
        System.out.println("You have loaded the bullets.Don't need to load again!");
    }

    @Override
    public void fire()
    {
        System.out.println("Fire!");
        // 由于gun中已经有明确的状态定义，所以此处直接调用即可
        // 如果gun中没有各个状态的明确定义，此处需要new一个新的下一个状态类对象
        // 如gun.setState(new Unload(gun))
        gun.setState(gun.getUnloadState());
    }
}
```

### Gun类

由于使用了状态类，Gun类中暴露的接口实现中已经不需要大段的 if 判断来获取对应的状态从而执行对应的动作了，直接委托给状态类执行即可。而状态类在不同的时刻表示的是不同的对象，且由于有共同的接口，所以直接调用状态类对应的接口并不需要关心此时的具体状态，这一动作交给具体的状态类执行。

```java
class Gun
{   
    /**
     * 枪的两个状态
     * UNLOAD: 未装弹
     * LOADED: 已装弹
     * 状态转移：UNLOAD（loadBullet）-->LOADED(fire)-->UNLOAD
     * 
     * 此处之所以用两个变量来表示两个状态，是为了在状态类中变化状态的过程中可以直接获取到下一个状态；
     * 当然，此处也可以不写，但是在状态类变化状态的过程中需要new一个新的下一个状态的状态类，
     * 如果是这样，当状态多次转移时，会产生多个实例对象，造成不必要的内存浪费
     */
    private State loaded = new Loaded(this);
    private State unload = new Unload(this);
    
    // 对象变量，用于记录对象的状态
    private State state = new Unload(this);
    
    // 暴露给外部调用的接口，直接委托为对象状态执行，此处不必再写一堆 if 条件判断进行状态转移
    public void loadBullet()
    {
        state.loadBullet();
    }
    
    // 暴露给外部调用的接口，直接委托为对象状态执行，此处不必再写一堆 if 条件判断进行状态转移
    public void fire()
    {
        state.fire();
    }
    
    // 暴露给状态类，用于改变状态
    public void setState(State state)
    {
        this.state = state;
    }
    
    // 暴露给状态类，用于获取Load状态类
    public State getLoadedState()
    {
        return loaded;
    }
    
    // 暴露给状态类，用于获取unload状态类
    public State getUnloadState()
    {
        return unload;
    }
}
```

### 验证程序

验证程序同有限状态自动机中的验证程序相同

```java
public class TestState
{
    public static void main(String[] args)
    {
        Gun gun = new Gun();
        
        // 正常情况
        System.out.println("----normal state----");
        gun.loadBullet();
        gun.fire();
        
        // 不正常情况
        System.out.println("----abnormal state----");
        gun.fire();
        gun.loadBullet();
        gun.loadBullet();
    }
}
```

输出如下

![状态模式例子输出结果](/images/designPattern/statePattern/samplePatternOutput.png)

对比两种实现方式的输出结果，可以看出效果是一样的。但是从实现方式而言，有限状态自动机实现比较简单，也容易理解，但是其内部存在大量的 if 条件判断，而且如果增加新的状态也需要改动多处代码，不易于扩展。而状态模式则有良好的扩展性，增加一个新状态时，只需要新增一个状态实现类，再在其他的状态实现类总对应的地方加上状态转移即可。

# 其他

状态模式和策略模式有点类似，都是对行为进行封装，将动作委托给其他类进行，对比两者的类图，可以看到两者几乎相同。但是策略模式注重的是算法的可替换性，替换算法时无需改动原有其他类的实现逻辑，但是调用者需要明确知道需要替换的算法以及进行替换的操作，换句话说，调用者知道所有可用的算法列表，甚至可能需要维护这些算法列表。而状态模式注重的是状态之间的切换，这之间的切换对调用者是无感知的，甚至调用者都不知道具体有哪些状态，当然也不用维护这些状态。

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007