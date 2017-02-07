---
layout: post
title: Java 线程池
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "java 线程池"
---

引入线程池主要是为了解决两个问题: 

1. 当处理大量异步的任务时, 不用对每个任务都创建一个线程, 从而降低了thread创建的日常;
2. 对资源(线程和任务的执行)进行管理和限制。

一种常用的thread pool则是fixed thread pool。
这种类型的线程池通常拥有固定数目的线程, 如果其中还需要使用的线程终止了, 该终止的线程将会被新的线程取代。
当提交到线程池的任务超过该线程池设置的数目时, 多余的任务将会进入队列等待执行。

这类fixed thread pool的一个很重要的优点则在于应用程序能够平稳地退化。
为了更好地理解它, 我们以web应用程序为例。
假如web服务器对每个http请求创建一个新的线程, 当大量的用户访问该web站点时, 系统将会立即创建大量的线程。
当线程的数目超过系统的上限时, 系统将停止 response。
因此, 当有固定数目的线程被创建时, 应用程序将会按照系统能够支持的处理速度进行处理。

上述fixed thread pool的创建方法是通过调用 Executors.newFixedThreadPool(int) 来进行创建。
除此之外, Executors.newCachedThreadPool() 来创建可扩展的线程池,
Executors.newSingleThreadExecutor() 来创建一次只能执行一个task地线程池。

### ThreadPoolExecutor类

该类的4个构造函数, 分别如下:

```
    private static final RejectedExecutionHandler defaultHandler =
            new AbortPolicy();
    
    /*
     * 构造器1  
     * corePoolSize: 保存在线程池中线程的数目, 即使线程空闲, 除非allowCoreThreadTimeOut被设置
     * maximumPoolSize: 线程池中线程的最大数目
     * keepAliveTime: 当线程池中线程的数目超过core, 那些空闲的线程保留的最长时间
     * unit: keepAliveTime的单位, 取值参照TimeUnit类
     * workQueue: 在线程执行之前, 用来保存任务
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
    
    /*
     * 构造器2
     * threadFactory: 创建新线程的工厂
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
    }
       
    /*
     * 构造器3 
     * handler: 当线程的数目达到上限, 新到达任务的阻塞处理机制
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }
    
    /*
     * 构造器4
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

#### ThreadFactory类
这里使用ThreadFactory来为线程池定制化创建线程

#### RejectedExecutionHandler类
作为创建线程池时的一个参数, 用来处理当线程的数目达到上限时, 新到达任务的阻塞处理机制。jdk中已有的处理机制如下:

1.ThreadPoolExecutor.AbortPolicy: 丢弃任务并抛出RejectedExecutionException异常, 也是默认的处理机制, 其代码如下:

```
    /**
     * A handler for rejected tasks that throws a
     * {@code RejectedExecutionException}.
     */
    public static class AbortPolicy implements RejectedExecutionHandler {
        /**
         * Creates an {@code AbortPolicy}.
         */
        public AbortPolicy() { }

        /**
         * Always throws RejectedExecutionException.
         *
         * @param r the runnable task requested to be executed
         * @param e the executor attempting to execute this task
         * @throws RejectedExecutionException always
         */
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("Task " + r.toString() +
                                                 " rejected from " +
                                                 e.toString());
        }
    }
```

2.ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常, 其代码如下:
 
```
    /**
     * A handler for rejected tasks that silently discards the
     * rejected task.
     */
    public static class DiscardPolicy implements RejectedExecutionHandler {
        /**
         * Creates a {@code DiscardPolicy}.
         */
        public DiscardPolicy() { }

        /**
         * Does nothing, which has the effect of discarding task r.
         *
         * @param r the runnable task requested to be executed
         * @param e the executor attempting to execute this task
         */
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        }
    }
```

3.ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）, 其相应代码如下:

```
    /**
     * A handler for rejected tasks that discards the oldest unhandled
     * request and then retries {@code execute}, unless the executor
     * is shut down, in which case the task is discarded.
     */
    public static class DiscardOldestPolicy implements RejectedExecutionHandler {
        /**
         * Creates a {@code DiscardOldestPolicy} for the given executor.
         */
        public DiscardOldestPolicy() { }

        /**
         * Obtains and ignores the next task that the executor
         * would otherwise execute, if one is immediately available,
         * and then retries execution of task r, unless the executor
         * is shut down, in which case task r is instead discarded.
         *
         * @param r the runnable task requested to be executed
         * @param e the executor attempting to execute this task
         */
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                e.getQueue().poll();
                e.execute(r);
            }
        }
    }
```

4.ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务, 其代码如下:

```
    /**
     * A handler for rejected tasks that runs the rejected task
     * directly in the calling thread of the {@code execute} method,
     * unless the executor has been shut down, in which case the task
     * is discarded.
     */
    public static class CallerRunsPolicy implements RejectedExecutionHandler {
        /**
         * Creates a {@code CallerRunsPolicy}.
         */
        public CallerRunsPolicy() { }

        /**
         * Executes task r in the caller's thread, unless the executor
         * has been shut down, in which case the task is discarded.
         *
         * @param r the runnable task requested to be executed
         * @param e the executor attempting to execute this task
         */
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                r.run();
            }
        }
    }
```

当然, 可以对上述处理机制进行定制, 比如对AbortPolicy抛出的异常信息定制一下等等。

#### BlockingQueue类
作为创建线程池时的一个参数, 用来处理线程执行之前, 用来保存任务。
根据任务排队策略的不同, 常用的BlockingQueue类有SynchronousQueue, LinkedBlockingQueue和 ArrayBlockingQueue等。

**ArrayBlockingQueue**

基于数组的阻塞队列实现，在ArrayBlockingQueue内部，维护了一个定长数组，以便缓存队列中的数据，这是一个常用的阻塞队列。
ArrayBlockingQueue在生产者放入数据和消费者获取数据，都是共用同一个锁对象，由此也意味着两者无法真正并行运行，
这点不同于LinkedBlockingQueue。另外按照实现原理来分析，
ArrayBlockingQueue完全可以采用分离锁，从而实现生产者和消费者操作的完全并行运行。

ArrayBlockingQueue和LinkedBlockingQueue间还有一个明显的不同之处在于，
前者在插入或删除元素时不会产生或销毁任何额外的对象实例，而后者则会生成一个额外的Node对象。
这在长时间内需要高效并发地处理大批量数据的系统中，其对于GC的影响还是存在一定的区别。

**LinkedBlockingQueue**

基于链表的阻塞队列，同ArrayListBlockingQueue类似，
其内部也维持着一个数据缓冲队列（该队列由一个链表构成），
当生产者往队列中放入一个数据时，队列会从生产者手中获取数据，并缓存在队列内部，而生产者立即返回；
只有当队列缓冲区达到最大值缓存容量时（LinkedBlockingQueue可以通过构造函数指定该值），
才会阻塞生产者队列，直到消费者从队列中消费掉一份数据，生产者线程会被唤醒，反之对于消费者这端的处理也基于同样的原理。
而LinkedBlockingQueue之所以能够高效的处理并发数据，还因为其对于生产者端和消费者端分别采用了独立的锁来控制数据同步，
这也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。

但是需要注意的是，如果构造一个LinkedBlockingQueue对象，而没有指定其容量大小，
LinkedBlockingQueue会默认一个类似无限大小的容量（Integer.MAX_VALUE）。
这样的话，如果生产者的速度一旦大于消费者的速度，也许还没有等到队列满阻塞产生，系统内存就有可能已被消耗殆尽了。

**SynchronousQueue**

内部没有任何容量，任何的入队操作都需要等待其他线程的出队操作，反之亦然。
如果将SynchronousQueue用于生产者/消费者模式，那么相当于生产者和消费者手递手交易，
即生产者生产出一个货物，则必须等到消费者过来取货，方可完成交易。

使用SynchronousQueue的好处是, [性能更高][SynchronousQueueAdvantage]。

[SynchronousQueueAdvantage]: http://stackoverflow.com/questions/5102570/implementation-of-blockingqueue-what-are-the-differences-between-synchronousque?noredirect=1&lq=1

### 参考文献:
1. http://fangjian0423.github.io/2016/03/22/java-threadpool-analysis/
2. http://www.cnblogs.com/leesf456/p/5560362.html
3. http://www.cnblogs.com/zhanjindong/p/java-concurrent-package-ThreadPoolExecutor.html
4. https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html
5. http://blog.csdn.net/johnstrive/article/details/50667557
6. http://ifeve.com/customizing-concurrency-classes-4/
