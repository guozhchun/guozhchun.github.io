---
title: 观察者模式
date: 2018-06-02 10:49:02
tags: 设计模式
categories: 设计模式
---

# 定义 

观察者模式定义了对象之间的一对多依赖，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。观察者模式主要有两个对象，主题（被观察者）和订阅者（观察者），这是一个一对多的关系，同一个主题可以有多个订阅者，当主题发生改变时，每个订阅者都能收到消息通知并执行对应的逻辑。

<!-- more -->

对于主题（被观察者）而言，需要提供注册、去注册、通知观察者的功能。

对于订阅者（观察者）而言，需要提供统一的接口让主题在通知时调用。

# 类图

![观察者设计模式类图](/images/observerPatternUML.png)

> 说明：此图来源于《Head First 设计模式》

# 例子

此处以订阅报纸为例，分别用自写的观察者模式和 java 内置的观察者模式实现。

假设有一份报纸Paper，有三个订阅者Tom、Amy和Alice。Tom和Amy一开始就订阅了报纸，并在报纸更新时分别输出自己获取到报纸的时间和报纸的标题。Alice在后面加入订阅报纸行列，在接收一次报纸更新后退出订阅。

## 自己实现观察者模式

### 定义观察者接口

此接口只有一个函数，用于主题更新通知观察者

```java
interface Observer
{
    void update(String title, String content);
}
```

### 定义主题接口

此接口主要有三个函数，一个用于注册增加观察者，一个用户退订移除观察者，一个用户改变时通知所有观察者

```java
interface Subject
{
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObserver();
}
```

### 实现Paper类

Paper类作为被观察的对象，需要实现主题接口

```java
class Paper implements Subject
{
    private String title;
    private String content;
    private List<Observer> observers = new ArrayList<>();
    
    // 产生新报纸，通知所有订阅者取阅
    public void makeNewPaper(String title, String content)
    {
        this.title = title;
        this.content = content;
        notifyObserver();
    }
    
    // 将订阅者加入队列中，如果已经存在则不加入
    @Override
    public void registerObserver(Observer observer)
    {
        if (!observers.contains(observer))
        {
            observers.add(observer);
        }
    }

    // 移除订阅者
    @Override
    public void removeObserver(Observer observer)
    {
        observers.remove(observer);
    }

    // 通知所有订阅者取阅报纸
    @Override
    public void notifyObserver()
    {
        for (Observer observer : observers)
        {
            observer.update(title, content);
        }
    }
    
}
```

### 实现订阅者类

订阅者（Tom、Amy、Alice）需要实现Observer接口中的update函数，以便报纸更新时能获取到通知

```java
class Tom implements Observer
{
    @Override
    public void update(String title, String content)
    {
        System.out.print("I'm Tom. I receive a paper");
        System.out.print("[title: " + title);
        System.out.print(", content: " + content + "]");
        System.out.print(" at " + new Date() + "\n");
    }
}

class Amy implements Observer
{
    @Override
    public void update(String title, String content)
    {
        System.out.print("I'm Amy. I receive a paper");
        System.out.print("[title: " + title);
        System.out.print(", content: " + content + "]");
        System.out.print(" at " + new Date() + "\n");
    }
}

class Alice implements Observer
{
    @Override
    public void update(String title, String content)
    {
        System.out.print("I'm Alice. I just try to subscribe a paper, and I receive a paper");
        System.out.print("[title: " + title);
        System.out.print(", content: " + content + "]");
        System.out.print(" at " + new Date() + "\n");
    }
}
```

### 验证程序

首先，先让Tom和Amy订阅报纸，然后生产一份报纸，此时只有Tom和Amy收到了报纸，而Alice没有收到报纸。

然后，Alice觉得报纸也挺有趣的，决定订阅报纸，于是Alice加入了观察者队列中，这样生产第二份报纸时，Tom、Amy、Alice都能收到报纸。

接着，Alice觉得报纸不符合自己的兴趣，决定退订报纸，于是Alice从观察者队列中移除出来，这样生产第三份报纸时，Tom、Amy能收到报纸，而Alice已经收不到报纸了。

程序如下

```java
public class TestObserver
{
    public static void main(String[] args)
    {
        // 初始化
        Paper paper = new Paper();
        Tom tom = new Tom();
        Amy amy = new Amy();
        Alice alice = new Alice();
        
        // Tom和Amy订阅Paper
        System.out.println("----Tom subscribe the paper----");
        paper.registerObserver(tom);
        System.out.println("----Amy subscribe the paper----");
        paper.registerObserver(amy);
        
        // 生产一份报纸，Tom和Amy收到报纸, Alice没订阅报纸，没收到通知
        System.out.println("----make first paper: Tom and Amy will receive，but Alice not----");
        paper.makeNewPaper("Fisrt Paper", "Hello World!");
        
        // Alice觉得报纸不错，要订阅报纸
        System.out.println("----Alice subscribe the paper----");
        paper.registerObserver(alice);
        paper.registerObserver(alice);
        paper.registerObserver(alice);
        
        // 生产第二份报纸，Tom、Amy、Alice都收到报纸
        System.out.println("----make secode paper: Tom、Amy and Alice will receive----");
        paper.makeNewPaper("Second Paper", "Weclome Alice to join us!");
        
        // Alice觉得报纸不适合自己，决定退订
        System.out.println("----Alice unsubscribe the paper----");
        paper.removeObserver(alice);
        
        // 生产第三份报纸，Tom和Amy能收到，而Alice已经收不到了
        System.out.println("----make third paper: Tom and Amy will receive，but Alice not----");
        paper.makeNewPaper("Third Paper", "Sorry for Alice leaving!");
    }
}
```

输出如下

![观察者模式程序输出结果](/images/observerPatternOutput1.png)

## 利用 java 内置观察者模式

java中已经提供了Observable类和Observer接口实现观察者模式。只要被观察者对象继承Observable，观察者实现Observer接口即可。

### 实现Paper

使用java内置的观察者模式时，在被观察者类中需要引入`java.util.Observable`类。

```java
import java.util.Observable;

class Paper extends Observable
{
    private String title;
    private String content;
    
    // 产生新报纸，通知所有订阅者取阅
    public void makeNewPaper(String title, String content)
    {
        this.title = title;
        this.content = content;

        // 通知观察者，这两个函数都是Observable类中的函数
        // setChanged必须调用，否则直接调用notifyObservers并不会通知观察者
        setChanged();
        notifyObservers();
    }

    // 以下函数主要是方便观察者获取相关数据
    public String getTitle()
    {
        return title;
    }

    public String getContent()
    {
        return content;
    }
}
```

需要注意的是：在通知观察者时，需要先调用`setChanged`函数，再调用`notifyObservers`函数，否则直接调用`notifyObservers`函数将不会通知观察者。如下是`java.util.Observable`类中的部分源码，从中可以证实以上结论。

```java
protected synchronized void setChanged() {
    changed = true;
}

public void notifyObservers() {
    notifyObservers(null);
}

public void notifyObservers(Object arg) {
    Object[] arrLocal;
    synchronized (this) {
        if (!changed)  // 如果状态未改变，则直接返回，不会通知观察者 
            return;
        arrLocal = obs.toArray();
        clearChanged();
    }

    for (int i = arrLocal.length-1; i>=0; i--)
        ((Observer)arrLocal[i]).update(this, arg);
}
```

### 实现订阅者类

使用java内置的观察者模式时，在观察者类中需要引入`java.util.Observer`类。

```java
import java.util.Observer;

class Tom implements Observer
{
    // 被观察者状态更新，调用此函数
    @Override
    public void update(Observable observable, Object obj)
    {
        System.out.print("I'm Tom. I receive a paper");
        System.out.print("[title: " + ((Paper)observable).getTitle());
        System.out.print(", content: " + ((Paper)observable).getContent() + "]");
        System.out.print(" at " + new Date() + "\n");
    }
}

class Amy implements Observer
{
    // 被观察者状态更新，调用此函数
    @Override
    public void update(Observable observable, Object obj)
    {
        System.out.print("I'm Amy. I receive a paper");
        System.out.print("[title: " + ((Paper)observable).getTitle());
        System.out.print(", content: " + ((Paper)observable).getContent() + "]");
        System.out.print(" at " + new Date() + "\n");
    }
}

class Alice implements Observer
{
    // 被观察者状态更新，调用此函数
    @Override
    public void update(Observable observable, Object obj)
    {
        System.out.print("I'm Alice. I just try to subscribe a paper, and I receive a paper");
        System.out.print("[title: " + ((Paper)observable).getTitle());
        System.out.print(", content: " + ((Paper)observable).getContent() + "]");
        System.out.print(" at " + new Date() + "\n");
    }
}
```

### 验证程序

首先，先让Tom和Amy订阅报纸，然后生产一份报纸，此时只有Tom和Amy收到了报纸，而Alice没有收到报纸。

然后，Alice觉得报纸也挺有趣的，决定订阅报纸，于是Alice加入了观察者队列中，这样生产第二份报纸时，Tom、Amy、Alice都能收到报纸。

接着，Alice觉得报纸不符合自己的兴趣，决定退订报纸，于是Alice从观察者队列中移除出来，这样生产第三份报纸时，Tom、Amy能收到报纸，而Alice已经收不到报纸了。

程序如下

```java
public class TestObserver
{
    public static void main(String[] args)
    {
        // 初始化
        Paper paper = new Paper();
        Tom tom = new Tom();
        Amy amy = new Amy();
        Alice alice = new Alice();
        
        // Tom和Amy订阅Paper
        System.out.println("----Tom subscribe the paper----");
        paper.addObserver(tom);
        System.out.println("----Amy subscribe the paper----");
        paper.addObserver(amy);
        
        // 生产一份报纸，Tom和Amy收到报纸, Alice没订阅报纸，没收到通知
        System.out.println("----make first paper: Tom and Amy will receive，but Alice not----");
        paper.makeNewPaper("Fisrt Paper", "Hello World!");
        
        // Alice觉得报纸不错，要订阅报纸
        System.out.println("----Alice subscribe the paper----");
        paper.addObserver(alice);
        
        // 生产第二份报纸，Tom、Amy、Alice都收到报纸
        System.out.println("----make secode paper: Tom、Amy and Alice will receive----");
        paper.makeNewPaper("Second Paper", "Weclome Alice to join us!");
        
        // Alice觉得报纸不适合自己，决定退订
        System.out.println("----Alice unsubscribe the paper----");
        paper.deleteObserver(alice);
        
        // 生产第三份保证，Tom和Amy能收到，而Alice已经收不到了
        System.out.println("----make third paper: Tom and Amy will receive，but Alice not----");
        paper.makeNewPaper("Third Paper", "Sorry for Alice leaving!");
    }
}
```

输出如下

![使用java内置观察者模式实现程序的输出](/images/observerPatternOutput2.png)

## 比较

使用 java 内置的观察者模式需要`java.util.Observable`和`java.util.Observer`两者紧密结合使用，缺一不可。而被观察者只能通过继承`java.util.Observable`的方式，这对于 java 这种单继承语言来说可以说是限制了被观察者类的扩展性。而使用自己实现观察者模式的方式时，灵活性和可扩展性都比较大，但是这种方式需要编写的代码量比较多，不如 java 内置的观察者模式简便，而且在线程安全上，java 内置的观察者模式已经用`synchronized`和`vector`保证了线程安全，自己实现观察者模式则需要额外编写代码保证线程安全。

因此，从可扩展性和灵活性上讲，推荐自己实现观察者模式，从简便性和线程安全方面上考虑，推荐使用 java自带的观察者模式。

# 一些思考

很多地方使用到了观察者模式，比如JaveBeans, swing, Dubbo。它很适合用于监控某个对象的状态变化，例如监控一个文件的状态改变，当有新数据加入这个文件时，状态发生改变，可以通知其他需要读取该文件的进程读取文件内容进行相应的处理。

# 参考资料

[1] Eric Freeman等，Head First 设计模式（中文版）[M]，北京：中国电力出版社，2007