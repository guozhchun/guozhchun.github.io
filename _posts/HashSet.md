---
title: java8中HashSet实现原理
date: 2018-06-15 20:54:57
tags: java
categories: java
---

# 概述

HashSet是一种比较常见的数据结构，经常用它来进行一些重复数据的过滤。在java8中，其主要是通过HashMap来实现的。在放置数据时，将数据作为HashMap的key进行保存，相比其他数据结构（如HashMap, ArrayList）而言，这是一种比较简单的数据结构，因为其方法均委托给HashMap数据结构来实现了。本文主要分析HashSet常用的几个方法。

<!-- more -->

# 一些成员变量

```java
// 用HashMap数据结构来实现HashSet
private transient HashMap<E,Object> map;

// HashMap中value的值，是固定的
private static final Object PRESENT = new Object();
```

# size函数

直接调用map数据结构的size()函数获得结果

```java
public int size() {
    return map.size();
}
```

# isEmpty函数

直接调用HashMap数据结构的isEmpty函数获取结果

```java
public boolean isEmpty() {
    return map.isEmpty();
}
```

# contains函数

直接调用HashMap数据结构的containsKey函数获取结果

```java
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

# add函数

将要新增的元素作为HashMap的key值，将常量`PRESENT`作为value值，往map中插入数据。

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

# remove函数

直接调用HashMap数据结构的remove函数进行删除元素操作

```java
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
```

# clear函数

直接调用HashMap数据结构的clear函数得到结果

```java
public void clear() {
    map.clear();
}
```

# 其他

由于HashSet使用HashMap来实现的，所以HashSet也具有HashMap的一些特点。

* 元素是无序的
* 允许有且仅有一个`null`元素
* 非线程安全。HashSet的add函数调用了HashMap的put方法，而在HashMap的 put 方法中，查找链表到尾部时，如果仍然找不到 key 值相同的元素，则在尾部插入新节点。假设线程A要插入（a, a）的键值对，线程B要插入（b, b）的键值对，再假设 a、b 这两个 key 会映射到同一个 table[i] 上，并且原先都没有插入过 key 为 a 或 b 的键值对；当线程A遍历到链表尾部时，在准备（下一步）插入新节点的时候，线程切换到B执行，B也遍历到链表尾部，并插入新节点（b, b），然后线程切换到A，A插入新节点（a, a），此时会造成插入的（b, b）节点被覆盖丢失。