---
title: java并发-java内存模型
date: 2018-09-03 21:33:54
tags:
---
## java内存模型

### &emsp;内存模型

JMM定义了线程和主内存之间的抽象关系：<br/>
线程之间的共享变量存储在主内存中，每一个线程都有一个私有的本地内存，
本地内存存储了该线程用来读/写共享变量的副本。<br/>
JMM通过控制主内存与每个线程的本地内存之间的交互，来为java程序提供内存可见性的保证。
### &emsp;volatile

#### &emsp;&emsp;特性

* 可见性。对一个volatile变量的读，总是能看到对这个volatile变量最后的写入。
* 原子性。对于单个volatile变量的读写具有原子性，但类似于volatile++的复合操作不具有原子性。

#### &emsp;&emsp;写-读的内存语义



#### &emsp;&emsp;内存语义实现原理

### &emsp;synchronized

#### &emsp;&emsp;特性
#### &emsp;&emsp;释放-获取的内存语义
#### &emsp;&emsp;内存语义实现原理

### &emsp;&emsp;final
