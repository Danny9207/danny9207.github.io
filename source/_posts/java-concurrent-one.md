---
title: java并发-java内存模型
date: 2018-09-03 21:33:54
tags:
---
## java内存模型

### &emsp;内存模型

JMM定义了线程和主内存之间的抽象关系：
线程之间的共享变量存储在主内存中，每一个线程都有一个私有的本地内存，
本地内存存储了该线程用来读/写共享变量的副本。
JMM通过控制主内存与每个线程的本地内存之间的交互，来为java程序提供内存可见性的保证。

<!-- more -->

### &emsp;volatile

#### &emsp;&emsp;特性

* 可见性。对一个volatile变量的读，总是能看到对这个volatile变量最后的写入。
* 原子性。对于单个volatile变量的读写具有原子性，但类似于volatile++的复合操作不具有原子性。

#### &emsp;&emsp;写-读的内存语义

* 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存。
* 当读一个volatile变量时，JMM会把该行程对应的本地内存的值置为无效，线程接下来将从主内存中读取共享变量。

#### &emsp;&emsp;内存语义实现原理

为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。
下面是基于保守策略的JMM内存屏障插入策略：

* 在每一个volatile写操作的前面插入一个StoreStore屏障。
* 在每一个volatile写操作的后面插入一个StoreLoad屏障。
* 在每一个volatile读操作的后面插入一个LoadLoad屏障。
* 在每一个volatile读操作的后面插入一个LoadStore屏障。

### &emsp;synchronized

#### &emsp;&emsp;特性

锁可以让临界区互斥执行。

#### &emsp;&emsp;释放-获取的内存语义

* 当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中。
* 当线程获取锁时，JMM会把该线程对应的本地内存置为无效。从而使线程中的代码必须从主内存中读取共享变量。 

#### &emsp;&emsp;内存语义实现原理

1. 利用volatile变量的读-写所具有的的内存语义。
2. 利用CAS所附带的volatile读和volatile写的内存语义。

### &emsp;&emsp;final

*待完成（有可能完不成）* :sleeping: