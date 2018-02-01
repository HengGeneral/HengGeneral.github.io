---
layout: post
title: Hystrix应用
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "hystrix"
---
## 基本介绍
在复杂的分布式系统中，依赖的系统出现故障通常是无法避免的。
Hystrix，作为一个容忍延迟和错误的类库，通过隔离第三方服务的访问、防止故障级联传播和提供降级机制，来提供系统的健壮性。
它被设计用来解决：

*   对第三方服务调用的超时和错误进行保护
*   对错误级联进行防护，从而避免雪崩
*   快速失败和迅速恢复
*   降级
*   提供近乎实时的监控、报警和动态操控

## 问题描述
在复杂的分布式系统中，应用程序通常存在非常多的依赖服务，而这其中的某些服务在运行时不可避免的会出现故障。
然而，如果应用程序没有和这些故障进行隔离，那么它也有存在被拖垮的风险。

举个例子, 某个应用程序依赖30个不同的服务，若每个服务的服务成功率为99.99%，这个服务的成功率就可以这样计算:
$$99.99^{30} = 99.7%$$ 
那么，1亿个请求中，就有30万（0.3%）个请求失败。然而，这个仅仅是一个计算值，实际的情况可能会更糟。

当每个服务都是监控的时候，请求的调用如下图所示：
![请求调用](/images/hystrix/request_flow.png)

当某个依赖服务调用延迟服务较大，该服务将会成为系统的瓶颈，如下图所示：
![调用瓶颈](/images/hystrix/request_block.png)

当该依赖服务调用造成大量的请求服务处理缓慢，瞬间就会造成系统的其它资源受影响。
在应用程序中，每一个通过网络或者第三方调用发送出去的请求都有可能会失败。
然而，比失败更糟糕的是，应用程序会将后续请求、数据等放在队列中，造成其它系统资源的占用，造成服务延迟加剧，从而引起了错误传播，引起雪崩。
如下图所示：
![失败加剧](/images/hystrix/request_block_deteriorate.png)

Hystrix解决方案如下：
*   每个服务都维护一个小的线程池，当线程池满时直接拒绝而不是请求排队
*   对请求的成功、失败、超时或者线程拒绝进行统计
*   当请求失败、被拒绝、超时或者短路时，提供降级服务

当你使用Hystrix对每个依赖服务进行封装，系统的整体架构和上面的那张图很相似。
但是，每一个依赖之间都是相互隔离的，当某个依赖发生超时，系统占用的资源也得到限制，同时可以使用降级机制，如下图所示：
![失败降级](/images/hystrix/request_fallback.png)


## Hystrix实战
### 引入JAR
```
    <dependency>
          <groupId>com.netflix.hystrix</groupId>
          <artifactId>hystrix-core</artifactId>
          <version>${hystrix.version}</version>
    </dependency>
```
  
### 应用

```
public abstract class DemoHystrixCommand<R> extends HystrixCommand<R> {

	protected DemoHystrixCommand(String groupKey, String threadPoolKey, int threadPoolSize) {
		this(groupKey, threadPoolKey, threadPoolSize, 10, 20, 1000 * 10);
	}

	protected DemoHystrixCommand(String groupKey, String threadPoolKey, int threadPoolSize,
								 int errorThresholdPercentage, int requestVolumeThreshold,
								 int sleepWindowInMilliseconds) {
		this(groupKey, threadPoolKey, threadPoolSize, errorThresholdPercentage,
				requestVolumeThreshold, sleepWindowInMilliseconds,true);
	}

	protected DemoHystrixCommand(String groupKey, String threadPoolKey, int threadPoolSize,
                                 int errorThresholdPercentage, int requestVolumeThreshold, int sleepWindowInMilliseconds, boolean interrupted) {
		super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey(groupKey))
				.andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey(threadPoolKey))
				.andCommandPropertiesDefaults(
						HystrixCommandProperties.Setter()
								.withExecutionIsolationStrategy(HystrixCommandProperties.ExecutionIsolationStrategy.THREAD)
								// 是否使用熔断器
								.withCircuitBreakerEnabled(true)
								// 错误率超过该值，触发熔断器由CLOSED状态变为OPEN状态
								.withCircuitBreakerErrorThresholdPercentage(errorThresholdPercentage)
								// 熔断闭合时失败率判断之前，一个采样周期内必须进行至少N个请求才能进行采样统计
								.withCircuitBreakerRequestVolumeThreshold(requestVolumeThreshold)
								// 熔断器处于OPEN状态之后，需要等待重试的时间
								.withCircuitBreakerSleepWindowInMilliseconds(sleepWindowInMilliseconds)
								// 该参数表示处于正在执行的thread超时后是否需要被interrupt
								.withExecutionIsolationThreadInterruptOnTimeout(interrupted)
								// 该参数表示不满足命令执行超时的条件，是否强制thread被interrupt
								.withExecutionIsolationThreadInterruptOnFutureCancel(interrupted))
				// 线程池配置
				.andThreadPoolPropertiesDefaults(
						HystrixThreadPoolProperties.Setter()
								// 线程池的最小线程数
								.withCoreSize(threadPoolSize >> 2 > 0 ? threadPoolSize >> 2 : 1)
								// 线程池的最大线程数
								.withMaximumSize(threadPoolSize)
								.withAllowMaximumSizeToDivergeFromCoreSize(true)
								.withKeepAliveTimeMinutes(1)
								// 线程池任务队列的最大容量
								.withMaxQueueSize(0)
								.withQueueSizeRejectionThreshold(1)));
	}

	/**
	 * 获取第三方数据方法, 必须实现
	 */
	@Override
	protected abstract R run() throws Exception;

	/**
	 * 降级，默认返回值, 必须实现
	 */
	@Override
	protected abstract R getFallback();
}
```

```
/**
 * @author liheng
 * @date 2017/12/28
 */
public class HttpHystrixCommand extends DemoHystrixCommand<String> {
	private static final Logger LOGGER = LoggerFactory.getLogger(HttpHystrixCommand.class);

	private int timeout;
	private HttpRequest.HttpMethod httpMethod;
	private String requestUrl;
	private Map<String, String> paramMap;

	public HttpHystrixCommand(String groupKey, int threadPoolSize, HttpRequest.HttpMethod httpMethod,
							  String requestUrl, Map<String, String> paramMap, int timeout) {
		this(groupKey, groupKey, threadPoolSize, httpMethod, requestUrl, paramMap, timeout);
	}

	public HttpHystrixCommand(String groupKey, String threadPoolKey, int threadPoolSize,
                              HttpRequest.HttpMethod httpMethod, String requestUrl,
							  Map<String, String> paramMap, int timeout) {
		super(groupKey, threadPoolKey, threadPoolSize);
		this.httpMethod = httpMethod;
		this.requestUrl = requestUrl;
		this.paramMap = paramMap;
		this.timeout = timeout;
	}

	@Override
	protected String run() throws Exception {
		long start = System.currentTimeMillis();
		HttpRequest request = new HttpRequest();
		request.setMethod(httpMethod);
		request.setParams(paramMap);
		QxHttpClient client = QxHttpClientFactory.getQxHttpClient();
		request.setUrl(requestUrl);
		client.setTimeout(timeout, timeout);
		HttpResponse response = request.send(client);
		LOGGER.info("httpHystrixCommand request info, requestUrl={} params={} response={} cost={}ms",
				new Object[]{requestUrl, paramMap, JSON.toJSONString(response),
						(System.currentTimeMillis() - start)});
		if (response != null && response.getHttpCode() == HttpStatus.SC_OK) {
			return response.getContent();
		}

		return null;
	}

	/**
	 * HTTP请求优雅降级，适用于读请求
	 * @return
	 */
	@Override
	protected String getFallback() {
		Throwable e = this.getExecutionException();
		if (null != e) {
			LOGGER.error("httpHystrixCommand request error, CircuitBreaker open={} ExecutionEvents={} requestUrl={} params={} exception={}",
					this.isCircuitBreakerOpen(), this.getExecutionEvents(), requestUrl, paramMap, e);
		}
		return null;
	}
}
```

```
String responseContent = new HttpHystrixCommand(this.getClass().getSimpleName(), 20,
                HttpRequest.HttpMethod.get, CONFIG_URL, params, reqTimeOutMS).execute();
```

## Hystrix原理

## Hystrix使用方法

## 监控报警




## 参考文献:
1. https://github.com/Netflix/Hystrix/wiki/How-it-Works