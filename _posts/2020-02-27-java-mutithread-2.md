
---
layout: post
title: Java多线程二
date: 2020-02-29 21:47 +0800
last_modified_at: 2025-1-1 21:00 +0800
tags: [Java多线程]
toc: true
---

# Java多线程二
## 线程、进程、协程
* __进程__ 进程是计算机资源调度和分配的单位。是程序的实体。计算机将自己的内存资源，CPU资源分配给不同的进程，这样程序才得以运行。
* __线程__ 线程是计算机运算的最小调度单位。线程依附于进程，一个进程至少有一个线程。线程没有自己的内存，而使用进程的内存。进程调度一个或多个线程进行运算。
* __协程__ 协程其实没有统一的定义。一般来说，协程是单线程内实现并发、非阻塞执行。kotlin中的协程实际上线程、只不过与传统的线程看起来不太一样。

## 线程的创建方式
线程的创建需要有系统的支持，所以在代码中，创建代码的方式本质上就几种，以及在其上做的一些封装。
* 使用Thread类创建线程。`Thread#start()`是native方法，调用这个方法后，进程会创建一个新线程，并且在新的线程里面执行`Thread#run()`方法。
  ```Java
  /**
   * If this thread was constructed using a separate
   * {@code Runnable} run object, then that
   * {@code Runnable} object's {@code run} method is called;
   * otherwise, this method does nothing and returns.
   * <p>
   * Subclasses of {@code Thread} should override this method.
   *
   * @see     #start()
   * @see     #stop()
   * @see     #Thread(ThreadGroup, Runnable, String)
   */
  @Override
  public void run() {
      if (target != null) {
          target.run();
      }
  }
  ```
所以,使用Thread类创建线程又有两种方式，1.创建一个Runnable对象作为target。2.创建一个Thread的子类，重写run()方法。
* 使用线程池创建和管理多线程。频繁的创建和切换线程，对于JVM会造成很大的负担。使用线程池即使可以降低这种负担。线程池本质上也是使用Thread，但是线程池提供了线程的复用方法。当一个线程结束后，根据之前线程池的设定，不会马上回收，而是等待一段时间，一段时间内如果用多线程任务的话，复用这个线程。线程池的基本构造方法如下
  ```Java
  /**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     * @param unit the time unit for the {@code keepAliveTime} argument
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     * @param threadFactory the factory to use when the executor
     *        creates a new thread
     * @param handler the handler to use when execution is blocked
     *        because the thread bounds and queue capacities are reached
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue}
     *         or {@code threadFactory} or {@code handler} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }  
  ```
* 以Android中的其他方式创建线程，如AsycTask，IntentService等，这些本质上都是对Thread的封装，在这里就不做详细介绍
