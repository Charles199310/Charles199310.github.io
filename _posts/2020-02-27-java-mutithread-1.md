---
layout: post
title: Java多线程一
date: 2020-02-29 21:47 +0800
last_modified_at: 2025-1-1 21:00 +0800
tags: [Java多线程]
toc: true
---

# Java多线程一

## 什么是线程
线程是轻量级的进程，是计算机调度的最小单位。计算机将CPU运算资源分配给不同的线程，当CPU资源不够时，不同线程采用分时复用的方法轮流获取运算资源。
## 为什么要用多线程
* __最大化利用硬件性能__。在多核处理器中，如果采用单线程处理任务，那么实际工作的核心数只有一个。而采用多线程的话可以充分利用硬件核心数。例如，8核硬件中，并发多线程理论上可以达到单线程处理的8倍速度。
* __避免阻塞__。有时我们需要开启一个耗时任务，但又不希望耗时任务影响其他任务的执行，造成阻塞，例如下载。这时候可以将耗时任务放入子线程执行，这样可以确保其他线程不会因为这个耗时任务阻塞其他任务。
* __简化模型__。我们可以将不相干的任务拆分到不同的线程中，各个线程各司其职，避免出现一个混杂了大量不同任务的线程。
## 多线程的挑战
* __可见性__ 在Java虚拟机中，对象是保存在堆中的，所以对象中的成员变量也在堆中保存。而Java的与运算是在方法中，方法执行时，先将成员变量加载到操作数栈，在操作数栈执行完成后再同步到堆中。这样在操作数栈执行完成但没有同步到堆中时，这时候同一个变量就出现了两个值。如果此时恰好有另一个线程要读取这个值，就会有我们意料不到的情况。
* __有序性__。有序性包含两个方面，一是当我们希望两个线程交替执行任务。有一个典型的面试题，两个线程交替打印0-99。另一个是指令重排序带来的问题，假设有下面代码：  
  ```Java
  # OrderingTest
  public class OrderingTest {
    private boolean inited;
    private int testInteger;

    public void init(){
      testInteger = 1;
      inited = true;
    }

    public void test(){
      if (inited) {
        System.out.println(100 / testInteger);
      }
    }
  }
  ```
  在上面代码假设在同一线程中执行，一点问题都没有。但是，如果两个方法如果执行在不同线程，则有可能造成崩溃。原因在于，JVM虚拟机在执行代码时有重排序的可能。init方法在多线程中的代码可能为
  ```Java
    inited = true;
    if (inited) {
      System.out.println(100 / testInteger);
    }
    testInteger = 1;
  ```
  单线程中，jvm有一套as-if-serial机制，可以确保不会产生这样的问题。即后面的代码如果对前面的代码有依赖关系，那么这段代码顺序不会错乱。例如上面的例子。`System.out.println(100 / testInteger);`对`testInteger = 1;`有依赖，那么在单线程中，前者即使在重排序后，也一定是后执行的。
* __原子性__。有时我们希望若干个操作不被其他线程打断。比如，卖火车票，我们希望票池票数减一和用户付款要么都被执行，要么都没有执行，否则会出现，多个用户都买了同一张票，或者用户没付款的情况下票就没有了。原子性，就是将若干个操作包装成不可分割的一个操作。
* __死锁__。锁是我们经常用来解决上面问题的工具。但是，如果使用的不好，可能会造成死锁的问题。死锁就是在多线程中，多个线程因为彼此之间持有着对方需要的资源，造成所有线程都不能进行的情况。
* __以最好的性能运行多线程__。解决了上面的问题可以确保我们的代码不出现我们不希望的情况，但是不一定最好的效果。我们编程时还要追求以最好的效果达到我们想要的效果。