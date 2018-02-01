---
layout: post
title: Hystrix工作机制
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "hystrix"
---
## Hystrix工作流

使用hystrix对第三方服务进行请求时，流程如下图所示：
![hystrix流程图](/images/hystrix/hystrix_flow.png)

接下来将每个环节进行逐一介绍。

### 构造HystrixCommand 或 HystrixObservableCommand对象
首先，要做的就是构造HystrixCommand或HystrixObservableCommand对象来表示将要对依赖服务进行的请求。
   
若只期望所依赖服务每次返回单一的响应，可以按如下方式构造一个 HystrixCommand 即可：
```
   HystrixCommand command = new HystrixCommand(arg1, arg2);
```   
   
若期望所依赖服务返回一个Observable（该Observable可以返回多条响应），可按如下方式构造一个 HystrixObservableCommand：
```  
   HystrixObservableCommand command = new HystrixObservableCommand(arg1, arg2);
```

### 执行Command
Hystrix 命令提供四种方式（HystrixCommand 支持所有四种方式，而 HystrixObservableCommand 仅支持后两种方式）来执行你包装的请求：
*   execute() —— 阻塞，当依赖服务返回单条响应时（或因为错误抛出异常），返回结果
*   queue() —— 返回 Future 对象，通过该对象异步得到单条返回结果
*   observe() —— 
*   toObservable() —

调用代码如下：
```
   K             value   = command.execute();
   Future<K>     fValue  = command.queue();
   Observable<K> ohValue = command.observe();         //hot observable
   Observable<K> ocValue = command.toObservable();    //cold observable
```

execute() 方法是同步调用，内部实现上会调用queue().get()方法。
而queue() 方法内部则会调用 toObservable().toBlocking().toFuture()。
也就是说，HystrixCommand 内部均通过一个 Observable 的实现来执行请求，即使这些HystrixCommand本来是用来返回单条的响应。

### 请求缓存
如果request caching特性被启用，并且请求时响应缓存命中，则会以Observable对象的形式立即返回响应。


### 熔断器

当执行一个Command时，Hystrix 会先检查熔断器状态，确定是否熔断。

若熔断器是熔断状态，Hystrix 将不会执行这个命令，而是直接进行失败回退步骤。

若不是熔断状态，Hystrix 将会继续执行，检查线程池中是否有可用的线程来执行Command。

### 线程池

若关联了当前command的线程池满了，Hystrix 将不会执行该command，而是直接进行失败回退步骤。

### HystrixObservableCommand.construct() or HystrixCommand.run()

### 健康度计算

Hystrix 会将请求成功、失败、被拒绝或超时信息报告给熔断器，熔断器维护一些用于统计数据用的计数器。

这些计数器产生的统计数据使得熔断器在特定的时刻，能短路某个依赖服务的后续请求，直到恢复期结束，
若恢复期结束根据统计数据熔断器判定线路仍然未恢复健康，熔断器会再次关闭线路

### 降级

### 成功返回

## 参考文献:
1. https://github.com/Netflix/Hystrix/wiki/How-it-Works