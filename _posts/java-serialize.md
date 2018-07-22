---
title: java序列化
date: 2018-07-15 13:38:15
tags: java
categories: java
---

# 概述

当需要将一个对象进行持久化时，除了保存到数据库或写入到文件之外，java中还提供了一种“轻量级”的方式：序列化。java的对象序列化将那些实现了Serializable接口的对象转换成一个字节序列，并能够在以后将这个字节序列完全恢复为原来的对象，这个恢复过程称为反序列化。

<!-- more -->

# 序列化特点

* 如果父类实现了Serializable接口，子类默认实现了Serializable接口，即使子类没有明确声明也一样
* 序列化只保存对象的成员变量信息，并不保存对象的方法信息
* 对于只实现Serializable接口的对象进行序列化时，静态成员变量（static修饰的变量）和transient 变量（transient修饰的变量），不会进行序列化
* 静态成员变量和transient变量可以通过实现Serializable接口同时增加writeObject和readObject函数并在writeObject函数中手动对变量进行序列化，或者让对象实现Externalizale接口同时增加writeExternal和readExternal函数并在writeExternal函数中手动对变量进行序列化。
* 当对象进行序列化时，如果对象中有成员变量不能进行序列化（非基本成员变量，非静态成员变量，非transient变量，非空变量且没有实现Serializable接口），会抛出NotSerializableException异常
* 当对象被序列化时，如果对象包含了对其他对象的引用，则其他对象也会被序列化。比如说一个类A里面有一个变量 b 是引用B类的一个对象，则当A类的对象a被序列化时，b对象也会被序列化。 
* 对象序列化时，先序列化父类的成员变量，再序列化子类的成员变量

# 反序列化特点

* 对实现Serializable接口的对象进行反序列化时不会调用对象的构造函数
* 对实现Externalizale接口的对象进行反序列化时，会调用对象的默认构造函数（即无参构造函数），如果对象没有默认的构造函数，则会抛出`java.io.InvalidClassException: no valid constructor`异常
* 反序列化是在堆中新建对象并尝试还原保存的对象的状态，而不是对原有对象的操作
* 反序列化时要求能够获取到反序列化对象的class文件，否则会抛出ClassNotFoundException异常
* 对静态变量和transient变量进行默认反序列化时（由系统保证，不自己单独实现反序列化），如果没有在成员变量声明时赋值，则默认是零值（基本类型是0（boolean是false），非基本类型是null），如果在声明变量时赋值了，则值不变。

# 序列化方式

将对象变成可序列化主要有以下几种方式：只实现Serializable接口；实现Serializable接口并增加writeObject和readObject方法；实现Externalizable接口。

Serializable接口是一个标记接口，并没有接口方法需要实现类实现，而Externalizable接口继承了Serializable接口，但是却有readExternal和writeExternal两个接口方法需要实现类实现。

## 只实现Serializable接口

### 进行序列化的Person对象

此处用Person类表示要进行序列化的对象。

成员变量中，用`DEFAULT_GENDER`变量验证`static final`的序列化和反序列化过程，用`count`变量验证`static`变量的序列化和反序列化的过程，用`id`变量验证`transient`变量的序列化和反序列化的过程。

构造函数中，有一个无参的默认构造函数和一个有参的构造函数，在构造函数中均输出一条信息表明调用的是哪个构造函数，并将count值加一，主要用于在反序列化时验证值是否为0

在`toString`函数中，输出对象的信息，除了输出成员变量的信息，也输出了类变量（static变量）的信息，更重要的是输出了对象的地址信息（`super.toString()`），这个主要用于验证反序列化时是在堆上新建对象而不是在原有对象上进行修改。

```java
import java.io.Serializable;
public class Person implements Serializable
{
    private static final String DEFAULT_GENDER = "man";
    private static int count = 0;
    private transient int id;
    private String name;
    private int age;
    
    public Person()
    {
        System.out.println("Persion()");
        count++;
    }
    
    public Person(String name, int age, int id)
    {
        System.out.println("Person(String name, int age, int id)");
        this.name = name;
        this.age = age;
        this.id = id;
        count++;
    }
    
    public static int getCount()
    {
        return count;
    }
    public static void setCount(int count)
    {
        Person.count = count;
    }
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    public int getAge()
    {
        return age;
    }
    public void setAge(int age)
    {
        this.age = age;
    }

    @Override
    public String toString()
    {
        return "Person [" + super.toString() + "]" 
                + "[id=" + id + ", name=" + name 
                + ", age=" + age + ", count=" + count 
                + ", DEFAULT_GENDER=" + DEFAULT_GENDER + "]";
    }
}
```

### 进行序列化

此处将序列化和反序列分开在不同的程序中运行，这样更能说明静态变量不进行序列化。当然，也可以将反序列化的代码写在序列化的后面，这样除了静态变量的结果稍微需要注意一下之外，其他变量的结果跟序列化和反序列化程序分开的情况并无不同。下面程序中将注释的代码放开即可得到序列化和反序列化程序合在一起的结果。

在序列化的程序中，首先新建了一个person对象并输出对象信息，然后将该对象进行序列化输出，再改变person对象的成员变量值，这样做的目的是在反序列化时验证序列化之后又修改对象是否会影响序列化的结果

```java
public class SerializeOutput
{
    public static void main(String[] args) throws Exception
    {
        System.out.println("----New a Person----");
        Person person = new Person("Alice", 12, 1111);
        System.out.println(person);
        
        System.out.println("----Begin Serialize----");
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("person.txt"));
        objectOutputStream.writeObject(person);
        
        System.out.println("----Change Person----");
        person.setAge(21);
        person.setId(2222);
        person.setName("Bob");
        person.setCount(10);
        System.out.println(person);
        
//        System.out.println("----Begin Deserialize----");
//        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("person.txt"));
//        Person person1 = (Person) objectInputStream.readObject();
//        // 此处会输出 Person [serialize.Person@6bc7c054][id=0, name=Alice, age=12, count=10, DEFAULT_GENDER=man]
//        // 可以看到地址与序列化前的地址不一样，另外count的值是10，这是因为count是静态变量，属于类，不属于某个具体的对象
//        // 而此处是在同一个程序中，Person类并没有被销毁或新建，其类信息保存者，因此反序列化后虽然对象的静态成员变量赋值为零值
//        // 但是由于Person类相关信息仍然存在，在对象读取类变量时，会将类变量的值赋给对象，所以count输出值是10
//        System.out.println(person1);
    }
}
```

程序输出结果如下

```
----New a Person----
Person(String name, int age, int id)
Person [serialize.Person@15db9742][id=1111, name=Alice, age=12, count=1, DEFAULT_GENDER=man]
----Begin Serialize----
----Change Person----
Person [serialize.Person@15db9742][id=2222, name=Bob, age=21, count=10, DEFAULT_GENDER=man]
```

### 进行反序列化

反序列化的程序很简单，就是直接读取序列化产生的文件，输出对象信息

```java
public class SerializeInput
{
    public static void main(String[] args) throws Exception
    {
        System.out.println("----Begin Deserialize----");
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("person.txt"));
        Person person = (Person) objectInputStream.readObject();
        System.out.println(person);
    }
}
```

程序输出如下

```
----Begin Deserialize----
Person [serialize.Person@70dea4e][id=0, name=Alice, age=12, count=0, DEFAULT_GENDER=man]
```

对比序列化和反序列化的输出结果，可以看出

* 对象地址不同，这说明反序列化是在堆上新建对象，而不是在原有对象上进行修改
* id变量值不同，序列化时id是111，反序列化后是0，这说明序列化时并不会对transient变量进行序列化
* count变量值不同，序列化时count是1，反序列化后是0，这说明序列化时并不会对静态变量进行序列化
* 对person对象进行序列化后，改变person对象，再将其反序列化时，并不会将序列化对象后的修改反映到反序列化的对象中

## 实现Serializable接口并增加writeObject和readObject方法

对于静态成员变量和transient变量，虽然默认的序列化操作不会将其进行序列化，但是可以在对象中增加writeObject和readObject方法将其手动序列化。需要注意的是方法签名有严格的要求，必须是以下的形式：

* 修饰符必须是private
* 返回类型必须是void
* 函数名称必须是`writeObject`或`readObject`
* 函数入参只能有一个，且`writeObject`函数的入参只能是`ObjectOutputStream`类型参数，`readObject`函数的入参只能是`ObjectInputStream`类型参数

如下两个是合格writeObject和readObject方法签名

`private void writeObject(ObjectOutputStream stream)`

`private void readObject(ObjectInputStream stream)`

对于使用了Serializable接口并且增加了writeObject和readObject方法的对象而言，在序列化时，系统会先判断对象是否有writeObject函数（严格按照上述约束进行判断），如果有此函数，则调用此函数进行序列化，不会调用默认的序列化函数，如果需要执行默认的序列化操作，需要在writeObject函数中手动调用`defaultWriteObject()`函数进行默认的序列化操作。也就是说，如果不调用`defaultWriteObject()`函数，则只会将writeObject函数中手动序列化的变量进行序列化。

同样，在反序列化时，系统也会先判断对象是否有readObject函数，如果有此函数，则调用此函数进行反序列化，不会调用默认的反序列化函数，如果需要执行默认的反序列化函数，需要在readObject函数中手动调用`defaultReadObject()`函数进行默认的反序列化操作。也就是说，如果不调用`defaultReadObject()`函数，则只会将readObject函数中手动反序列化的变量进行序列化。

在将对象变量进行手动序列化和反序列化时，需要注意反序列化的顺序和序列化的顺序是一致的。

### 进行序列化的Person对象

与“只实现Serializable接口方式”的Person对象不同，此处增加了writeObject和readObject方法，并在writeObject中调用了defaultWriteObject进行了了默认的序列化操作后将transient变量和静态变量进行了序列化输出。相应地，在readObject中调用了deafultReadObject进行了默认的反序列化操作，然后将对应的transient变量和静态变量进行了反序列化。

```java
public class Person implements Serializable
{
    private static final String DEFAULT_GENDER = "man";
    private static int count = 0;
    private transient int id;
    private String name;
    private int age;
    
    public Person()
    {
        System.out.println("Persion()");
        count++;
    }
    
    public Person(String name, int age, int id)
    {
        System.out.println("Person(String name, int age, int id)");
        this.name = name;
        this.age = age;
        this.id = id;
        count++;
    }
    
    private void writeObject(ObjectOutputStream stream) throws Exception
    {
        System.out.println("----private void writeObject(ObjectOutputStream stream)----");
        // 调用defaultWriteObject函数，可以执行默认的序列化操作
        // 否则name和age这两个正常的变量不会进行序列化
        stream.defaultWriteObject();
        
        // 将transizent变量进行序列化
        stream.writeObject(id);
        
        // 将静态变量进行序列化
        stream.writeObject(count);
    }
    
    private void readObject(ObjectInputStream stream) throws Exception
    {
        System.out.println("----private void readObject(ObjectInputStream stream)----");
        // 调用defaultReadObject函数，可以执行默认的反序列化操作
        // 否则name和age这两个正常的变量不会进行反序列化
        stream.defaultReadObject();
        
        // 将transizent变量进行反序列化
        id = (int) stream.readObject();
        
        // 将静态变量进行反序列化
        count = (int) stream.readObject();
    }
    
    public static int getCount()
    {
        return count;
    }
    public static void setCount(int count)
    {
        Person.count = count;
    }
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    public int getAge()
    {
        return age;
    }
    public void setAge(int age)
    {
        this.age = age;
    }

    @Override
    public String toString()
    {
        return "Person [" + super.toString() + "]" 
                + "[id=" + id + ", name=" + name 
                + ", age=" + age + ", count=" + count 
                + ", DEFAULT_GENDER=" + DEFAULT_GENDER + "]";
    }
}
```

### 进行序列化

序列化的程序与“只实现Serializable接口”方式中的序列化程序相同，不同的是输出结果，从程序的输出结果中可以看出，序列化时调用了对象的writeObject函数。

```java
public class SerializeOutput
{
    public static void main(String[] args) throws Exception
    {
        System.out.println("----New a Person----");
        Person person = new Person("Alice", 12, 1111);
        System.out.println(person);
        
        System.out.println("----Begin Serialize----");
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("person.txt"));
        objectOutputStream.writeObject(person);
        
        System.out.println("----Change Person----");
        person.setAge(21);
        person.setId(2222);
        person.setName("Bob");
        person.setCount(10);
        System.out.println(person);
        
//        System.out.println("----Begin Deserialize----");
//        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("person.txt"));
//        Person person1 = (Person) objectInputStream.readObject();
//        // 此处会输出Person [serialize.Person@1540e19d][id=1111, name=Alice, age=12, count=1, DEFAULT_GENDER=man]
//        // 可以看到地址与序列化前的地址不一样，另外count的值是1，这是因为在序列化时将静态变量count进行了序列化输出
//        // 在反序列化时将该值进行反序列化，重新给类变量赋值，变成了1
//        System.out.println(person1);
    }
}
```

程序输出如下

```
----New a Person----
Person(String name, int age, int id)
Person [serialize.Person@15db9742][id=1111, name=Alice, age=12, count=1, DEFAULT_GENDER=man]
----Begin Serialize----
----private void writeObject(ObjectOutputStream stream)----
----Change Person----
Person [serialize.Person@15db9742][id=2222, name=Bob, age=21, count=10, DEFAULT_GENDER=man]
```

### 进行反序列化

反序列化的程序与“只实现Serializable接口”方式中的反序列化程序相同，不同的是输出结果，从程序的输出结果中可以看出，反序列化时调用了对象的readObject函数。而对比序列化和反序列化的输出结果，也可以看出静态变量和transient变量也进行了序列化和反序列化。

```java
public class SerializeInput
{
    public static void main(String[] args) throws Exception
    {
        System.out.println("----Begin Deserialize----");
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("person.txt"));
        Person person = (Person) objectInputStream.readObject();
        System.out.println(person);
    }
}
```

程序输出结果如下

```
----Begin Deserialize----
----private void readObject(ObjectInputStream stream)----
Person [serialize.Person@42a57993][id=1111, name=Alice, age=12, count=1, DEFAULT_GENDER=man]
```

## 实现Externalizable接口

与实现Serializable接口不同，实现Externalizable接口需要实现writeExternal和readExternal这两个接口方法。在进行序列化时，会调用writeExternal方法进行序列化，而在反序列化时，readExternal方法会被调用。需要注意的是，用此方法实现序列化和反序列化，必须手动对所有需要序列化的对象变量进行手动序列化和反序列化，包括基本数据类型的变量，否则不会对该变量进行序列化，而且此方法也没有类似defaulWriteObject和defaultReadObject函数可供调用实现默认的序列化和反序列化的功能。另外，此中方式实现反序列化时，会调用对象的默认构造函数（即无参构造函数），如果对象没有默认的构造函数，则会抛出`java.io.InvalidClassException: no valid constructor`异常。

在将对象变量进行手动序列化和反序列化时，需要注意反序列化的顺序和序列化的顺序是一致的。

### 进行序列化的Person对象

与“只实现Serializable接口”方式不同，此处实现Externalizable接口，并实现了writeExternal和readExternal接口方法。在writeExternal方法中，对所有的成员变量进行了手动序列化，包括基本成员变量（如age变量）、transient变量和静态变量。相应地，在readExternal方法中，按照序列化成员变量的顺序，对所有成员变量进行反序列化。

```java
public class Person implements Externalizable
{
    private static final String DEFAULT_GENDER = "man";
    private static int count = 0;
    private transient int id;
    private String name;
    private int age;
    
    public Person()
    {
        System.out.println("Persion()");
        count++;
    }
    
    public Person(String name, int age, int id)
    {
        System.out.println("Person(String name, int age, int id)");
        this.name = name;
        this.age = age;
        this.id = id;
        count++;
    }
    
    @Override
    public void writeExternal(ObjectOutput out) throws IOException
    {
        System.out.println("----public void writeExternal(ObjectOutput out)----");
        // 所有需要序列化的对象变量都需要手动序列化，包括基本类型的变量
        out.writeObject(name);
        out.writeObject(age);
        // 序列化transient变量
        out.writeObject(id);
        // 序列化静态变量
        out.writeObject(count);
    }
    
    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException
    {
        System.out.println("----public void readExternal(ObjectInput in)----");
        // 所有需要反序列化的对象变量都需要手动反序列化，包括基本类型的变量
        name = (String) in.readObject();
        age = (int) in.readObject();
        // 反序列化transient对象
        id = (int) in.readObject();
        // 反序列化静态变量
        count = (int) in.readObject();
    }

    public static int getCount()
    {
        return count;
    }
    public static void setCount(int count)
    {
        Person.count = count;
    }
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    public int getAge()
    {
        return age;
    }
    public void setAge(int age)
    {
        this.age = age;
    }

    @Override
    public String toString()
    {
        return "Person [" + super.toString() + "]" 
                + "[id=" + id + ", name=" + name 
                + ", age=" + age + ", count=" + count 
                + ", DEFAULT_GENDER=" + DEFAULT_GENDER + "]";
    }
}
```

### 进行序列化

进行序列化的程序与前两种方式相同，不同的是程序的输出结果。从输出结果可以看出，在序列化时调用了对象的writeExternal函数

```java
public class SerializeOutput
{
    public static void main(String[] args) throws Exception
    {
        System.out.println("----New a Person----");
        Person person = new Person("Alice", 12, 1111);
        System.out.println(person);
        
        System.out.println("----Begin Serialize----");
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("person.txt"));
        objectOutputStream.writeObject(person);
        
        System.out.println("----Change Person----");
        person.setAge(21);
        person.setId(2222);
        person.setName("Bob");
        person.setCount(10);
        System.out.println(person);
        
//        System.out.println("----Begin Deserialize----");
//        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("person.txt"));
//        Person person1 = (Person) objectInputStream.readObject();
//        // 此处会输出Person [serialize.Person@1540e19d][id=1111, name=Alice, age=12, count=1, DEFAULT_GENDER=man]
//        // 可以看到地址与序列化前的地址不一样，另外count的值是1，这是因为在序列化时将静态变量count进行了序列化输出
//        // 在反序列化时将该值进行反序列化，重新给类变量赋值，变成了1
//        System.out.println(person1);
    }
}
```

程序输出结果如下

```
----New a Person----
Person(String name, int age, int id)
Person [serialize.Person@15db9742][id=1111, name=Alice, age=12, count=1, DEFAULT_GENDER=man]
----Begin Serialize----
----public void writeExternal(ObjectOutput out)----
----Change Person----
Person [serialize.Person@15db9742][id=2222, name=Bob, age=21, count=10, DEFAULT_GENDER=man]
```

### 进行反序列化

反序列化的程序与前两种相同，不同的是输出结果，从程序的输出结果中可以看出，反序列化时调用了对象的readExternal函数。而对比序列化和反序列化的输出结果，也可以看出静态变量和transient变量也进行了序列化和反序列化。

```java
public class SerializeInput
{
    public static void main(String[] args) throws Exception
    {
        System.out.println("----Begin Deserialize----");
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("person.txt"));
        Person person = (Person) objectInputStream.readObject();
        System.out.println(person);
    }
}
```

程序输出结果如下

```
----Begin Deserialize----
Persion()
----public void readExternal(ObjectInput in)----
Person [serialize.Person@3d4eac69][id=1111, name=Alice, age=12, count=1, DEFAULT_GENDER=man]
```

# 扩展阅读

深入学习java序列化：[http://beautyboss.farbox.com/post/study/shen-ru-xue-xi-javaxu-lie-hua](http://beautyboss.farbox.com/post/study/shen-ru-xue-xi-javaxu-lie-hua)

# 参考资料

[1] Eckel,B. Java编程思想（第四版）[M]. 北京：机械工业出版社，2007.6