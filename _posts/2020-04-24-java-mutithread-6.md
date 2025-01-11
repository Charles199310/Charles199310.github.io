---
layout: post
title: Java多线程六
date: 2020-04-24 22:21 +0800
last_modified_at: 2025-1-1 21:14 +0800
tags: [Java多线程, CAS]
toc: true
---
# Java多线程六

## CAS
CAS（Compare And Swap）是一种无锁原子操作。具体是对比内存中的值与当前线程中我们预期的值，如果一致，则进行下一步赋值，即swap；如果不一致，则采取其他策略。  
CAS主要使用Unsafe这个类实现。在Java 9以后VarHandler也可以实现类似功能。这两个类都通过native的方法实现了CAS的方法。  
CAS在Java中有很多应用，下面我们以AtomicBoolean为例，看看他是如何使用的。
## AtomicBoolean
``` Java
public class AtomicInteger extends Number implements java.io.Serializable {
    /*
     * This class intended to be implemented using VarHandles, but there
     * are unresolved cyclic startup dependencies.
     */
    private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();

    //通过Unsafe拿到了这个类中的value并拿到偏移量
    private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");

    private volatile int value;

  /**
   * Atomically sets the value to {@code newValue}
   * if the current value {@code == expectedValue},
   * with memory effects as specified by {@link VarHandle#compareAndSet}.
   *
   * @param expectedValue the expected value
   * @param newValue the new value
   * @return {@code true} if successful. False return indicates that
   * the actual value was not equal to the expected value.
   */
  public final boolean compareAndSet(int expectedValue, int newValue) {
    //对比预期值如果一致就将心智赋值进入，原子操作
      return U.compareAndSetInt(this, VALUE, expectedValue, newValue);
  }
｝
```
