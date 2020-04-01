# Gateway

## 一、简介

Gateway是在Spring生态系统之上构建的API网关服务，基于Spring 5，Spring Boot2 和Project Reactor等技术。Gateway旨在提供一个简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能。例如：熔断、限流、重试。SpringCloud Gateway使用的Webflux中的reactor-netty响应式编程组件，底层使用了Netty通讯框架。

## 二、用途

- 反向代理
- 鉴权
- 流量控制
- 熔断
- 日志监控

## 三、路由（Route）

路由是构建网关的基本模块，由Id、目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由。

## 四、断言（Predicate）

参考Java8的java.util.function.Predicate。开发人员可以匹配HTTP请求中的所有内容（例如请求头、请求参数），如果请求与断言相匹配则进行路由。

断言的配置predicates 中

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由的功能,利用微服务名进行路由
      routes:
        - id: payment_route # 路由的id,没有规定规则但要求唯一,建议配合服务名
          #匹配后提供服务的路由地址
#          uri: http://localhost:8001
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/get/** # 断言，路径相匹配的进行路由
          # 过滤
          #filters:
          #  - AddRequestHeader=X-Request-red, blue
        - id: payment_route2
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/lb/** #断言,路径相匹配的进行路由
            - After=2020-03-27T13:56:11.303+08:00[Asia/Shanghai]
              #- Before=2017-01-20T17:42:47.789-07:00[America/Denver]
            - Cookie=username,xuj
              #- Header=X-Request-Id, \d+ #请求头要有X-Request-Id属性，并且值为正数
              #- Host=**.atguigu.com
            #- Method=GET
            #- Query=username, \d+ # 要有参数名username并且值还要是正整数才能路由
```



## 五、过滤（Filter）

Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或之后对请求进行修改。

```java
@Component
@Slf4j
public class MyLogGatewayFilter implements GlobalFilter, Ordered {

  @Override
  public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
    log.info("**** come in MyLogGatewayFilter" + new Date());
    String uname = exchange.getRequest().getQueryParams().getFirst("uname");
    if (uname == null) {
      log.info("uname is null");
      exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
      return exchange.getResponse().setComplete();
    }
    return chain.filter(exchange);
  }

  @Override
  public int getOrder() {
    return 0;
  }
}
```



## 六、网关配置

### 1、application.yml配置

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_route # 路由的id,没有规定规则但要求唯一,建议配合服务名
          # 匹配后提供服务的路由地址
          uri: http://localhost:8001
          predicates:
            - Path=/payment/get/** # 断言，路径相匹配的进行路由
            #- After=2017-01-20T17:42:47.789-07:00[America/Denver]
            #- Before=2017-01-20T17:42:47.789-07:00[America/Denver]
            #- Cookie=username,zzyy
            #- Header=X-Request-Id, \d+ #请求头要有X-Request-Id属性，并且值为正数
            #- Host=**.atguigu.com
            #- Method=GET
            #- Query=username, \d+ # 要有参数名username并且值还要是正整数才能路由
          # 过滤
          #filters:
          #  - AddRequestHeader=X-Request-red, blue
        - id: payment_route2
          uri: http://localhost:8001
          predicates:
            - Path=/payment/lb/** #断言,路径相匹配的进行路由
```

### 2、代码中注入RouteLocator的Bean

```java
@Configuration
public class GatewayConfig {
	@Bean
	public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder){
		RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
		routes.route("path_route1", r -> r.path("/guonei").uri("http://news.baidu.com/guonei")).build();
		return routes.build();
	}
}
```

## 七、开启动态路由

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由的功能,利用微服务名进行路由
      routes:
        - id: payment_route # 路由的id,没有规定规则但要求唯一,建议配合服务名
          #匹配后提供服务的路由地址
#          uri: http://localhost:8001
          uri: lb://cloud-payment-service # eureka中注册的服务名
          predicates:
            - Path=/payment/get/** # 断言，路径相匹配的进行路由
            #- After=2017-01-20T17:42:47.789-07:00[America/Denver]
            #- Before=2017-01-20T17:42:47.789-07:00[America/Denver]
            #- Cookie=username,zzyy
            #- Header=X-Request-Id, \d+ #请求头要有X-Request-Id属性，并且值为正数
            #- Host=**.atguigu.com
            #- Method=GET
            #- Query=username, \d+ # 要有参数名username并且值还要是正整数才能路由
          # 过滤
          #filters:
          #  - AddRequestHeader=X-Request-red, blue
        - id: payment_route2
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/lb/** #断言,路径相匹配的进行路由
```

## 八、curl 请求命令

- curl http://localhost:9527/payment/lb --cookie "username=xuj"  带cookie的get请求
- curl http://localhost:9527/payment/lb -H "X-Request-Id:5"  请求头中包含X-Request-Id

