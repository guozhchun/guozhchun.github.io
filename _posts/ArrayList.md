---
title: java8中ArrayList实现原理
date: 2018-06-14 22:15:26
tags: java
categories: java
---

# 概述

ArrayList是常用的数据结构，在java8中，其主要是通过数组来实现的。在插入一个元素时会先判断数组容量是否还能放置元素，如果不能，则进行扩容。本文主要分析add、get、set、remove方法。

<!-- more -->

# 一些成员变量

```java
// 默认的初始化数组大小
private static final int DEFAULT_CAPACITY = 10;

// 空数组，主要用于new ArrayList<>()的情况 
private static final Object[] EMPTY_ELEMENTDATA = {};

// 数组数据结构，用于存放ArrayList的元素
transient Object[] elementData; // non-private to simplify nested class access

// 当前ArrayList的元素个数
private int size;
```

# add方法

## add(E e)方法

### 大致流程

```flow
start=>start: Start
isArrayEmpty=>condition: 数组是否为空
reCalcMinCapacity=>operation: 重新计算minCapacity
						    大小,取
                              DEFAULT_CAPACITY
                              和minCapacity
                              的最大值
isResize=>condition: minCapacity大于
				    当前数组大小
resize=>operation: 扩容数组为原先的1.5倍
addObj=>operation: 往数组末尾中加入元素
end=>end: End

start->isArrayEmpty(yes)->reCalcMinCapacity->isResize(yes)->resize->addObj->end
isArrayEmpty(no)->isResize(no)->addObj
```

### 代码

```java
public boolean add(E e) {
    // 保证数组大小能够放置新元素，必要时进行扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 在数组末端增加新元素
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    // 如果数组是空数组，第一次增加元素时取默认初始大小
    if (elementData == EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 当前数组已经满了，要增加新元素，只能扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

// 数组扩容的函数
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 新的数组大小为原先数组大小的 1.5 倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 拷贝原先数组元素
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

## add(int index, E e)方法

这个方法与`add(E e)`方法类似，只不过在函数刚开始的地方多了个边界检查，在确保数组能够放置新元素后面增加了下标元素及其后面的元素向后移动一位的操作。

```java
public void add(int index, E element) {
    // 边界检查
    rangeCheckForAdd(index);
    // 确保数组能够放置新元素，必要时进行 1.5 倍大小的扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 从下标开始的元素均向后移动一位
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    // 在下标的位置放置新元素
    elementData[index] = element;
    size++;
}
```

# get方法

get方法很简单，就是检查下标的边界，然后直接取数组下标的元素返回。

```java
public E get(int index) {
    // 边界检查
    rangeCheck(index);
	// 返回数组下标所对应元素的值
    return elementData(index);
}

// 返回数组下标所对应元素的值
@SuppressWarnings("unchecked")
E elementData(int index) {
    return (E) elementData[index];
}
```

# set方法

set方法也很简单，也是先检查下标的边界，然后先取出原有下标的元素，再直接替换新的元素，最后将原先的元素返回。

```java
public E set(int index, E element) {
    // 边界检查
    rangeCheck(index);
	// 取出下标对应的老元素
    E oldValue = elementData(index);
    // 替换下标元素
    elementData[index] = element;
    return oldValue;
}
```

# remove方法

##  remove(int index)方法

根据下标删除元素，首先进行边界检查，然后取出下标对应的元素，再将这个元素后面的元素都向前移动一位，接着将数组最后一个元素置空以便垃圾回收器能够回收该对象，最后将下标指向的原先的对象返回。

```java
public E remove(int index) {
    // 边界检查
    rangeCheck(index);

    modCount++;
    // 取出原先下标对应的值
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 将下标后面的元素向前移动一位
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    // 将数组最后一个元素置空，以便垃圾回收器进行回收
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

## remove(Object o)方法

根据对象删除对象的方法主要是根据对象是否是空的进行分别处理。

* 删除的对象是`null`，则查找数组中的元素，如果有`null`元素，则将第一个`null`元素删除
* 删除的对象不是null，则根据`equals`方法将对象与数组中的元素进行比较，将第一个相等的元素删除

```java
public boolean remove(Object o) {
    if (o == null) {
        // 查找数组的null元素，找到时将第一个null元素删除
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        // 根据equals方法将对象与数组元素进行比较，将第一个相等的元素删除
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

// 将下标后面的元素向前移动一位，将数组最后一个元素置空，以便垃圾回收器进行回收
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 将下标后面的元素向前移动一位
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    // 将数组最后一个元素置空，以便垃圾回收器进行回收
    elementData[--size] = null; // clear to let GC do its work
}
```

# 其他

* ArrayList允许增加`null`对象
* ArrayList允许添加重复对象
* ArrayList对象是有序的，新增的对象在尾部
* ArrayList不是线程安全的，需要在外部调用的地方手动保证线程安全。因为在add函数中有size++的操作，当有两个线程同时调用add函数时，假设A线程已经在对应的位置上放置了元素，但是size还未加一，此时切换到B线程执行add函数直到结束，size加一了，切回A线程执行size++，此时A线程中的size值还是原先的没有经过B线程加一的值，在A线程执行完size++后，此时size只会加一而不会加二，从而导致数据丢失。