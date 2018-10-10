---
title: java并发-java中的锁
date: 2018-09-19 20:32:25
tags:
---

## java中的锁

围绕两个方面：“使用”，“实现”。

<!-- more -->

### 1. Lock接口

Lock接口提供了synchronized所不具备的特性：

特性|描述
-|-
尝试非阻塞地获取锁|当线程尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取到锁。
能被中断的获取锁|与synchronized不同，获取到锁的线程能够响应中断，当获取到锁的线程被中断，中断异常会被抛出，同时锁会被释放。
超时获取锁|在指定的时间内获取锁，如果时间到了仍无法获取锁，则返回。

### 2. 队列同步器（QAS）

AbstractQueuedSynchronizer是用来构建锁或其他同步组件的基础框架，它使用一个int成员变量表示同步状态；通过内置的FIFO队列来完成资源获取线程的排队工作。

* 使用

自定义同步组件（CustomLock）需要聚合一个AQS的子类（SpecificSynchronizer）。
SpecificSynchronizer可重写的方法：

方法名称|描述
-|-
protected boolean tryAcquire (int arg)|独占式获取同步状态。实现该方法需要查询当前状态并判断同步状态是否符合预期，然后再进行 CAS 设置同步状态。
protected boolean tryRelease (int arg)|独占式释放同步状态。等待获取同步状态的线程将有机会获取同步状态。
protected int tryAcquireShared (int arg)|共享式获取同步状态。返回大于等于0的值，表示获取成功，反之获取失败。
protected boolean tryReleaseShared (int arg)|共享式释放同步状态。
protected boolean isHeldExclusively()|当前同步器是否在独占模式卜被线程占用，一般该方法表示是否被当前线程所独占。

CustomLock可调用的SpecificSynchronizer提供的部分模板方法如下：

方法名称|描述
-|-
void acquire (int arg)|独占式获取同步状态。如果当前线程获取同步状态成功，则由该方法返回；否则，将会进入同步队列等待，该方法将会调用重写的tryAcquire(int arg)方法。
void acquireInterruptibly (int arg)|与acquire (int arg)相同，但是该方法响应中断，当前线程未获取到同步状态而进人同步队列中，如果当前线程被中断，则该方法会抛出InterruptedException并返回。
boolean tryAcquireNanos (int arg, long nanos)|在acquireInterruptibly (int arg)的基础上增加了超时限制，如果当前线程在超时时间内没有获取到同步状态，那么将会返回false，如果获取到了返回true。
void acquireShared(int arg)|共享式的获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待，与独占式获取的主要区别是在同一时刻可以有多个线程获取到同步状态。
void acquireSharedInterruptibly(int arg)|与acquireShared(int arg)相同，该方法响应中断。
boolean tryAcquireSharedNanos(int arg, long nanos)|与acquireSharedInterruptibly(int arg)基础上增加了超时等待限制。
boolean release(int arg)|独占式释放同步状态，该方法会在释放同步状态后，将同步队列中第一个节点包含的线程唤醒。
boolean releaseShared(int arg)|共享式的释放同步状态。
Collection<Thread> getQueuedThreads()|获取等待在同步队列上的线程集合。

* 实现：

#### 独占式获取同步状态的关键点

1. AQS依赖内部的同步队列（一个FIFO的双向队列）来完成同步状态的管理。同步队列中节点（Node）用来保存获取同步状态失败的线程引用、等待状态以及前驱节点和后继节点。
2. 通过调用AQS的acquire(int arg)方法可以获取同步状态。
3. 首先调用SpecificSynchronizer实现的tryAcquire(int arg)，该方法保证线程安全的获取同步状态。
4. 如果获取同步状态失败，通过调用addWaiter(Node node)方法将该节点加入到同步队列的尾部。主要是有一段快速尝试在尾部添加节点的操作。
5. 在enq(final Node node)方法中，主要是通过CAS将节点正确地设置成尾节点。
6. 节点加入同步队列后，调用acquireQueued(final Node node, int arg)方法进入一个自旋的过程。每个节点（或者说每个线程）都在自省的观察；
当条件满足，获取到了同步状态，就会从自旋过程中退出；否则依旧停留在自旋过程中。
7. acquireQueued(final Node node, int arg)的主要操作有两个：
一是判断是否当前节点的前驱节点是head节点，并且成功获取到了同步状态；则将当前节点设置为head节点，并将前驱节点设置为null。最后是返回并退出自旋，继续执行线程任务。
二是根据当前节点的前驱节点的waitStatus是否是SIGNAL来决定是否park当前节点中的线程。这里有一段将前驱节点的waitStatus设置为SIGNAL的操作，所以同步队列中的任意节点的前驱节点的waitStatus为SIGNAL，则任意节点中的线程就会park。
8. release(int arg)的作用是调用tryRelease(int arg)将state设置为0，如果成功则唤醒（unpark）在队列自旋中park的，并且是head节点的后继节点中的线程。

* 参考测试代码：

> https://github.com/legend9207/concurrent_code/blob/master/src/com/roocon/thread/ta4/Main.java


### 3. 重入锁

* tryAcquire增加了再次获取同步状态的逻辑：判断当前线程是否为获取锁的线程；如果是，则将同步状态的值增加并返回true，表示获取同步状态成功。

* tryRelease会判断，在经过前面n-1次的tryRelease后；判断此次的同步状态是否为0，如果是，则将占有线程设为null，并返回true，表示释放成功。

### 4. 读写锁   