---
title: java 排序
date: 2018-08-19 20:19:38
tags: java
categories: java
---

# 概述

在平常的开发中经常需要用到排序，在 java 中主要有两种类型的排序，一种是数组 Arrays，另一种是集合 Collections，都是通过调用 sort 函数来实现排序的。而这也要求进行排序的对象需要 Comparable 接口或者在调用 sort 函数时将实现 Comparator 的对象作为另一个参数传入，从而达到排序的目的。

<!-- more -->

# 数组 Arrays 排序

数组的排序主要是依靠 `Arrays.sort` 函数来完成。由于数组的元素可以是由基本类型组成，也可以由对象组成，因此这个函数可以对基本类型的数组进行排序，也可以对对象数组进行排序。排序主要有两种方式，一种是由对象实现`Comparable ` 接口中的 `compareTo` 函数（*注意：基本类型不用实现这个接口*）。另一种是在调用 `Arrays.sort` 函数时将实现 `Comparator ` 接口的 `compare` 函数的对象作为参数传入该函数进行比较。

## 使用 Comparable 实现排序

使用 Comparable 的方式进行排序，要求排序的对象实现 Comparable 接口，并实现接口中的 compareTo 函数。此函数将本对象与另一个对象进行比较，如果本对象小于另一个对象，则返回负数，如果本对象大于另一个对象，则返回正数，相等则返回 0。

程序如下

```java
public class Student implements Comparable<Student>
{
    private int id;
    private int score;
    private String name;
    
    public Student(int id, int score, String name)
    {
        super();
        this.id = id;
        this.score = score;
        this.name = name;
    }
    
    public int getId()
    {
        return id;
    }

    public void setId(int id)
    {
        this.id = id;
    }

    public int getScore()
    {
        return score;
    }

    public void setScore(int score)
    {
        this.score = score;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    @Override
    public String toString()
    {
        return "Student [id=" + id + ", score=" + score + ", name=" + name + "]";
    }

    @Override
    public int compareTo(Student s)
    {
        return this.score - s.score;
    }
}
```

```java
public class TestSort
{
    public static void main(String[] args)
    {
        System.out.println("----init students----");
        Student[] students = new Student[4];
        students[0] = new Student(4, 10, "Tom");
        students[1] = new Student(2, 50, "Alice");
        students[2] = new Student(3, 30, "Tony");
        students[3] = new Student(1, 20, "Amy");
        Arrays.stream(students).forEach(x -> System.out.println(x));
        
        System.out.println("----sort by Comparable [by score]----");
        Arrays.sort(students);
        Arrays.stream(students).forEach(x -> System.out.println(x));
    }
}
```

程序输出如下

```
----init students----
Student [id=4, score=10, name=Tom]
Student [id=2, score=50, name=Alice]
Student [id=3, score=30, name=Tony]
Student [id=1, score=20, name=Amy]
----sort by Comparable [by score]----
Student [id=4, score=10, name=Tom]
Student [id=1, score=20, name=Amy]
Student [id=3, score=30, name=Tony]
Student [id=2, score=50, name=Alice]
```

## 使用 Comparator 实现排序

使用 Comparator 方式进行排序，不要求排序对象实现 Comparable  接口，只需要在调用 Arrays.sort 函数时将将实现 Comparator  接口的 compare 函数的对象作为参数传入该函数进行比较即可。compare 函数接收两个排序类型的参数，如果第一个对象对象小于第二个对象，则返回负数，如果第一个对象大于第二个对象，则返回正数，相等则返回 0。

程序如下

```java
public class Student
{
    private int id;
    private int score;
    private String name;
    
    public Student(int id, int score, String name)
    {
        super();
        this.id = id;
        this.score = score;
        this.name = name;
    }
    
    public int getId()
    {
        return id;
    }

    public void setId(int id)
    {
        this.id = id;
    }

    public int getScore()
    {
        return score;
    }

    public void setScore(int score)
    {
        this.score = score;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    @Override
    public String toString()
    {
        return "Student [id=" + id + ", score=" + score + ", name=" + name + "]";
    }
}
```

```java
public class TestSort
{
    public static void main(String[] args)
    {
        System.out.println("----init students----");
        Student[] students = new Student[4];
        students[0] = new Student(4, 10, "Tom");
        students[1] = new Student(2, 50, "Alice");
        students[2] = new Student(3, 30, "Tony");
        students[3] = new Student(1, 20, "Amy");
        Arrays.stream(students).forEach(x -> System.out.println(x));
        
        System.out.println("----sort by Comparator [by id]----");
        Arrays.sort(students, new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2)
            {
                return o1.getId() - o2.getId();
            }
        });
        Arrays.stream(students).forEach(x -> System.out.println(x));
    }
}
```

输出如下

```
----init students----
Student [id=4, score=10, name=Tom]
Student [id=2, score=50, name=Alice]
Student [id=3, score=30, name=Tony]
Student [id=1, score=20, name=Amy]
----sort by Comparator [by id]----
Student [id=1, score=20, name=Amy]
Student [id=2, score=50, name=Alice]
Student [id=3, score=30, name=Tony]
Student [id=4, score=10, name=Tom]
```

## 混合使用 Comparable 和 Comparator

单独使用 Comparable 或 Comparator 都能达到排序的效果，但是如果排序的对象实现了 Comparable  接口，同时在调用 Arrays.sort 函数时又传入了实现 Comparator 接口的比较对象，那结果会是怎样呢。实验证明，如果同时实现了这两个接口，将以 Comparator 的比较为准。

程序如下

```java
public class Student implements Comparable<Student>
{
    private int id;
    private int score;
    private String name;
    
    public Student(int id, int score, String name)
    {
        super();
        this.id = id;
        this.score = score;
        this.name = name;
    }
    
    public int getId()
    {
        return id;
    }

    public void setId(int id)
    {
        this.id = id;
    }

    public int getScore()
    {
        return score;
    }

    public void setScore(int score)
    {
        this.score = score;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    @Override
    public String toString()
    {
        return "Student [id=" + id + ", score=" + score + ", name=" + name + "]";
    }

    @Override
    public int compareTo(Student s)
    {
        return this.score - s.score;
    }
}
```

```java
public class TestSort
{
    public static void main(String[] args)
    {
        System.out.println("----init students----");
        Student[] students = new Student[4];
        students[0] = new Student(4, 10, "Tom");
        students[1] = new Student(2, 50, "Alice");
        students[2] = new Student(3, 30, "Tony");
        students[3] = new Student(1, 20, "Amy");
        Arrays.stream(students).forEach(x -> System.out.println(x));
        
        System.out.println("----sort by Comparator [by id]----");
        Arrays.sort(students, new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2)
            {
                return o1.getId() - o2.getId();
            }
        });
        Arrays.stream(students).forEach(x -> System.out.println(x));
    }
}
```

输出如下

```
----init students----
Student [id=4, score=10, name=Tom]
Student [id=2, score=50, name=Alice]
Student [id=3, score=30, name=Tony]
Student [id=1, score=20, name=Amy]
----sort by Comparator [by id]----
Student [id=1, score=20, name=Amy]
Student [id=2, score=50, name=Alice]
Student [id=3, score=30, name=Tony]
Student [id=4, score=10, name=Tom]
```

# 集合 Collections 排序

集合 Collections 排序和数组 Arrays 排序的方式类似，只不过集合的排序主要是依靠 `Collections.sort` 函数来完成。集合排序主要有两种方式，一种是由对象实现`Comparable ` 接口中的 `compareTo` 函数。另一种是在调用 `Collections.sort` 函数时将实现 `Comparator ` 接口的 `compare` 函数的对象作为参数传入该函数进行比较。

## 使用 Comparable 实现排序

使用 Comparable 的方式进行排序，要求排序的对象实现 Comparable 接口，并实现接口中的 compareTo 函数。此函数将本对象与另一个对象进行比较，如果本对象小于另一个对象，则返回负数，如果本对象大于另一个对象，则返回正数，相等则返回 0。

程序如下

```java
public class Student implements Comparable<Student>
{
    private int id;
    private int score;
    private String name;
    
    public Student(int id, int score, String name)
    {
        super();
        this.id = id;
        this.score = score;
        this.name = name;
    }
    
    public int getId()
    {
        return id;
    }

    public void setId(int id)
    {
        this.id = id;
    }

    public int getScore()
    {
        return score;
    }

    public void setScore(int score)
    {
        this.score = score;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    @Override
    public String toString()
    {
        return "Student [id=" + id + ", score=" + score + ", name=" + name + "]";
    }

    @Override
    public int compareTo(Student s)
    {
        return this.score - s.score;
    }
}
```

```java
public class TestSort
{
    public static void main(String[] args)
    {
        System.out.println("----init students----");
        List<Student> students = new ArrayList<>();
        students.add(new Student(1, 10, "Tom"));
        students.add(new Student(2, 50, "Alice"));
        students.add(new Student(3, 30, "Tony"));
        students.add(new Student(4, 20, "Amy"));
        students.forEach(x -> System.out.println(x));
        
        System.out.println("----sort by Comparable [by score]----");
        Collections.sort(students);
        students.forEach(x -> System.out.println(x));
    }
}
```

输出如下

```
----init students----
Student [id=1, score=10, name=Tom]
Student [id=2, score=50, name=Alice]
Student [id=3, score=30, name=Tony]
Student [id=4, score=20, name=Amy]
----sort by Comparable [by score]----
Student [id=1, score=10, name=Tom]
Student [id=4, score=20, name=Amy]
Student [id=3, score=30, name=Tony]
Student [id=2, score=50, name=Alice]
```

## 使用 Comparator 实现排序

使用 Comparator 方式进行排序，不要求排序对象实现 Comparable  接口，只需要在调用 Collections.sort 函数时将将实现 Comparator  接口的 compare 函数的对象作为参数传入该函数进行比较即可。compare 函数接收两个排序类型的参数，如果第一个对象对象小于第二个对象，则返回负数，如果第一个对象大于第二个对象，则返回正数，相等则返回 0。由于 java8 中新增了对 lambda 表达式的支持，因此，可以直接用 lambda 表达式实现比较对象并将其作为参数传入 Collection.sort 函数中。

程序如下

```java
public class Student
{
    private int id;
    private int score;
    private String name;
    
    public Student(int id, int score, String name)
    {
        super();
        this.id = id;
        this.score = score;
        this.name = name;
    }
    
    public int getId()
    {
        return id;
    }

    public void setId(int id)
    {
        this.id = id;
    }

    public int getScore()
    {
        return score;
    }

    public void setScore(int score)
    {
        this.score = score;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    @Override
    public String toString()
    {
        return "Student [id=" + id + ", score=" + score + ", name=" + name + "]";
    }
}
```

```java
public class TestSort
{
    public static void main(String[] args)
    {
        System.out.println("----init students----");
        List<Student> students = new ArrayList<>();
        students.add(new Student(1, 10, "Tom"));
        students.add(new Student(2, 50, "Alice"));
        students.add(new Student(3, 30, "Tony"));
        students.add(new Student(4, 20, "Amy"));
        students.forEach(x -> System.out.println(x));
        
        // 使用外部比较器，方式一：显式调用 Collections排序
        System.out.println("----sort by Comparator [by id]----");
        Collections.sort(students, new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2)
            {
                return o1.getId() - o2.getId();
            }
        });
        students.forEach(x -> System.out.println(x));
        
        // 使用外部比较器，方式二：隐式调用 Collections排序
        System.out.println("----sort by Comparator [by name]----");
        students.sort(new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2)
            {
                return o1.getName().compareTo(o2.getName());
            }
        });
        students.forEach(x -> System.out.println(x));
        
        // 使用外部比较器，方式三：隐式调用 Collections排序，用 lambda 表达式实现 Comparator 接口
        System.out.println("----sort by lambda [by id]----");
        students.sort((x, y) -> x.getId() - y.getId());
        students.forEach(x -> System.out.println(x));
    }
}
```

输出如下

```
----init students----
Student [id=1, score=10, name=Tom]
Student [id=2, score=50, name=Alice]
Student [id=3, score=30, name=Tony]
Student [id=4, score=20, name=Amy]
----sort by Comparator [by id]----
Student [id=1, score=10, name=Tom]
Student [id=2, score=50, name=Alice]
Student [id=3, score=30, name=Tony]
Student [id=4, score=20, name=Amy]
----sort by Comparator [by name]----
Student [id=2, score=50, name=Alice]
Student [id=4, score=20, name=Amy]
Student [id=1, score=10, name=Tom]
Student [id=3, score=30, name=Tony]
----sort by lambda [by id]----
Student [id=1, score=10, name=Tom]
Student [id=2, score=50, name=Alice]
Student [id=3, score=30, name=Tony]
Student [id=4, score=20, name=Amy]
```

## 混合使用 Comparable 和 Comparator

与数组 Arrays 排序一样，如果同时实现了这两个接口，将以 Comparator 的比较为准。

程序如下

```java
public class Student implements Comparable<Student>
{
    private int id;
    private int score;
    private String name;
    
    public Student(int id, int score, String name)
    {
        super();
        this.id = id;
        this.score = score;
        this.name = name;
    }
    
    public int getId()
    {
        return id;
    }

    public void setId(int id)
    {
        this.id = id;
    }

    public int getScore()
    {
        return score;
    }

    public void setScore(int score)
    {
        this.score = score;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    @Override
    public String toString()
    {
        return "Student [id=" + id + ", score=" + score + ", name=" + name + "]";
    }

    @Override
    public int compareTo(Student s)
    {
        return this.score - s.score;
    }
}
```

```java
public class TestSort
{
    public static void main(String[] args)
    {
        System.out.println("----init students----");
        List<Student> students = new ArrayList<>();
        students.add(new Student(1, 10, "Tom"));
        students.add(new Student(2, 50, "Alice"));
        students.add(new Student(3, 30, "Tony"));
        students.add(new Student(4, 20, "Amy"));
        students.forEach(x -> System.out.println(x));
        
        // 使用外部比较器，方式一：显式调用 Collections排序
        System.out.println("----sort by Comparator [by id]----");
        Collections.sort(students, new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2)
            {
                return o1.getId() - o2.getId();
            }
        });
        students.forEach(x -> System.out.println(x));
        
        // 使用外部比较器，方式二：隐式调用 Collections排序
        System.out.println("----sort by Comparator [by name]----");
        students.sort(new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2)
            {
                return o1.getName().compareTo(o2.getName());
            }
        });
        students.forEach(x -> System.out.println(x));
        
        // 使用外部比较器，方式三：隐式调用 Collections排序，用 lambda 表达式实现 Comparator 接口
        System.out.println("----sort by lambda [by id]----");
        students.sort((x, y) -> x.getId() - y.getId());
        students.forEach(x -> System.out.println(x));
    }
}
```

输出如下

```java
----init students----
Student [id=1, score=10, name=Tom]
Student [id=2, score=50, name=Alice]
Student [id=3, score=30, name=Tony]
Student [id=4, score=20, name=Amy]
----sort by Comparator [by id]----
Student [id=1, score=10, name=Tom]
Student [id=2, score=50, name=Alice]
Student [id=3, score=30, name=Tony]
Student [id=4, score=20, name=Amy]
----sort by Comparator [by name]----
Student [id=2, score=50, name=Alice]
Student [id=4, score=20, name=Amy]
Student [id=1, score=10, name=Tom]
Student [id=3, score=30, name=Tony]
----sort by lambda [by id]----
Student [id=1, score=10, name=Tom]
Student [id=2, score=50, name=Alice]
Student [id=3, score=30, name=Tony]
Student [id=4, score=20, name=Amy]
```

# 总结

java 中排序主要是通过对排序对象实现 Comparable 接口或者在调用排序函数时传入一个实现 Comparator 接口的比较对象来实现的。这两种方法各有优点，其中 Comparable 接口可以看出是对排序对象的一种默认实现，在调用排序函数时只需要传入排序对象集合即可得到排序结果，但是一种类型的对象只能有一种默认实现的排序方法。相比而言， Comparator 接口的排序方式显得更加灵活，其可以不受排序对象限制（无论排序对象有没有默认的排序实现），可以无限制地实现多种不同的排序方式，而且，即使排序对象有默认的排序行为（实现了 Comparable 接口），也可以通过外部实现 Comparator 接口的方式改变其排序行为。

因此，对于需要多种不同的排序方式的对象，建议对排序对象实现 Comparable 接口，为其赋有默认的排序行为，对其他的排序方式则通过外部实现 Comparator 接口的方式来实现。当然，如果某些情况下不允许改变排序对象，那就只能通过外部实现 Comparator 接口的方式来实现排序了。