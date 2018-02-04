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

在 HystrixCommand 和 HystrixObservableCommand 的实现中，你可以定义一个缓存的 Key，
这个 Key 用于在同一个请求上下文（全局或者用户级）中标识缓存的请求结果，当然，该缓存是线程安全的。

下例展示了在一个完整 HTTP 请求周期内，两个线程执行命令的流程：
![hystrix缓存](/images/hystrix/hystrix-cache.png)

请求缓存有如下好处：
*   不同请求路径上针对同一个依赖服务进行的重复请求（有同一个缓存 Key），不会真实请求多次
这个特性在企业级系统中非常有用，在这些系统中，开发者往往开发的只是系统功能的一部分。（注：这样，开发者彼此隔离，不太可能使用同样的方法或者策略去请求同一个依赖服务提供的资源）

    例如，请求一个用户的 Account 的逻辑如下所示，这个逻辑往往在系统不同地方被用到：
    ```
        Account account = new UserGetAccount(accountId).execute();
    
        //or
    
        Observable<Account> accountObservable = new UserGetAccount(accountId).observe();
    ```
    Hystrix 的 RequestCache 只会在内部执行 run() 方法一次，
    上面两个线程在执行 HystrixCommand 命令时，会得到相同的结果，即使这两个命令是两个不同的实例。

*   数据获取具有一致性
    因为缓存的存在，除了第一次请求需要真正访问依赖服务以外，后续请求全部从缓存中获取，可以保证在同一个用户请求内，不会出现依赖服务返回不同的回应的情况。

*   避免不必要的线程执行
    在 construct() 或 run() 方法执行之前，会先从请求缓存中获取数据，因此，Hystrix 能利用这个特性避免不必要的线程执行，减小系统开销。

    若 Hystrix 没有实现请求缓存，那么 HystrixCommand 和 HystrixObservableCommand 的实现者需要自己在 construct() 或 run() 方法中实现缓存，这种方式无法避免不必要的线程执行开销。

### 熔断器

当执行一个Command时，Hystrix 会先检查熔断器状态，确定是否熔断。

若熔断器是熔断状态，Hystrix 将不会执行这个命令，而是直接进行失败回退步骤。

若不是熔断状态，Hystrix 将会继续执行，检查线程池中是否有可用的线程来执行Command。

下图展示了HystrixCommand或HystrixObservableCommand如何与HystrixCircuitBreaker进行交互： 
![hystrix熔断器](/images/hystrix/circuit-breaker-logic-flow.png)

熔断器的打开和关闭逻辑详细如下：
*   假设线路内的容量（请求QPS）达到一定阈值（通过 HystrixCommandProperties.circuitBreakerRequestVolumeThreshold() 配置）；
*   假设线路内的错误率达到一定阈值百分比（通过 HystrixCommandProperties.circuitBreakerErrorThresholdPercentage() 配置）；
*   然后，熔断器将从关闭转换成打开状态；
*   若此时是打开状态，熔断器将短路后续所有经过该熔断器的请求，这些请求直接走fallback()逻辑；
*   经过一定时间（即休眠窗口，通过 HystrixCommandProperties.circuitBreakerSleepWindowInMilliseconds() 配置），
后续第一个请求将会被允许通过熔断器（此时熔断器处于半开状态），若该请求失败，熔断器将又进入打开状态，且在休眠窗口内保持此状态；
若该请求成功，熔断器将进入关闭状态，回到逻辑1循环往复。

### 线程池

若关联了当前command的线程池满了，Hystrix 将不会执行该command，而是直接进行失败回退步骤。

Hystrix 通过使用舱壁模式来隔离依赖服务。
（注：将船的底部划分成一个个的舱室，这样一个舱室进水不会导致整艘船沉没。将系统所有依赖服务隔离起来，一个依赖延迟升高或者失败，不会导致整个系统失败）
![hystrix舱壁](/images/hystrix/hystrix-bulkhead.png)

通过将对依赖服务的访问执行放到单独的线程，将其与调用线程（例如 Tomcat 线程池中的线程）隔离开来，
调用线程就能空出来去做其他的工作而不至于被依赖服务的访问阻塞过长时间。

Hystrix 使用独立的，每个依赖服务对应一个线程池的方式，来隔离这些依赖服务，这样，某个依赖服务的高延迟只会拖慢这个依赖服务对应的线程池。
![hystrix线程池](/images/hystrix/hystrix-threadpool.png)

当然，也可以不使用线程池来使你的系统免受依赖服务失效的影响，
这需要你小心的设置超时时间和重试配置，并保证这些配置能正确正常的运作，以及能快速返回错误。

Netflix 在设计 Hystrix 时，使用线程/线程池来实现隔离，原因如下：
*   多数系统同时运行了（有时甚至多达数百个）不同的后端服务，这些服务由不同开发组开发；
*   第三方依赖经常会发生变动；
*   第三方依赖对于使用者来说，相当于黑盒，其实现细节，网络访问方式，默认配置等等均对使用者透明；
*   第三方依赖可能也会有失效或者高延迟，而不仅仅是在网络访问时；
![hystrix超时](/images/hystrix/hystrix-timeout.png)

当然，线程池也有弊端，主要是会增加系统 CPU 的负载，每个命令的执行，都包含了 CPU 任务的排队，调度，上下文切换。
Netflix 在设计 Hystrix 时，认为相对于其带来的好处，其带来的负载的一点点升高对系统的影响是微乎其微的。


### HystrixObservableCommand.construct() or HystrixCommand.run()

### 健康度计算

Hystrix 会将请求成功、失败、被拒绝或超时信息报告给熔断器，熔断器维护一些用于统计数据用的计数器。

这些计数器产生的统计数据使得熔断器在特定的时刻，能短路某个依赖服务的后续请求，直到恢复期结束，
若恢复期结束根据统计数据熔断器判定线路仍然未恢复健康，熔断器会再次关闭线路。

### fallback机制

当Hystrix command执行失败时，Hystrix 将会执行失败回退逻辑，失败原因可能是：
*   当construct()方法或run()方法抛出异常；
*   熔断器打开；
*   comand执行超时；

fallback逻辑包含了通用的响应信息，这些响应从本地内存中或其他固定逻辑中得到，而不应有任何的网络依赖。
如果一定要在fallback逻辑中包含网络请求，必须将该网络请求包装在另一个 HystrixCommand 或 HystrixObservableCommand中。

若实现了fallback()方法，Hystrix 会将这个响应返回给command的调用者。
*   若 Hystrix 内部调用HystrixCommand.getFallback()时，会产生一个 Observable 对象，并包装用户实现的 getFallback()方法返回的响应；
*   若 Hystrix 内部调用HystrixObservableCommand.resumeWithFallback()时，会将用户实现的resumeWithFallback()返回的Observable对象直接返回。

若没有实现fallback()方法，或者fallback()方法抛出异常，Hystrix 内部还是会生成一个 Observable 对象，但它不会产生任何响应，
并通过onError通知立即中止请求。Hystrix 默认会通过onError通知调用者发生了何种异常。
你需要尽量避免fallback()方法执行失败，保持该方法尽可能的简单不易出错。

若fallback()方法执行失败，或者用户未提供fallback()方法，Hystrix 会根据调用执行命令的方法的不同而产生不同的行为：
*   execute() —— 抛出异常
*   queue() —— 成功返回 Future 对象，但其 get() 方法被调用时，会抛出异常
*   observe() —— 
*   toObservable() —— 


## 参考文献:
1. https://github.com/Netflix/Hystrix/wiki/How-it-Works