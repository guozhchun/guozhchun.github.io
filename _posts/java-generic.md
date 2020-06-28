---
title: java 泛型
date: 2020-01-05 15:56:27
tags: java
categories: java
---

# 为什么使用泛型

大部分情况下，类和方法中用到的参数或变量都是基本类型或者是某种具体的类型（Integer、String 或 自己编写的类······）。这样做的好处是代码阅读起来很清晰。但是，如果要对多种不同的类型使用相同的功能，仍然采用具体的类型就会产生很多重复的代码，而采用 `Object` 类型代替所有类型虽然能消除重复代码，但是却存在多种类型混用导致转换错误的隐患。而泛型刚好融合了上述两种方式的优点，既能够用一套代码达到重复利用的目的，也能避免多种类型混用的强制转换错误隐患的发生。

<!-- more -->

假设现在要分别对 `Integer` 和 `String` 类型实现存储和读取，用三种方式实现如下

```java
// 方式一：底层分别用 Integer 和 String 实现，写成 IntegerArrayList 和 StringArrayList 两个类型
class IntegerArrayList
{
    private Integer[] elements;
    public Integer get(int i)
    {
        // 其他代码
        return elements[i];
    }
    public void add(int i, Integer element)
    {
        // 其他代码
        elements[i] = element;
        // 其他代码
    }
    // 其他代码
}

class StringArrayList
{
    private String[] elements;
    public String get(int i)
    {
        // 其他代码
        return elements[i];
    }
    public void add(int i, String element)
    {
        // 其他代码
        elements[i] = element;
        // 其他代码
    }
    // 其他代码
}

// 方法一调用
IntegerArrayList integerArrayList = new IntegerArrayList();
integerArrayList.add(0, 1);        // OK
integerArrayList.add(0, "hello");  // 编译报错，类型不匹配
Integer a = integerArrayList(0);   // OK

StringArrayList stringArrayList = new StringArrayList();
stringArrayList.add(0, "hello");   // OK
stringArrayList.add(0, 1);         // 编译报错，类型不匹配
String b = stringArrayList.get(0); // OK


// 方法二： 用 Object 实现，允许不同类型混用，存在类型强制转换报错风险
class ObjectArrayList
{
    private Object[] elements;
    public Object get(int i)
    {
        // 其他代码
        return elements[i];
    }
    public void add(int i, Object element)
    {
        // 其他代码
        elements[i] = element;
        // 其他代码
    }
    // 其他代码
}

// 方法二调用
ObjectArrayList objectArrayList = new ObjectArrayList();
objectArrayList.add(0, 1);        // OK
objectArrayList.add(0, "hello");  // OK
Integer a = (Integer) objectArrayList.get(0); // 运行时报强制类型转换错误，String不能转Integer
String b = (String) objectArrayList.get(0);   // 0K


// 方法三：泛型
class GenericArrayList<E>
{
    private E[] elements;
    public E get(int i)
    {
        // 其他代码
        return elements[i];
    }
    public void add(int i, E element)
    {
        // 其他代码
        elements[i] = element;
        // 其他代码
    }
    // 其他代码
}

// 方法三调用
GenericArrayList<Integer> genericArrayList = new GenericArrayList<>();
genericArrayList.add(0, 1);       // OK
genericArrayList.add(0, "hello"); // 编译报错，类型不匹配
```

从上述例子中可以看出，使用泛型，可以消除重复的功能代码，也能让某块功能只保持一种类型，避免多种不同类型混用，同时能够在编译时期发现不合法的强制类型转换，避免将错误遗留到运行期。

# 泛型的用法

## 泛型类

泛型用在类上时，需要在定义类名后面用 `<>` 表明，`<>` 里面的 `E` 表示某种类型，`E` 可以是其他的字母，如果有多种不同的类型，可以通用 `,` 隔开，如定义 `map` 时用 `<K, V>`。

在创建泛型类的对象实例时，需要将具体的类型传入，如 `new GenericArrayList<Integer> `

```java
class GenericArrayList<E>
{
    private E[] elements;
    public E get(int i)
    {
        // 其他代码
        return elements[i];
    }
    public void add(int i, E element)
    {
        // 其他代码
        elements[i] = element;
        // 其他代码
    }
    // 其他代码
}
```

## 泛型接口

泛型用在接口上时，其定义跟泛型类一样。但是在实现类中，需要将实现类也声明为泛型类；或者将具体的类型传入接口的泛型参数，此时，在实现类中，接口中的相关泛型参数会被替换具体的类型。

```java
public interface GenericInterface<T>
{
    void doSomething(T t);
}

// 实现类：接口中传入具体的类型
public class GenericInterfaceImpl implement GenericInterface<Integer>
{
    @Override
    void doSomething(Integer t) {}
}

// 实现类：接口中不传入具体类型，但是将实现类定位为泛型类
public class GenericClass<T> implement GenericInterface<T>
{
    @Override
    void doSomething(T t) {}
}
```

## 泛型方法

泛型同样可以用于方法，用于方法时，此泛型参数跟类（或者接口）中的泛型参数并无任何关系。另外，如果泛型参数没有指定边界时，此时跟用 `Object` 类型效果相同。

```java
// 以下两种写法等价，而且不能同时出现，因为泛型擦除后也是 Object 类型，变成同名同类型函数，无法通过编译
public void print(Object obj)
{
    System.out.println(obj.getClass().getName());
}

public <T> void print(T t)
{
    System.out.println(t.getClass().getName());
}

// 调用输出，无论调用上面哪种写法的函数，输出结果相同。
print("");    // java.lang.String
print(1);     // java.lang.Integer
print(1.0);   // java.lang.Double
print(1.0F);  // java.lang.Float
print('c');   // java.lang.Character
```

# 泛型的原理

java 中的泛型和 C++ 中的泛型不同，C++ 中的泛型是强泛型，在编译期和运行期都持有泛型的信息，而 java 中的泛型是一个*假泛型*，或者说是一个语法糖，其信息只保留到编译期的静态类型检查（语义分析中的第一部分），在此之后，泛型信息将被擦除（语义分析中另一部分：解语法糖），无法保留到运行期。也就是说，泛型只是为了方便代码的编写，其本质上跟原型类型并无太大差别。

## 泛型擦除

泛型类型只有在静态类型检查期间才出现，在此之后，程序中的所有泛型类型都将被擦除，替换为它们的非泛型上界，如 `List<? extend Integer>` 将被擦除为 `List<Integer>`，`List<T>` 将被擦除为 `List<Object>` ，而普通的类型变量在未指定边界的情况下将被查出为 `Object`，如 `T`  被擦除为 `Object`。

下面两个程序，是用 `Object` 和泛型实现的同一个功能，对某个元素进行赋值和读取的操作。从反编译出来的字节码可以看到，两者在进行相关函数的操作时动作是完全一样的。也就是说泛型类型 `String` 并没有保留到字节码中，在进行 `get` 操作时，仍然要进行类型转换（`checkcast` 指令），但是在源码中并没有这一步，这一步其实是编译器自动补充进去的。

使用 `Object` 类型实现的类及其字节码

```java
public class SimpleHolder
{
    private Object obj;
    
    public void set(Object obj)
    {
        this.obj = obj;
    }
    
    public Object get()
    {
        return obj;
    }

    public static void main(String[] args)
    {
        SimpleHolder holder = new SimpleHolder();
        holder.set("hello world");
        String s = (String) holder.get();
    }
}


// 字节码
Compiled from "SimpleHolder.java"
public class SimpleHolder {
  public SimpleHolder();
    Code:
       0: aload_0       
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return        

  public void set(java.lang.Object);
    Code:
       0: aload_0       
       1: aload_1       
       2: putfield      #2                  // Field obj:Ljava/lang/Object;
       5: return        

  public java.lang.Object get();
    Code:
       0: aload_0       
       1: getfield      #2                  // Field obj:Ljava/lang/Object;
       4: areturn       

  public static void main(java.lang.String[]);
    Code:
       0: new           #3                  // class SimpleHolder
       3: dup           
       4: invokespecial #4                  // Method "<init>":()V
       7: astore_1      
       8: aload_1       
       9: ldc           #5                  // String hello world
      11: invokevirtual #6                  // Method set:(Ljava/lang/Object;)V
      14: aload_1       
      15: invokevirtual #7                  // Method get:()Ljava/lang/Object;
      18: checkcast     #8                  // class java/lang/String
      21: astore_2      
      22: return        
}
```

使用泛型实现的类及其字节码

```java
public class GenericHolder<T>
{
    private T obj;
    
    public void set(T obj)
    {
        this.obj = obj;
    }
    
    public T get()
    {
        return obj;
    }

    public static void main(String[] args)
    {
        GenericHolder<String> holder = new GenericHolder<>();
        holder.set("hello world");
        String s = holder.get();
    }
}


// 字节码
Compiled from "GenericHolder.java"
public class GenericHolder<T> {
  public GenericHolder();
    Code:
       0: aload_0       
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return        

  public void set(T);
    Code:
       0: aload_0       
       1: aload_1       
       2: putfield      #2                  // Field obj:Ljava/lang/Object;
       5: return        

  public T get();
    Code:
       0: aload_0       
       1: getfield      #2                  // Field obj:Ljava/lang/Object;
       4: areturn       

  public static void main(java.lang.String[]);
    Code:
       0: new           #3                  // class GenericHolder
       3: dup           
       4: invokespecial #4                  // Method "<init>":()V
       7: astore_1      
       8: aload_1       
       9: ldc           #5                  // String hello world
      11: invokevirtual #6                  // Method set:(Ljava/lang/Object;)V
      14: aload_1       
      15: invokevirtual #7                  // Method get:()Ljava/lang/Object;
      18: checkcast     #8                  // class java/lang/String
      21: astore_2      
      22: return        
}
```

## 泛型擦除的问题

泛型擦除会给函数重载、`instanceof`、泛型实例化带来问题。如下所示，代码均由于泛型擦除而无法通过编译。

```java
class TestGeneric<T>
{
    // 编译报错，print(Object obj) 泛型擦除后与 print(Object obj) 有相同的函数签名 
    public <E> void print(E e)
    {
        System.out.println(e);
    }
    
    public void print(Object obj)
    {
        System.out.println(obj);
    }
    
    // 编译报错，List<String> list 和 print(List<Integer> list) 泛型擦除后有相同的函数签名 
    public void print(List<String> list)
    {
        System.out.println(list);
    }
    
    public void print(List<Integer> list)
    {
        System.out.println(list);
    }
    
    public void test(T obj)
    {
        // 编译报错，运行其没有具体的类型信息，无法进行判断
        if (obj instanceof T) {}
        
        // 编译报错，运行其没有具体的类型信息，无法新建具体实例
        T o = new T;
        
        // 编译报错，运行其没有具体的类型信息，无法新建具体类型的数组
        T[] array = new T[1];
    }
}
```

对于泛型的类型判断，虽然无法通过 `instanceof` 来完成，但是可以通过 `isInstance` 来判断，不过此时，需要传入一个 `class` 对象，如下所示

```java
class GenericType<T>
{
    private Class<T> kind
        
    public GenericType(Class<T> kind)
    {
        this.kind = kind;
    }
    
    public boolean isInstance(Object obj)
    {
        return kind.isInstance(obj);
    }
    
    public static void main(String[] args)
    {
        GenericType<Integer> genericType = new GenericType<>(Integer.class);
        System.out.println(genericType.isInstance(new Integer(3)));  // true
    }
}
```

对于创建泛型的实例，稍微麻烦一点，需要通过工厂类的方式来创建，如下所示

```java
interface Factory<T>
{
    T create();
}

class IntegerFactory implements Factory<Integer>
{
    @Override
    public Integer create()
    {
        return new Integer(0);
    }
}
```

## 其他

泛型会擦除，那对于以下代码，都会把泛型类型 `String` 和 `Integer` 都擦除成 `Object` ，但是为什么编译器仍然报错了呢。主要原因是，在编译期间，有很多步骤，在语义分析阶段，会先进行标注检查（包含静态类型检查），然后再进行解语法糖（泛型擦除），而在静态类型检查时，泛型类型的信息仍然存在，此时 `ArrayList<Integer>` 类型的参数无法转成 `ArrayList<String>` 类型，无法通过编译，根本就没有到达泛型擦除那部分，所以即使泛型擦除后两者是一致的，但是仍然无法通过编译，而这也是 java 提供泛型的一个初衷：将问题提前在编译期暴露出来。

```java
public void test(ArrayList<String> list) {...}

test(new ArrayList<Integer>());
```

# 泛型边界

## 上界通配符 <? extend T>

`<? extend T>` 表示能接受的泛型类型是 `T` 或者 `T` 的子类，如 `<? extend Number>` 表示接受的泛型类型是 `Number` 或 `Number` 的子类。对于上界通配符，对于 `T` 的子类，只允许读操作（类似 `List.get` 的操作），不允许写操作（类似 `List.add` 操作）。

因为泛型擦除会替换为它们的非泛型**上界**，所以使用 `List<? extend T>` 在泛型擦除后，实际上变成 `List<T>` 类型，对于 `T` 的子类，类型不匹配，无法添加进去，所以上界通配符对于子类是不支持写操作的。

另一方面，对于读操作，由于获取出来的元素都是 `T` 及 `T` 的子类，所以可以直接用 `T` 类型来表示，因此上界通配符对于子类是支持读操作的。

## 下界通配符 <? super T>

`<? super T>` 表示接受的泛型类型是 `T` 或者 `T` 的父类（祖先类），如 `<? super Integer>` 表示接受的泛型类型是 `Integer` 或者 `Integer` 的父类（如 `Number`）。对于下界通配符，对于 `T` 的父类（祖先类），只允许写操作（类似 `List.add` 的操作），不允许读操作（类似 `List.get` 操作）。

因为泛型擦除会替换为它们的非泛型**上界**，而 `List<? super T>` 并没有明确其上边界，所以其上边界是 `Object`。那么在泛型擦除后，其会变成 `List<Object>`，自然所有的类型都能放进去（`T` 的子类放不进去是因为类型校验失败），所以下界通配符对于父类（祖先类）是支持写操作的。

另一方面，对于读操作，因为集合中存放的是 `T` 及 `T` 的祖先类，无法用 `T` 类型来统一表示（会发生类型转换错误）。所以，下界通配符对于父类（祖先类）是不支持读操作的。

## 无界通配符<?>

无界通配符（如 `List<?>`）表示可以使用任何对象，因此使用它类似于使用原生类型(如 `List`)。但它跟原生类型又有点差别，原先类型可以持有任何类型，而无界通配符修饰的容器持有的是某种具体的类型。

## PECS原则

`PECS` 是  `Producer Extends,Consumer Super`  的缩写。主要是为了判断什么情况下用 `extend`，什么情况下用 `supper`。其说明如下

从集合的角度出发看问题，如果从集合中取数据，则集合是生产者，用 `? extend T`。如果往集合中放数据，则集合是消费者，用 `? supper`。

# 参考资料

 [1] Eckel,B. Java编程思想（第四版）[M]. 北京：机械工业出版社，2007.6 