---
title: java8中HashMap实现原理
date: 2018-06-03 15:54:02
tags: java
categories: java
---

# 概述

HashMap是常用的数据结构，在java8中，其底层是用数组、链表、红黑树结合实现的。在新建一个HashMap时会初始化一个指定大小的数组，在插入元素的过程中，会通过key定位到数组的某个位置上，然后根据链表或红黑树的数据结构进行插入。当数组中已使用的位置达到某个阈值时，会对数组进行扩容。HashMap的数据结构大致如下图所示

<!-- more -->

![HashMap数据结构](/images/hashmap.png)

本文主要分析put、get和remove方法。

# 一些成员变量

```java
// 默认初始化数组的大小为16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

// 数组大小的最大值
static final int MAXIMUM_CAPACITY = 1 << 30;

// 默认负载因子，主要用于计算threshold的值
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 将链表转为红黑树的节点数阈值，当超过这个值时，会将链表转为红黑树
static final int TREEIFY_THRESHOLD = 8;

// 将红黑树转为链表的节点数阈值，当小于这个值时，会将红黑数转为链表
static final int UNTREEIFY_THRESHOLD = 6;

// 数组，连接红黑树和链表
transient Node<K,V>[] table;

// 判断数组是否要扩容的阈值 threshold = loadFactor * capacity
int threshold;

// 负载因子，用于计算threshold
final float loadFactor;

// HashMap中元素的个数
transient int size;
```

# put方法

## 大致流程

```flow
start=>start: Start
hash=>operation: 对key进行hash求值
isTableEmpty=>condition: 数组未初始化
initTable=>operation: 初始化数组
isFindNodeInArray=>condition: 在数组位置上
							找到相同key
							的节点
isRedBlackTree=>condition: 数组位置关联
						 的是红黑树
handleInRedBlackTree=>operation: 在红黑树中执行
							  节点查找插入操作
isFindNode=>condition: 找到相同
					 key的节点
handleReplacement=>operation: 根据需要替换对应的value
isToReSize=>condition: map元素大小
					 达到阈值
resize=>operation: 扩容
isFindInList=>condition: 在链表中找到
					   相同key节点
insertNodeInList=>operation: 在链表尾部插入节点
isChangeToTree=>condition: 链表节点数是
						 否超过阈值
changeListToTree=>operation: 将链表转为红黑树
end=>end

start->hash->isTableEmpty(yes)->initTable->isFindNodeInArray(yes,right)->isFindNode(yes)->handleReplacement->isToReSize(yes)->resize->end
isTableEmpty(no)->isFindNodeInArray(no)->isRedBlackTree(yes)->handleInRedBlackTree->isFindNode
isRedBlackTree(no)->isFindInList(no)->insertNodeInList->isChangeToTree(yes)->changeListToTree(right)->isFindNode
isFindInList(yes)->isFindNode
isFindNode(no)->isToReSize
isToReSize(no)->end
```

## 代码

### putVal函数

```java
public V put(K key, V value) {
    // 首先对key取hash定位到对应的数组的位置，然后调用putVal函数放置数据
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 如果数组并未初始化，则调用resize函数初始化数组，resize函数后面解析
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 如果hash(key)对应的数组位置上没有节点（数据）
    // 则直接新建一个节点，放在这个位置上
    // 否则，根据是链表还是红黑树执行对应的查找插入操作
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        // hash(key)在数组上能找到位置，属于hash冲突
        // 此时，需要查找这个位置关联的链表或红黑树中是否有相同key的节点
        // 如果有，则根据需要替换value的值
        // 如果没有，则根据链表或红黑树执行对应的插入操作
        Node<K,V> e; K k;
        // 数组位置上的节点的key值与插入的key值相同
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果数组位置上关联的是红黑树，则执行红黑树的查找插入操作
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 如果数组位置上关联的是链表，则执行链表的查找插入操作
            for (int binCount = 0; ; ++binCount) {
                // 遍历到链表最后，没找到相同key的节点，此时插入新节点
                if ((e = p.next) == null) {
                    // 在链表尾部插入新的节点
                    p.next = newNode(hash, key, value, null);
                    // 如果链表上的节点数达到阈值，则将其转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                // 在链表中找到相同key值的节点
                p = e;
            }
        }
        // 如果找打了相同key值的节点，则根据需要替换对应的value值
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 如果HashMap中元素个数超过阈值，则进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

### resize函数

resize函数是HashMap扩容的函数，扩容的主要流程为：

1. 将capacity和threshold进行扩容，一般情况下是原先的两倍。
2. 根据新的capacity新建数组（一般情况下为原先长度的两倍），将table指向新建的数组
3. 对原先数组及其关联的链表、红黑树的节点进行重分布。将一些节点留在 i （i 为节点所在的原先数组位置）位置上，另一些节点留在 i + oldCapacity 上（i 为节点所在的原先数组位置，oldCapacity为原先数组长度）

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 如果数组大小已经超过MAXIMUM_CAPACITY，则将阈值设为整形最大值，不进行扩容
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 对数组进行扩容，大小变为原先的两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    // 对应初始化HashMap(capacity)的情况
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    // 对应初始花HashMap()的情况
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    // 新建数组长度为原先的两倍，将table指向新建的数组
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 对原先数组中的节点进行重分布
    // 一半节点在原先数组的位置上
    // 另一半节点分布到数组的 i + oldCapacity位置上
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 数组位置上只有一个节点，直接放到新的数组上
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 数组位置上关联的是红黑树，执行红黑树的重分布
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // 数组位置上关联的是链表，执行链表的重分布
                // 重分布会将原先的链表分成两个链表
                // 一个链表在新数组的 i 位置上（i指原先在数组的位置）
                // 另一个链表在新数组的 i + oldCapacity位置上（oldCapacity指原先数组长度）
                // 重分布后，在每个链表内部，节点的顺序跟原先的保持一致
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

### treeifyBin函数

这是将链表转换为红黑树的函数。在转换之前，会将节点的数据结构由Node转为TreeNode，此转换过程中并不会改变原先链表的顺序，这也是方便重分布红黑树和后续将红黑树转为链表时仍能维持原先的顺序。

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            // 将链表的数据结构Node转为红黑树的数据结构TreeNode
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                // 保持原先链表的节点顺序，方便重分布红黑树和后续由红黑树转为链表时仍能维持节点顺序
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            // 将链表转为红黑树，执行的是红黑树的插入操作
            hd.treeify(tab);
    }
}
```

### split函数

这个函数主要是对红黑树进行重分布。由于将链表转为红黑树的过程中维持者链表节点的顺序，因此此处重分布红黑树也是先执行链表的重分布步骤将节点以链表的形式分散在数组两个位置上，然后对这两个位置的链表以插入的方式构造红黑树。

```java
final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
    TreeNode<K,V> b = this;
    // Relink into lo and hi lists, preserving order
    TreeNode<K,V> loHead = null, loTail = null;
    TreeNode<K,V> hiHead = null, hiTail = null;
    int lc = 0, hc = 0;
    // 红黑树内部仍然维持者链表的节点顺序
    // 此处按照重分布链表的方式，将节点分布在数组的两个位置上
    for (TreeNode<K,V> e = b, next; e != null; e = next) {
        next = (TreeNode<K,V>)e.next;
        e.next = null;
        if ((e.hash & bit) == 0) {
            if ((e.prev = loTail) == null)
                loHead = e;
            else
                loTail.next = e;
            loTail = e;
            ++lc;
        }
        else {
            if ((e.prev = hiTail) == null)
                hiHead = e;
            else
                hiTail.next = e;
            hiTail = e;
            ++hc;
        }
    }

    // 处理 i 位置上的红黑树
    if (loHead != null) {
        // 如果红黑树的节点数小于阈值，则将红黑树转为链表
        if (lc <= UNTREEIFY_THRESHOLD)
            tab[index] = loHead.untreeify(map);
        else {
            // 将链表安装红黑树插入方式转为红黑树
            tab[index] = loHead;
            if (hiHead != null) // (else is already treeified)
                loHead.treeify(tab);
        }
    }
    // 处理 i + oldCapacity位置上的红黑树
    if (hiHead != null) {
        // 如果红黑树的节点数小于阈值，则将红黑树转为链表
        if (hc <= UNTREEIFY_THRESHOLD)
            tab[index + bit] = hiHead.untreeify(map);
        else {
            // 将链表安装红黑树插入方式转为红黑树
            tab[index + bit] = hiHead;
            if (loHead != null)
                hiHead.treeify(tab);
        }
    }
}
```

###  untreeify函数

此函数是将红黑树转为链表的函数，由于将链表转为红黑树的过程中，替换了节点的数据结构，但是链表的节点顺序并为改变，因此，在将红黑树转为链表的过程中，只需将节点的数据结构由TreeNode转为Node即可。

```java
final Node<K,V> untreeify(HashMap<K,V> map) {
    Node<K,V> hd = null, tl = null;
    for (Node<K,V> q = this; q != null; q = q.next) {
        // 由于将链表转为红黑树的过程中，维持者链表节点顺序
        // 此处只需要将节点数据结构由TreeNode转为Node即可
        Node<K,V> p = map.replacementNode(q, null);
        if (tl == null)
            hd = p;
        else
            tl.next = p;
        tl = p;
    }
    return hd;
}
```

# get方法

get方法相对而言比较简单，主要是先计算hash(key)的值，然后进行查找操作。

## 大致流程

```flow
start=>start: Start
hash=>operation: 对key进行hash求值
isFindInArray=>condition: 在数组位置上找
						到相同key的节点
returnValue=>operation: 返回节点值
returnNull=>operation: 返回空
isRedBlackTree=>condition: 数组位置
						 关联红黑树
findInRedBlackTree=>operation: 在红黑树中查找节点
isFindNode=>condition: 找到相同key节点
findInList=>operation: 在链表中查找节点
end=>end: End

start->hash->isFindInArray(no, left)->isRedBlackTree(yes)->findInRedBlackTree->isFindNode(yes)->returnValue->end
isFindNode(no)->returnNull->end
isFindInArray(yes)->isFindNode
isRedBlackTree(no)->findInList->isFindNode
```

## 代码

```java
public V get(Object key) {
    Node<K,V> e;
    // 先计算hash(key),再执行查找
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 数组位置上的节点key值相同，则直接返回数组位置节点
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // 数组位置上关联的是红黑树，执行红黑树的查找操作
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 数组位置上关联的是链表，执行链表的查找操作
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

# remove方法

## 大致流程

```flow
start=>start: Start
hash=>operation: 对key进行hash求值
isFindInArray=>condition: 在数组位置上找
						到相同key的节点
isRedBlackTree=>condition: 数组位置
						 关联红黑树
findInRedBlackTree=>operation: 在红黑树中查找节点
isFindNode=>condition: 找到相同key节点
findInList=>operation: 在链表中查找节点
isRBT=>condition: 数组位置
				关联红黑树
deleteNodeInRBT=>operation: 执行红黑树删除节点操作
deleteNodeInList=>operation: 执行链表删除操作
end=>end: End

start->hash->isFindInArray(no, left)->isRedBlackTree(yes)->findInRedBlackTree->isFindNode(yes)->isRBT(yes)->deleteNodeInRBT->end
isFindNode(no)->end
isFindInArray(yes)->isFindNode
isRedBlackTree(no)->findInList->isFindNode
isRBT(no)->deleteNodeInList->end
```

## 代码

```java
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        // 先查找要删除的节点
        Node<K,V> node = null, e; K k; V v;
        // 数组位置上节点key值相同，即为要删除的节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            // 数组位置关联的是红黑树，执行红黑树的查找操作
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                // 数组位置关联的是链表，执行链表的查找操作
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        // 如果相同key值的节点，则执行删除操作
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            // 数组位置关联的是红黑树，执行红黑树的删除操作
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            // 数组位置节点是要删除的节点，删除数组位置节点，将数组位置指针指向链表下一个节点
            else if (node == p)
                tab[index] = node.next;
            // 数组位置节点关联的是链表，执行链表的删除操作
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

# 其他

## HashMap是非线程安全的

在 put 方法中，查找链表到尾部时，如果仍然找不到 key 值相同的元素，则在尾部插入新节点。假设线程A要插入（a, a）的键值对，线程B要插入（b, b）的键值对，再假设 a、b 这两个 key 会映射到同一个 table[i] 上，并且原先都没有插入过 key 为 a 或 b 的键值对；当线程A遍历到链表尾部时，在准备（下一步）插入新节点的时候，线程切换到B执行，B也遍历到链表尾部，并插入新节点（b, b），然后线程切换到A，A插入新节点（a, a），此时会造成插入的（b, b）节点被覆盖丢失。

## 疑问

在查找节点和删除节点这两个操作中，都涉及了查找节点的过程，但是在remove的查找节点的过程中并不是调用get方法中调用的`getNode`函数，而是重新写了类似的查找过程，这让我很疑惑。不管怎样，不可否认，java8中HashMap的源码是优秀的。关于疑问就先留着以后解决吧。