---
title: java反射基本用法
date: 2018-06-06 22:10:30
tags: java
categories: java
---

# 概述

java反射允许程序在运行时获取类的相关信息并动态的调用类实例的函数（包括私有函数），设置成员变量的值等。java反射在很多地方都有应用，例如，通过配置项调用对应的类中的函数。在写单元测试时，有时需要对私有方法进行单独的测试，此时也可以使用反射调用私有的方法。

<!-- more -->

# 获取类对象

通过反射获取类对象主要有三种方式

* 通过实例：instance.getClass()，此种方式需要有一个类实例。如`person.getClass()`，其中person是Person的实例
* 通过类名：className.class，此种方式需要导入类。如：`Person.class`
* 通过类全路径名：Class.forName(classFullPathName)，此种方式需要写类的全路径名（包含包路径）。如`Class.forName("reflect.Person")`

# 获取构造函数

* 使用getConstructors()获取所有的public构造函数
* 使用getDeclaredConstructors()获取所有的private、protected、public构造函数
* 使用getConstructors(obj.class...)获取特定的public构造函数
* 使用getDeclaredConstructors(obj.class...)获取特定的private、protected、public方法
* 通过constructors.newInstance可以创建类实例
* 访问private和protected构造函数需要设置`accessible`值为true，如`constructor2.setAccessible(true)`

# 获取成员函数

* 使用getMethods()获取所有的public方法，包括从父类继承过来的方法
* 使用getDeclaredMethods()获取本类所有的private、protected、public方法，不包含从父类继承过来的方法
* 使用getMethod(name, obj.class...)获取特定public方法
* 使用getDeclaredMethod(name, obj.class...)获取特定的private、protected、public方法
* 调用方法用method.invoke(instance, param...)，可以有返回值，是Object对象，如果需要指定类型要进行转换
* 访问private和protected方法需要设置`accessible`值为true，如`method2.setAccessible(true)`

# 获取成员变量

* 使用getFields()获取所有public变量
* 使用getDeclaredFields()获取所有private、protected、public变量
* 使用getField(fieldName)获取特定的public变量
* 使用getDeclaredField(fieldName)获取特定的private、protected、public变量
* 设置成员变量用field.set(instance, value)
* 访问private和protected方法需要设置`accessible`值为true，如`field2.setAccessible(true)`

# 示例代码

## 基础类

```java
package reflect;

import java.util.List;

public class Person
{
    // 成员变量
    private String name;
    public int age;
    protected List<String> aliasNames;

    // 构造函数
    private Person()
    {
        System.out.println("call constructor: private Person()");
    }
    
    protected Person(String name)
    {
        this.name = name;
        System.out.println("call constructor: protected Person(String name)");
    }
    
    public Person(int age, List<String> aliasNames)
    {
        this.age = age;
        this.aliasNames = aliasNames;
        System.out.println("call constructor: public Person(int age, List<String> aliasNames)");
    }
    
    // 成员函数
    private int getAge()
    {
        System.out.println("call method: private int getAge()");
        return age;
    }
    
    protected List<String> addAliasName(String aliasName)
    {
        System.out.println("call method: protected List<String> addAliasName(String aliasName)");
        aliasNames.add(aliasName);
        return aliasNames;
    }

    public void setName(String name)
    {
        System.out.println("call method: public void setName(String name)");
        this.name = name;
    }

    @Override
    public String toString()
    {
        return "Person [name=" + name + ", age=" + age + ", aliasNames=" + aliasNames + "]";
    }
}
```

## 测试类

```java
package reflect;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class Main
{
    public static void main(String[] args) throws Exception
    {
        // 三种方式获取类
        // 1. 通过实例：instance.getClass()，此种方式需要有一个类实例
        // 2. 通过类名：Class.class，此种方式需要导入类
        // 3. 通过类全路径名：Class.forName()，此种方式需要写类的全路径名（包含包路径）
        System.out.println("======获取类======");
        Person person = new Person("aaa");
        Class<?> personClass1 = person.getClass();
        Class<?> personClass2 = Person.class;
        Class<?> personClass3 = Class.forName("reflect.Person");
        
        System.out.println("person.getClass() == Person.class? " + (personClass1 == personClass2));
        System.out.println("Person.class == Class.forName(\"Person\")? " + (personClass2 == personClass3));
        
        // 使用getConstructors()获取所有的public构造函数
        // 使用getDeclaredConstructors()获取所有的private、protected、public构造函数
        // 使用getConstructors(obj.class...)获取特定的public构造函数，通过newInstance可以创建实例
        // 使用getDeclaredConstructors(obj.class...)获取特定的private、protected、public构造函数，通过newInstance可以创建实例
        System.out.println("======获取构造函数======");
        System.out.println("------1. 使用getConstructors()获取所有的public构造函数------");
        Constructor[] allPublicContructors = personClass3.getConstructors();
        Arrays.stream(allPublicContructors).forEach(c -> System.out.println(c));
        
        System.out.println("------2. 使用getDeclaredConstructors()获取所有的private、protected、public构造函数------");
        Constructor[] allDeclareContructorss = personClass3.getDeclaredConstructors();
        Arrays.stream(allDeclareContructorss).forEach(c -> System.out.println(c));
        
        System.out.println("------3. 使用getConstructors(obj.class...)获取特定的public构造函数，并调用------");
        Constructor constructor = personClass3.getConstructor(int.class, List.class);
        System.out.println(constructor);
        List<String> aliasNames = new ArrayList<>();
        aliasNames.add("bbb");
        Person person2 = (Person)constructor.newInstance(9, aliasNames);
        System.out.println(person2);
        
        System.out.println("------4. 使用getDeclaredConstructors(obj.class...)获取特定的private、protected、public构造函数,并调用------");
        Constructor constructor2 = personClass3.getDeclaredConstructor();  // 获取无参构造函数时，可以不用传参数
        constructor2.setAccessible(true);  // 访问private和protected构造函数需要设置为true
        System.out.println(constructor2);
        Object person3 = constructor2.newInstance();
        System.out.println(person3);
        
        // 使用getMethods()获取所有的public方法，包括从父类继承过来的方法
        // 使用getDeclaredMethods()获取本类所有的private、protected、public方法，不包含从父类继承过来的方法
        // 使用getMethod(name, obj.class...)获取特定public方法
        // 使用getDeclaredMethod(name, obj.class...)获取特定的private、protected、public方法
        // 调用方法用method.invoke(instance, param...)，可以有返回值，是Object对象，如果需要指定类型要进行转换
        System.out.println("======获取方法======");
        System.out.println("------1. 使用getMethods()获取所有的public方法，包括从父类继承过来的方法------");
        Method[] allPublicMethods = personClass2.getMethods();
        Arrays.stream(allPublicMethods).forEach(x -> System.out.println(x));
        
        System.out.println("------2. 使用getDeclaredMethods()获取本类所有的private、protected、public方法，不包含从父类继承过来的方法------");
        Method[] allDeclaredMethods = personClass2.getDeclaredMethods();
        Arrays.stream(allDeclaredMethods).forEach(x -> System.out.println(x));
        
        System.out.println("------3. 使用getMethod(name, obj.class...)获取特定public方法，并使用------");
        Method method = personClass2.getMethod("setName", String.class);
        System.out.println(method);
        System.out.println("before change name: " + person);
        method.invoke(person, "ccc");
        System.out.println("after change name: " + person);
        
        System.out.println("------4. 使用getDeclaredMethod(name, obj.class...)获取特定的private、protected、public方法，并使用-----");
        Method method2 = personClass2.getDeclaredMethod("addAliasName", String.class);
        method2.setAccessible(true);  // 访问private和protected方法需要设置为true
        System.out.println(method2);
        System.out.println("before add alias name: " + person2);
        List<String> retVal = (List<String>)method2.invoke(person2, "ddd");
        System.out.println("after add alias name: " + person2);
        System.out.println("the return value is " + retVal);
        
        // 使用getFields()获取所有public变量
        // 使用getDeclaredFields()获取所有private、protected、public变量
        // 使用getField(fieldName)获取特定的public变量
        // 使用getDeclaredField(fieldName)获取特定的private、protected、public变量
        // 设置成员变量用field.set(instance, value)
        System.out.println("======获取变量======");
        System.out.println("------1. 使用getFields()获取所有public变量------");
        Field[] allPublicFields = personClass1.getFields();
        Arrays.stream(allPublicFields).forEach(x -> System.out.println(x));
        
        System.out.println("------2. 使用getDeclaredFields()获取所有private、protected、public变量------");
        Field[] allDeclaredFields = personClass1.getDeclaredFields();
        Arrays.stream(allDeclaredFields).forEach(x -> System.out.println(x));
        
        System.out.println("------3. 使用getField(fieldName)获取特定的public变量，并设置值------");
        Field field = personClass1.getField("age");
        System.out.println(field);
        System.out.println("before change age: " + person);
        field.set(person, 99);
        System.out.println("after change age: " + person);
        
        System.out.println("------4. 使用getDeclaredField(fieldName)获取特定的private、protected、public变量，并设置值------");
        Field field2 = personClass1.getDeclaredField("aliasNames");
        field2.setAccessible(true);  // 访问private和protected成员变量需要设置为true
        System.out.println(field2);
        System.out.println("before change alaisNames: " + person2);
        List<String> newVal = new ArrayList<>();
        newVal.add("ppp");
        field2.set(person2, newVal);
        System.out.println("after change alaisNames: " + person2);
    }
}
```

## 输出

```
======获取类======
call constructor: protected Person(String name)
person.getClass() == Person.class? true
Person.class == Class.forName("Person")? true
======获取构造函数======
------1. 使用getConstructors()获取所有的public构造函数------
public reflect.Person(int,java.util.List)
------2. 使用getDeclaredConstructors()获取所有的private、protected、public构造函数------
public reflect.Person(int,java.util.List)
protected reflect.Person(java.lang.String)
private reflect.Person()
------3. 使用getConstructors(obj.class...)获取特定的public构造函数，并调用------
public reflect.Person(int,java.util.List)
call constructor: public Person(int age, List<String> aliasNames)
Person [name=null, age=9, aliasNames=[bbb]]
------4. 使用getDeclaredConstructors(obj.class...)获取特定的private、protected、public构造函数,并调用------
private reflect.Person()
call constructor: private Person()
Person [name=null, age=0, aliasNames=null]
======获取方法======
------1. 使用getMethods()获取所有的public方法，包括从父类继承过来的方法------
public java.lang.String reflect.Person.toString()
public void reflect.Person.setName(java.lang.String)
public final void java.lang.Object.wait() throws java.lang.InterruptedException
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
public boolean java.lang.Object.equals(java.lang.Object)
public native int java.lang.Object.hashCode()
public final native java.lang.Class java.lang.Object.getClass()
public final native void java.lang.Object.notify()
public final native void java.lang.Object.notifyAll()
------2. 使用getDeclaredMethods()获取本类所有的private、protected、public方法，不包含从父类继承过来的方法------
public java.lang.String reflect.Person.toString()
public void reflect.Person.setName(java.lang.String)
protected java.util.List reflect.Person.addAliasName(java.lang.String)
private int reflect.Person.getAge()
------3. 使用getMethod(name, obj.class...)获取特定public方法，并使用------
public void reflect.Person.setName(java.lang.String)
before change name: Person [name=aaa, age=0, aliasNames=null]
call method: public void setName(String name)
after change name: Person [name=ccc, age=0, aliasNames=null]
------4. 使用getDeclaredMethod(name, obj.class...)获取特定的private、protected、public方法，并使用-----
protected java.util.List reflect.Person.addAliasName(java.lang.String)
before add alias name: Person [name=null, age=9, aliasNames=[bbb]]
call method: protected List<String> addAliasName(String aliasName)
after add alias name: Person [name=null, age=9, aliasNames=[bbb, ddd]]
the return value is [bbb, ddd]
======获取变量======
------1. 使用getFields()获取所有public变量------
public int reflect.Person.age
------2. 使用getDeclaredFields()获取所有private、protected、public变量------
private java.lang.String reflect.Person.name
public int reflect.Person.age
protected java.util.List reflect.Person.aliasNames
------3. 使用getField(fieldName)获取特定的public变量，并设置值------
public int reflect.Person.age
before change age: Person [name=ccc, age=0, aliasNames=null]
after change age: Person [name=ccc, age=99, aliasNames=null]
------4. 使用getDeclaredField(fieldName)获取特定的private、protected、public变量，并设置值------
protected java.util.List reflect.Person.aliasNames
before change alaisNames: Person [name=null, age=9, aliasNames=[bbb, ddd]]
after change alaisNames: Person [name=null, age=9, aliasNames=[ppp]]
```

# 参考资料

[https://blog.csdn.net/sinat_38259539/article/details/71799078](https://blog.csdn.net/sinat_38259539/article/details/71799078)