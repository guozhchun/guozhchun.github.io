---
title: java 自动装箱和拆箱
date: 2018-10-20 19:37:30
tags: java
categories: java
---

# 概念

自动装箱和自动拆箱是 java5 提供的一种语法糖，主要是为了方便程序员进行代码编写，在编译时会自动将语法糖去除还原成其真正的语法。

自动装箱：将基本类型转换为其包装类型。如将 `int` 转成 `Integer`。其主要是通过包装类中的 `valueOf` 函数完成的。

自动拆箱：将包装类型转成基本类型。如将 `Integer` 转成 `int`。其主要是通过包装类中的 `xxxValue`函数完成的，其中 `xxx`代表基本类型。例如 `Integer` 类中是 `intValue`，`Long` 类中是 `longValue`。

<!-- more -->

# 注意点

自动装箱和自动拆箱虽然极大地方便了程序员进行代码的编写，但是在使用过程中仍然需要注意以下几点，否则很容易造成错误。

* 基本类型作为函数形参，包装类型作为函数实参，需要保证函数实参是非空的，否则会报空指针异常。因为将一个包装类转为基本类型时，需要调用包装类的 xxxValue 函数，如果此时包装类是 null ，则调用该函数会抛空指针异常。
* 对于数字参与运算的情况，会自动执行拆箱过程，即将包装类型转成基本类型。
* 对于 `==` 的比较情况，如果两边中有任何一边是表达式（即包含算术运算），则会触发自动拆箱的过程，其比较的是比较的是数值，而不是对象地址
* 对于整数（Long、Integer、Short 类型），一般情况下，`-128 ~ 127` 这个范围的数字在使用自动装箱时，同一类型下，同一个数字的包装类型返回的是同一个对象。即 `Integer a = 3` 和 `Integer b = 3`，其得到的对象的地址是一样的，而对于 `Integer c = 1234567` 和 `Integer d = 1234567`，其得到的对象的地址是不同的。这主要是因为在 Long、Integer、Short 这几种类型中，其内部维护了一个缓存数组，放置着 `-128 ~ 127` 的包装类型，当外部调用 `valueOf` 函数时，如果其参数在这个范围内，则返回数组中已有的对象，否则新建一个对象返回。下面是 `Integer.valueOf` 函数的源码，从中可以证明上述结论。

```java
public static Integer valueOf(int i) {
    // 如果传入参数在缓存范围内，可返回缓存中已有的对象，否则新建一个对象返回
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}

private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        // 为缓存中预置对象
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

# 例子

以下例子参考《 深入理解Java虚拟机：JVM高级特性与最佳实践》而来。

```java
public class TestInteger
{
    public static void main(String[] args)
    {
        Integer a = 1;
        Integer b = 2;
        Integer c = 3;
        Integer d = 3;
        Integer e = 321;
        Integer f = 321;
        Long g = 3L;
        
        System.out.println("c == d: " + (c == d));
        System.out.println("e == f: " + (e == f));
        System.out.println("c == (a + b): " + (c == (a + b)));
        System.out.println("c.equals(a + b): " + (c.equals(a + b)));
        System.out.println("g == (a + b): " + (g == (a + b)));
        System.out.println("g.equals(a + b): " + (g.equals(a + b)));
    }
}
```

程序输出如下

```java
c == d: true
e == f: false
c == (a + b): true
c.equals(a + b): true
g == (a + b): true
g.equals(a + b): false
```

下面对输出结果进行解释。

首先 `c == d` 的输出结果：由于 c 和 d 都是对于 3 的封装类型，并且 3 在`-128 ~ 127` 这个缓存范围内，所以其指向的是同一个对象，所以返回 true。

对于 `e == f` 的输出结果：虽然 e 和 f 都是 321 的封装类型，但是由于 321 在`-128 ~ 127` 这个缓存范围外，每次对其进行装箱时都是新建一个对象，所以 e 和 f 实质上指向的是不同的对象，因此返回 false。

对于 `c == (a + b)` 的输出结果：由于 `==` 右边是一个运算表达式，所以会将 `==` 两边都自动拆箱进行数值之间的比较，所以返回的是 true。

对于 `c.equals(a + b)` 的输出结果：由于 `a + b` 包含运算符，所以会先将 a 和 b 进行拆箱，然后进行运算，最后将结果进行装箱与 c 进行比较，由于 `a + b` 运算得到的结果是 3，在`-128 ~ 127` 这个缓存范围内，所以其装箱后与 c 指向的是同一个对象，因此返回 true。

对于 `g == (a + b)`  的输出结果：由于 `==` 右边是一个运算表达式，所以会将 `==` 两边都自动拆箱进行数值之间的比较，虽然 `a + b` 得到的结果 `3` 是 `int` 类型，但是转成 `long` 类型后还是数字 3，与 g 的数值是相等的，因此返回 true。

对于 `g.equals(a + b)` 的输出结果：`a + b` 先拆箱进行运算，得到数值 `3`，由于数值是 `int` 类型，且不是进行 `==` 数值间的比较，不需要拆箱强制转换成 `long` 类型，因此其装箱得到的是 `Integer` 类型，此时再与 g 对象（`Long` 类型）进行比较，由于 `Intege` 不是 `Long` 类型，也不是其子类型，所以其返回 false。这可以从 `Long.equals` 中看出。

```java
public boolean equals(Object obj) {
    if (obj instanceof Long) {
        return value == ((Long)obj).longValue();
    }
    return false;
}
```

下面附上反编译后的代码，证明上述结论与分析

```java
public class TestInteger {

   public static void main(String[] var0) {
      Integer var1 = Integer.valueOf(1);
      Integer var2 = Integer.valueOf(2);
      Integer var3 = Integer.valueOf(3);
      Integer var4 = Integer.valueOf(3);
      Integer var5 = Integer.valueOf(321);
      Integer var6 = Integer.valueOf(321);
      Long var7 = Long.valueOf(3L);
      System.out.println("c == d: " + (var3 == var4));
      System.out.println("e == f: " + (var5 == var6));
      System.out.println("c == (a + b): " + (var3.intValue() == var1.intValue() + var2.intValue()));
      System.out.println("c.equals(a + b): " + var3.equals(Integer.valueOf(var1.intValue() + var2.intValue())));
      System.out.println("g == (a + b): " + (var7.longValue() == (long)(var1.intValue() + var2.intValue())));
      System.out.println("g.equals(a + b): " + var7.equals(Integer.valueOf(var1.intValue() + var2.intValue())));
   }
}
```

