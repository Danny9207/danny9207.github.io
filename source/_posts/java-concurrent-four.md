---
title: java并发-java并发容器和框架
date: 2018-10-10 21:56:31
tags:
---

## 并发容器和框架

<!-- more -->

### 1. ConcurrentHashMap的实现原理和使用

*HashMap在并发操作下引起死循环的原因：
在执行put()方法时会使Entry链表形成环形数据结构，一旦形成环形链表，Entry的next节点永远不为空，就会产生死循环获取Entry。
*

#### 结构

（java7）ConcurrentHashMap由Segment数组和HashEntry数组结构组成。Segment是一种可重入锁（ReentrantLock）;HashEntry用于存储键值对数据。
一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素。
每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组进行修改时，必须获得与它对应的Segment锁。

（java8）多了一种数据结构-红黑树，当链表长度大于8就会被转化为红黑树。

#### 初始化

（java7）initialCapacity-初始化ConcurrentHashMap容量为16；loadFactor-负载因子为0.75；ConcurrencyLevel-并发级别为16，理论上，如果线程的操作分布在不同的Segment上，
，则最多支持16个线程并发写。

（java8）

#### get方法

（java7）经过一次hash算法定位到Segment，再经过一次hash算法定位到HashEntry。
基于volatile的内存语义，所以get操作没有加锁。

（java8）计算hash值，根据hash值找到数组对应位置；根据该位置处的节点性质进行相应查找：
如果该位置为null，就返回null，如果找到对应值，就返回该节点的值，如果是正在扩容，或者是红黑树，在调用find方法，如果是链表，则进行遍历查找。

#### put方法

（java7）put方法首先定位到Segment，然后在Segment里进行插入操作。
插入操作有两个步骤：
一是判断是否需要对Segment里的HashEntry数组进行扩容；
二是定位添加元素的位置，然后将其放到HashEntry数组里。

（java8）1，初始化数组；2，找到hash值对应的数组下表，然后用CAS尝试放入新值；3，数据迁移；4，遍历及放入链表；5，调用红黑树并插入；6，链表转换为红黑树

### 2. ConcurrentLinkedQueue

### 3. java中的阻塞队列

### 4. Fork/Join框架