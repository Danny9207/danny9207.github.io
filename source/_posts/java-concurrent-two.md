---
title: java并发-并发编程基础
date: 2018-09-13 21:29:13
tags:
---

##java并发编程基础

### 1. 线程简介

线程都拥有各自的计数器、堆栈和局部变量等属性，并且能够访问共享的内存变量。

<!-- more -->

* 线程状态：

NEW：初始状态，线程被构建，但是还没有调用`start()`方法。
RUNNABLE：运行状态，java将操作系统中的“就绪”和“运行”两种状态统称为“运行中”。此时处于`Thread.start()`方法调用之后。
处于WAITING，TIME_WAITING状态的线程在调用`Object.notify()`，`Object.notifyAll()`，`LockSupport.unpark(Thread)`方法之后，也会进入RUNNABLE状态。
BLOCKED：阻塞状态，表示线程阻塞于锁。此时线程等待进入`synchronized`代码块或方法。获取到锁后进入RUNNABLE状态。
WAITING：等待状态，进入该状态表示当前线程需要其他线程做出一些特定的动作（通知或中断），此时处于`Object.wait()`，`Object.join()`或`LockSupport.park()`方法调用之后。
TIME_WAITING：超时等待状态，它与WAITING不同，它可以在指定的时间自行返回。此时处于`Thread.sleep(long)`，`Object.wait(long)`，
`Thread.join(long)`，`LockSupport.parkNanos()`，`LockSupport.parkUntil()`方法调用之后。
TERMINATED：终止状态，表示线程已经执行完毕。
![](java-concurrent-two/thread-status.png)

### 2. 启动和终止线程

### 3. 线程间通信

### 4. 线程应用实例