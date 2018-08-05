---
title: ArrayList一些注意事项
date: 2018-07-26 22:42:05
tags: java
categories: java
---

# 前言

ArrayList 是 java 中最常用的也是最基本的集合类，但是其中一部分函数如果不注意就会很容易出错。

<!-- more -->

# remove函数

java5 后引入了自动封箱和自动拆箱的语法糖，一方面简化了程序，但是另一方面，在对支持`remove(int index)`又支持`remove(Object o)`的 ArrayList 类的方法调用上，就很容易疑惑，也很容易出错。如果调用`remove`函数时传入的是数字 1 ，那调用的是 `remove(int index)`函数还是`remove(Object o)`函数呢？答案可以从下面的程序及输出结果中得知。

```java
public void testRemove()
{
    List<Integer> nums = new ArrayList<>();
    nums.add(1);
    nums.add(2);
    nums.add(3);
    nums.add(4);
    System.out.println("----before remove----");
    System.out.println(nums.toString());

    nums.remove(1);
    System.out.println("----after remove(1)");
    System.out.println(nums.toString());

    nums.remove((Integer) 1);
    System.out.println("----after remove((Integer) 1)----");
    System.out.println(nums.toString());
}
```

输出如下

```
----before remove----
[1, 2, 3, 4]
----after remove(1)----
[1, 3, 4]
----after remove((Integer) 1)----
[3, 4]
```

从上述程序输出可以得知，当调用 remove 函数时，如果输入的是数字或 int 类型，则调用 `remove(int index)`函数，而如果传入的是  Integer 类型或将数字强制转换为 Integer 类型，则调用的是`remove(Object o)`函数。

更近一步，推而广之，如果一个类中存在重载函数，既有基本类型的函数，也有封装类型的函数，当传入数字或基本类型变量时，优先调用的基本类型的函数，而不是封装类型的函数，只有明确将传入的基本类型转换成封装类型或传入封装类型的参数变量，才会调用封装类型的函数。

# subList

subList 函数是返回 ArrayList 集合中的一个子序列，但是需要注意以下几点：

* subList 是原有 ArrayList 集合中的一个"指针"的集合，对 subList 做出的改动会同步反映到原有的 ArrayList 集合中，同时对原有的 ArrayList 集合的非改变集合大小的操作也都会直接同步反映到 subList 集合中
* 对原有的 ArrayList 集合的add、remove等改变集合大小的操作，会使原有的 subList 失效，即再调用 subList 的操作会抛出`ConcurrentModificationException`异常

如下程序及其输出证明了上述特点

```java
public void testSubList()
{
    ArrayList<Integer> nums = new ArrayList<>();
    nums.add(1);
    nums.add(2);
    nums.add(3);
    nums.add(4);

    // subList is [2, 3]
    List<Integer> subList = nums.subList(1, 3);
    System.out.println("----init-----");
    System.out.println("nums: " + nums.toString());
    System.out.println("subList: " + subList.toString());

    // 对subList的remove操作会反映到原有的集合中
    subList.remove(0);
    System.out.println("----after subList.remove(0)----");
    System.out.println("nums: " + nums.toString());
    System.out.println("subList: " + subList.toString());

    // 对subList的add操作会反映到原有的集合中
    subList.add(2);
    System.out.println("----after subList.add(2)----");
    System.out.println("nums: " + nums.toString());
    System.out.println("subList: " + subList.toString());

    // 对subList的remove操作会反映到原有的集合中
    subList.remove((Integer) 2);
    System.out.println("----after subList.remove((Integer) 2)----");
    System.out.println("nums: " + nums.toString());
    System.out.println("subList: " + subList.toString());

    // 对subList的set操作会反映到原有的集合中
    subList.set(0, 333);
    System.out.println("----after subList.set(0, 333)----");
    System.out.println("nums: " + nums.toString());
    System.out.println("subList: " + subList.toString());

    // 对原有集合的set操作会反映到subList集合中
    nums.set(1, 3);
    System.out.println("----after nums.set(1, 3)----");
    System.out.println("nums: " + nums.toString());
    System.out.println("subList: " + subList.toString());

    // 不能对原有的集合进行add或remove等会改变集合大小的操作
    // 否则再次对subList集合进行操作时会抛 ConcurrentModificationException 异常
    // nums.remove(0);
    // System.out.println(subList.toString());
}
```

程序输出如下

```
----init-----
nums: [1, 2, 3, 4]
subList: [2, 3]
----after subList.remove(0)----
nums: [1, 3, 4]
subList: [3]
----after subList.add(2)----
nums: [1, 3, 2, 4]
subList: [3, 2]
----after subList.remove((Integer) 2)----
nums: [1, 3, 4]
subList: [3]
----after subList.set(0, 333)----
nums: [1, 333, 4]
subList: [333]
----after nums.set(1, 3)----
nums: [1, 3, 4]
subList: [3]
```

至于 subList 为什么有以上的特点，这可以从 ArrayList 的源码中得知答案。

ArrayList 中 subList 返回的是一个内部类`SubList`，从内部类的构造函数可以看出，当创建一个 SubList 时，会将 ArrayList 对象及其部分参数传入构造函数中。当调用 subList 中的 set、get、add、remove 操作都是直接操作原有的 ArrayList 集合，所以修改 subList 对象会导致原有的 ArrayList 对象。

而对于修改原有的 ArrayList 对象的情况，则需要分两种情况考虑，一种是不修改集合大小的操作（即不改变 modCount 的操作，如 get、set 函数），这种情况由于没有修改 modCount 变量的值，所以下次调用 subList 对象的函数进行校验时仍然满足 modCount 变量值相等的条件，不会抛出 ConcurrentModificationException 异常。另一种是会修改集合大小的操作（即改变 modCount 的操作，如 add、remove 函数），这种情况由于修改了原有的 ArrayList 对象中的 modCount 变量值而没有修改 subList 变量中的 modCount 变量值，所以下次再调用 subList 对象的函数时，由于两者的 modCount 变量不相等，会抛出ConcurrentModificationException 异常。

```java
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);
    return new SubList(this, 0, fromIndex, toIndex);
}

// SubList 内部类
private class SubList extends AbstractList<E> implements RandomAccess {
    // ... 其他代码 ...
    // 构造函数
    SubList(AbstractList<E> parent,
            int offset, int fromIndex, int toIndex) {
        this.parent = parent;
        this.parentOffset = fromIndex;
        this.offset = offset + fromIndex;
        this.size = toIndex - fromIndex;
        // 此参数用于记录改变集合大小的操作次数，
        // 当 SubList 中的参数与原有的 ArrayList 集合中的参数不相等时，
        // 再次对 SubList 操作会抛出 ConcurrentModificationException 异常
        this.modCount = ArrayList.this.modCount;
    }

    public E set(int index, E e) {
        rangeCheck(index);
        // 校验 SubList 和原有 ArrayList 集合中的 modCount 变量是否相等，
        // 不相等则抛出 ConcurrentModificationException 异常
        checkForComodification();
        // 直接操作的是原有的 ArrayList 集合中对应的变量
        E oldValue = ArrayList.this.elementData(offset + index);
        ArrayList.this.elementData[offset + index] = e;
        return oldValue;
    }

    public E get(int index) {
        rangeCheck(index);
        // 校验 SubList 和原有 ArrayList 集合中的 modCount 变量是否相等，
        // 不相等则抛出 ConcurrentModificationException 异常
        checkForComodification();
        // 直接返回原有的 ArrayList 集合中对应的变量
        return ArrayList.this.elementData(offset + index);
    }

    // ... 其他代码 ...

    public void add(int index, E e) {
        rangeCheckForAdd(index);
        // 校验 SubList 和原有 ArrayList 集合中的 modCount 变量是否相等，
        // 不相等则抛出 ConcurrentModificationException 异常
        checkForComodification();
        // 直接在原有的 ArrayList 集合中对应的位置中增加元素
        parent.add(parentOffset + index, e);
        // 修改 modCount 的值，避免在下次对 SubList 进行操作时抛出异常
        this.modCount = parent.modCount;
        this.size++;
    }

    public E remove(int index) {
        rangeCheck(index);
        // 校验 SubList 和原有 ArrayList 集合中的 modCount 变量是否相等，
        // 不相等则抛出 ConcurrentModificationException 异常
        checkForComodification();
        // 直接删除原有的 ArrayList 集合中对应的位置的元素
        E result = parent.remove(parentOffset + index);
        // 修改 modCount 的值，避免在下次对 SubList 进行操作时抛出异常
        this.modCount = parent.modCount;
        this.size--;
        return result;
    }

    // ... 其他代码 ...

    private void checkForComodification() {
        if (ArrayList.this.modCount != this.modCount)
            throw new ConcurrentModificationException();
    }

    // ... 其他代码 ...
}
```

