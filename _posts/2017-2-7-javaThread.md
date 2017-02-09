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
多线程执行是 Java 平台的一个重要特点。每个应用程序都至少有一个线程或者多个。
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

#### **阻塞**

#### **同步**

#### **异步**


### 线程创建

### 常用方法

### 生命周期





### 参考文献:
1. http://fangjian0423.github.io/2016/03/22/java-threadpool-analysis/
2. http://www.ibm.com/developerworks/java/library/j-jtp05236/j-jtp05236-pdf.pdf
3. http://stackoverflow.com/questions/3265640/why-threadgroup-is-being-criticised
4. https://docs.oracle.com/javase/tutorial/essential/concurrency/runthread.html
5. http://ifeve.com/customizing-concurrency-classes-4/
6. https://wangchangchung.github.io/2016/12/05/Java%E5%B8%B8%E7%94%A8%E7%B1%BB%E6%BA%90%E7%A0%81%E2%80%94%E2%80%94Thread%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/#comments
7. https://waylau.gitbooks.io/essential-java/docs/concurrency-Processes%20and%20Threads.html