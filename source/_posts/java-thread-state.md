---
title: Java 中线程的状态
date: 2021-08-29 11:11:53
tags:
  - 线程
categories:
  - java
---

### 看源码

查看 Thread.State 枚举类我们可以看到Java中线程有以下几种状态：

1. **初始(NEW)** ：新创建了一个线程对象，但还没有调用start()方法。

2. **运行(RUNNABLE)** ：Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。

3. **阻塞(BLOCKED)** ：表示线程因 synchronized 阻塞于锁。

4. **等待(WAITING)** ：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。

5. **限时等待(TIMED_WAITING)** ：该状态类似WAITING，但它可以在指定的时间后自行返回。

6. **终止(TERMINATED)** ：表示该线程已经执行完毕。

### 上图

![](/images/java-thread-state/java-thread-state.jpg)

### 问题

##### Thread.sleep 与 Object.wait 有什么区别？

1. sleep 不会释放锁，wait 会释放锁
2. sleep 可在任何地方执行，wait 只能在 synchronized 块中执行

##### 如何令线程 a 等待线程 b 执行完毕后再执行？

在线程 a 中执行 b.join()，**不释放锁**

##### Thread.yield 有什么作用？

让出 CPU 时间片段，使线程进入 ready 状态，**不释放锁**

------
![福利](/images/骚图/三国杀/王异4.jpg)
