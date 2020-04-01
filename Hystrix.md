# 一、Hystrix

## 1、简介

​	Hystrix 是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

​	断路器本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的，可处理的备选响应（Fallback），而不是长时间的等待或者抛出调用方法无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中蔓延，乃至雪崩。

## 2、Hystrix中的重要概念

### 1）服务降级

​	服务器忙，稍后再试，不让客户端等待并立刻返回一个友好提示，fallback。

​	什么情况下触发降级：

   - 程序运行异常
   - 超时
   - 服务熔断触发服务降级
   - 线程池/信号量打满也会导致服务降级

### 2）服务熔断

​	类似于保险丝，达到最多服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示

### 3）服务限流

​	秒杀高并发的操作，一秒钟N个，有序进行

## 3、使用

### 1）改pom

```xml
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

### 2）添加标签@EnableCircuitBreaker

### 3）service方法

```java
@Service
public class PaymentService {
	public String paymentInfoOK(Integer id){
		return "线程池" + Thread.currentThread().getName() + "paymentInfoOK，id：" + id + "\t";
	}

	@HystrixCommand(fallbackMethod = "paymentInfoTimeoutHandler", commandProperties = {
			@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
	})
	public String paymentInfoTimeout(Integer id){
		try{
			TimeUnit.SECONDS.sleep(5);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}

		return "线程池" + Thread.currentThread().getName() + "paymentInfoTimeout，id：" + id + "\t";
	}

	public String paymentInfoTimeoutHandler(Integer id){
		return "线程池: " + Thread.currentThread().getName() + "系统繁忙或运行报错,请稍后再试，id：" + id + "\t" + "handler";
	}
}
```

## 4、消费者（客户端）服务降级

### 1）application.yml修改，启动feign的Hystrix

```yml
feign:
	hystrix:
		enable: true
```

### 2)  主启动类上添加注解 `@EnableHystrix`

### 3)  添加注解实现服务降级

```java
@HystrixCommand(fallbackMethod = "paymentTimeoutFallbackMethod", commandProperties = {
			@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1500")
	})
```

## 5、服务降级，代码膨胀问题解决

​	方法一：每个方法配一个降级方法，这样会导致代码膨胀。`@DefaultProperties(defaultFallback = "paymentGlobalFallbackMethod")` 使用全局默认降级方法。监控需被降级的方法上使用 `@HystrixCommand`

​	方法二：HystrixService 上添加fallback注解 eg. `@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT", fallback = PaymentFallbackService.class)` 写该接口的实现类 PaymentFallbackService

```java
@Component
public class PaymentFallbackService implements PaymentHystrixService{
	@Override
	public String paymentInfoOK(Integer id) {
		return "----- PaymentFallbackService Fall Back";
	}

	@Override
	public String paymentInfoTimeout(Integer id) {
		return "----- PaymentFallbackService Fall Back timeout";
	}
}
```

## 6、服务熔断

### 1）熔断

​	熔断机制是应对雪崩效应的一种微服务链路保护机制，当扇出链路的某个微服务出错不可用或响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。当检测到该节点微服务响应正常后，恢复调用链路。

​	在SpringCloud框架里，熔断机制通过Hystrix实现，Hystrix会监控微服务间调用的状况，当失败的调用到到一定阈值，缺省：5秒内20次调用失败，就会启动熔断机制。熔断机制的注解是 `@HystrixCommand`

```java
/**
 * 服务熔断
 * @param id
 * @return
 */
@HystrixCommand(fallbackMethod = "paymentCircuitBreakerFallback", commandProperties = {
    @HystrixProperty(name = "circuitBreaker.enabled", value = "true"), //是否开启断路器
    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"), // 请求次数
    @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"), // 时间窗口期
    @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60") // 失败率达到多少后跳闸
})
public String paymentCircuitBreaker(@PathVariable("id") Integer id){
    if (id < 0){
        throw new RuntimeException("id 不能为负数");
    }
    String serialNum = IdUtil.simpleUUID();
    return Thread.currentThread().getName() + "\t" + "调用成功,流水号" + serialNum;
}

public String paymentCircuitBreakerFallback(@PathVariable("id") Integer id){
    return "id 不能为负数,请稍后再试, id: " + id;
}
```



