---
layout: post
title: Java 线程
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "java 线程"
---

### 基本概念

#### **进程**

进程有一个独立的执行环境。
进程通常有一个完整的、私人的基本运行时资源; 特别是, 每个进程都有其自己的内存空间。
进程往往被视为等同于程序或应用程序。
然而, 用户看到一个单独的应用程序实际上可能是一组相互协作的进程。
大多数操作系统都支持进程间通信( Inter Process Communication，简称 IPC), 如管道和套接字。
IPC 不仅用于同个系统的进程之间的通信，也可以用在不同系统的进程。
大多数 Java 虚拟机都是作为一个进程运行, 同时Java 应用程序也可以使用 ProcessBuilder 对象创建额外的进程, 比如调用shell脚本就可以用它。


#### **线程**

线程有时被称为轻量级进程。进程和线程都提供一个执行环境, 但创建线程比创建进程耗费资源的资源要少。
线程中存在于进程中, 而且仅存在于一个进程中。
线程共享进程的资源, 包括内存和打开的文件, 这样可以使得工作更高效，但也存在了一个潜在的问题 —— 通信。
多线程是 Java 平台的一个重要特点。每个应用程序都至少有一个线程或者多个。
如果算上“系统”的线程（负责内存管理和信号处理）那就更多。但从程序员的角度来看, 程序开始启动时只有一个线程，这个线程称为主线程。
同时, 该线程有能力创建额外的线程。


#### **中断**

中断是一种协作机制, 就是一个线程向另一个线程发送请求, 请求停止那个线程正在做的事情。
另外, 当一个线程中断另一个线程时，被中断的线程不一定要立即停止正在做的事情。
相反，中断是礼貌地请求另一个线程在它愿意并且方便的时候停止它正在做的事情。

每个线程都有一个与之相关联的 boolean 属性，用于表示线程的中断状态(interrupted status), 中断状态初始时为 false。
当另一个线程通过调用某个线程的interrupt()方法来中断那个线程时，会出现以下两种情况之一:

*   如果那个线程在执行一个低级可中断阻塞方法，如Thread.sleep()、 Thread.join() 或 Object.wait()，那么它将取消阻塞并抛出 InterruptedException;
*   否则, interrupt()只是设置线程的中断状态, 被中断线程中可以轮询中断状态，看它是否被请求停止正在做的事情。


同时, 中断状态可以通过 isInterrupted() 来读取，并且可以通过一个名为 interrupted() 的操作读取和清除。


#### **睡眠**

Thread.sleep() 方法使得当前线程暂停执行一段时间。该方法的好处是可以让出cpu资源, 给其它线程或者其他应用。
另外, 由于sleep()方法是Thread 类的方法，因此它不能改变对象的锁状态。
当在一个Synchronized方法中调用sleep()方法时，虽然线程休眠了，但是对象锁并没有被释放，其他线程仍然无法持这个对象锁。

另外, 线程在sleep的过程中有可能被其他对象调用该线程的interrupt(), 产生InterruptedException异常。
如果你的程序不捕获这个异常，线程就会异常终止，进入TERMINATED状态;
如果你的程序捕获了这个异常，那么程序就会继续执行catch语句块(可能还有finally语句块)以及以后的代码。

sleep()方法让一个线程进入睡眠状态，等待一定的时间之后，自动醒来进入到可运行状态，但不会马上进入运行状态，因为线程调度机制恢复线程的运行也需要时间。
因此在任何情况下, 都不能认为sleep()方法让线程休眠精确的时间。

注意sleep()方法是一个静态方法，也就是说它只对当前对象有效, 使用时须注意。


#### **等待和唤醒**

object.wait()方法, 也是使得当前线程暂停执行一段时间, 等待被唤醒或者被中断或者时间到达, 属于线程之间通信的范畴。
它的好处是, 如果没有该方法, 那么要实现线程间的协作目的就需要线程不停地检查某个变量是否满足某条件了。

另外, 该方法必须获得该对象的管程锁, 同时将本线程放置在该对象的wait set中, 然后放弃所有本线程对该对象的同步声明。

这里我们假设该线程为thread、对象为object, 线程thread停止工作直到以下任何事情发生为止:

*   其他某线程调用object.notify()方法, 同时thread碰巧被唤醒;
*   其他某线程调用object.notifyAll() 方法;
*   其他某线程调用该线程的thread.interrupt()方法;
*   指定大约等待的时间到达;
     
然后, 线程thread从对象object的wait set中移除, 重新被调度并和其他在该对象object上同步的线程进行竞争。
另外, 线程唤醒不需要类似于object.notify()方法, 中断和时间到达都会唤醒线程, 这类唤醒被称作为虚假唤醒(spurious wakeup)。
虽然, 这种情况在实际编程中出现并不多, 但是有时候应用程序需要对相应的条件进行判断来防止虚假唤醒, 比如:

```
    synchronized (obj) {
         while (condition does not hold) {
             obj.wait(timeout);
             //Perform action appropriate to condition
         }    
    }
```

***

**注意**: wait()方法调用的时候需要先获得该object的锁，调用wait()后，会把当前的锁释放掉同时阻塞住；
当其它线程调用该object的notify/notifyAll方法之后，该线程就有可能得到cpu资源，同时重新获得锁。

***

### 线程创建

1.继承Thread类创建线程, 代码如下:

```
    private class WorkerThread extends Thread {
		public WorkerThread(){
			this.setName(batchQueueName + THREAD_INDEX++);
		}
		
		public long timestamp = 0;
		public final ArrayList<E> buffer = new ArrayList<E>(bufferSize);
		
		@Override public void run(){
			dispatch(WorkerThread.this);
		}
	}
```


2.实现Runnable接口, 代码如下:

```
    private final class Worker extends AbstractQueuedSynchronizer implements Runnable {
        /**
         * This class will never be serialized, but we provide a
         * serialVersionUID to suppress a javac warning.
         */
        private static final long serialVersionUID = 6138294804551838833L;

        /** Thread this worker is running in.  Null if factory fails. */
        final Thread thread;
        /** Initial task to run.  Possibly null. */
        Runnable firstTask;
        /** Per-thread task counter */
        volatile long completedTasks;

        /**
         * Creates with given first task and thread from ThreadFactory.
         * @param firstTask the first task (null if none)
         */
        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            this.thread = getThreadFactory().newThread(this);
        }

        /** Delegates main run loop to outer runWorker  */
        public void run() {
            runWorker(this);
        }
    }    
```

3.实现Callable接口, 再适配成实现Runnable接口的类, 如FutureTask类;

```
    public class FutureTask<V> implements RunnableFuture<V> {
        private Callable<V> callable;
        
        public FutureTask(Callable<V> callable) {
            if (callable == null)
                throw new NullPointerException();
            this.callable = callable;
            this.state = NEW;       // ensure visibility of callable
        }
    
        public FutureTask(Runnable runnable, V result) {
            this.callable = Executors.callable(runnable, result);
            this.state = NEW;       // ensure visibility of callable
        }
        
        public void run() {
            if (state != NEW ||
                !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                             null, Thread.currentThread()))
                return;
            try {
                Callable<V> c = callable;
                if (c != null && state == NEW) {
                    V result;
                    boolean ran;
                    try {
                        result = c.call();
                        ran = true;
                    } catch (Throwable ex) {
                        result = null;
                        ran = false;
                        setException(ex);
                    }
                    if (ran)
                        set(result);
                }
            } finally {
                // runner must be non-null until state is settled to
                // prevent concurrent calls to run()
                runner = null;
                // state must be re-read after nulling runner to prevent
                // leaked interrupts
                int s = state;
                if (s >= INTERRUPTING)
                    handlePossibleCancellationInterrupt(s);
            }
        }
        ... 
    }    
```

### 常用方法

#### join()方法
```
    /*
     * 等待至多millis毫秒, 直至线程结束; 当millis为0时, 表示一直等待直至线程结束;
     */    
    public final synchronized void join(long millis) throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }

    /**
     * 等待某线程, 直至该线程结束;
     */
    public final void join() throws InterruptedException {
        join(0);
    }
```

### 生命周期

线程有6个状态: NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING 和 TERMINATED。每个状态的说明如下:

```
    public enum State {
        /**
         * 新创建但还尚未start线程所处于的状态;
         */
        NEW,

        /**
         * 在jvm中正在执行或者等待调度器调度的线程所对应的状态;
         */
        RUNNABLE,

        /**
         * 等待获得管程锁的线程所对应的状态(用于互斥);
         * 处于阻塞状态的线程表示该线程想要进入synchronized的代码块/方法,
         * 或者在执行wait()方法后需要重新进入synchronized的代码块/方法
         */
        BLOCKED,

        /**
         * 等待线程所对应的状态(用于同步);
         * 若一个线程处于等待状态则表明该线程等待另一个线程完成某个特定的操作,
         * 如object.wait()方法等待其他线程调用类似于object.notify()等操作。
         * 当执行以下三种方法后, 标志着该线程进入等待状态:
         *   object.wait()方法 //等待其他线程执行notify()方法
         *   thread.join()方法 //等待其他线程结束
         *   LockSupport.park()方法 
         */
        WAITING,

        /**
         * 指定等待时间的线程所对应的状态(用于同步);
         * 当执行以下几种方法后, 标志着该线程进入timed-waiting状态;
         *   thread.sleep()方法
         *   object.wait(long)方法
         *   thread.join(long)方法
         *   LockSupport.parkNanos()方法
         *   LockSupport#parkUntil()方法
         * </ul>
         */
        TIMED_WAITING,

        /**
         * 线程完成执行并已经结束线程所对应的状态;
         */
        TERMINATED;
    }
```

6种状态的迁移关系, 如下图所示:

![threadState](/images/java/threadState.png)


### 参考文献:
1. http://fangjian0423.github.io/2016/03/22/java-threadpool-analysis/
2. http://www.ibm.com/developerworks/java/library/j-jtp05236/j-jtp05236-pdf.pdf
3. http://stackoverflow.com/questions/3265640/why-threadgroup-is-being-criticised
4. https://docs.oracle.com/javase/tutorial/essential/concurrency/runthread.html
5. http://ifeve.com/customizing-concurrency-classes-4/
6. https://wangchangchung.github.io/2016/12/05/Java%E5%B8%B8%E7%94%A8%E7%B1%BB%E6%BA%90%E7%A0%81%E2%80%94%E2%80%94Thread%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/#comments
7. https://waylau.gitbooks.io/essential-java/docs/concurrency-Processes%20and%20Threads.html
8. https://www.kancloud.cn/seaboat/java-concurrent/117878
9. http://www.java67.com/2012/08/what-are-difference-between-wait-and.html
10. http://www.uml-diagrams.org/examples/java-6-thread-state-machine-diagram-example.html