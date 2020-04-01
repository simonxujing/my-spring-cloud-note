### Ribbon

##### 1. 简介：Client Side Load Balancer

​	Netflix发布的云中间层服务开源项目，提供客户端负载均衡算法。我们可以在配置文件中Load Balancer后面的所有机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器，我们也很容易使用Ribbon实现自定义的负载均衡算法。

##### 2. 使用

​	pom.xml引入（Eureka中带有ribbon）

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

​	代码中使用，RestTemplate配置类添加@LoadBalanced注解。使用RestTemplate的postForObject、getForObject、postForEntity、getForObject。

```java
@Bean
@LoadBalanced
public RestTemplate getRestTemplate(){
    return new RestTemplate();
}
```

3. #####  Ribbon自带的负载均衡

   IRule根据特定算法中从服务列表中选取一个要访问的服务。

   - com.netflix.loadbalancer.RoundRobinRule 轮询
   - com.netflix.loadbalancer.RandomRule 随机
   - com.netflix.loadbalancer.RetryRule 按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内进行重试，获取可用的服务
   - WeightedReponseTimeRule 是RoundRobinRule的扩展，响应速度越快越容易被选中/。
   - BestAvailableRule 先过滤由于多次访问故障处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
   - AvailabilityFilteringRule 先过滤故障实例，在选择并发较小的实例
   - ZoneAvoidanceRule 默认规则，复合判断server所在区域的性能喝server的可用性选择服务器

4. ##### 负载均衡规则替换

   - 自定义配置类不能放在@ComponentScan所扫描的当前包及其子包下（注意主启动类注解@SpringBootApplication含有@ComponentScan）
   - 新建package：com.atguigu.myrule
   - 新建规则类：MySelfRule