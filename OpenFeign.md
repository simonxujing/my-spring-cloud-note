### OpenFeign

1. ##### 简介

   Feign是一个声明式WebService客户端，Feign使得编写WebService客户端更加简单。 使用方法是定义一个服务接口在上面添加注解。Feign也支持可拔插式编码器和解码器。SpringCloud对Feign进行了封装，使其支持了SpringMVC标准和HttpMessageConverters。Feign可以与Eureka与Ribbon组合使用以支持负载均衡

2. ##### 使用步骤

   1. 改pom.xml, 引入openfeign

      ```xml
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-openfeign</artifactId>
      </dependency>
      ```

   2. 在启动类上加上注解`@EnableFeignClients`

   3. 新增FeignService接口, 添加`@FeignClient`注解, value中填写Eureka中的ApplicationName

      ```java
      @Component
      @FeignClient(value = "CLOUD-PAYMENT-SERVICE")
      public interface IPaymentFeignService {
      	@GetMapping(value = "/payment/get/{id}")
      	CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
      }
      ```

   4. Controller中调用FeignService接口

      ```java
      @RestController
      @Slf4j
      public class OrderFeignController {
      
      	@Resource
      	private IPaymentFeignService paymentFeignService;
      
      	@GetMapping(value = "/consumer/payment/get/{id}")
      	public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
      		return paymentFeignService.getPaymentById(id);
      	}
      }
      ```

   5. 注意Feign返回的数据为xml格式, 修改pom.xml, 去除spring-cloud-starter-netflix-eureka-server中的jackson-dataformat-xml的引用

      ```xml
      <!-- eureka client -->
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
          <exclusions>
              <exclusion>
                  <artifactId>jackson-dataformat-xml</artifactId>
                  <groupId>com.fasterxml.jackson.dataformat</groupId>
              </exclusion>
          </exclusions>
      </dependency>
      ```

      

3. ##### OpenFeign超时控制 

   修改application.xml文件

   ```yml
   # 设置feign客户端超时时间（OpenFeign默认支持ribbon）
   ribbon:
     # 建立连接所用时间，适用于网络状况正常的情况下，两端连接所用的时间 5秒
     ReadTimeout: 5000
     # 指的是建立连接后从服务器读取到可用资源所用时间
     ConnectTimeout: 5000
   ```

   

4. ##### OpenFeign 日志

   **日志级别：**

   1. NONE：默认的，不显示任何日志

   2. BASIC：仅记录请求方法、URL、响应状态码及执行时间

   3. HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息

   4. FULL：除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据

      

   - 新增FeignConfig配置类

     ```java
     @Configuration
     public class FeignConfig {
     	@Bean
     	Logger.Level feignLoggerLevel(){
     		return Logger.Level.FULL;
     	}
     }
     ```

     

   - 修改application.yml文件

     ```yml
     logging:
       level:
         # feign 日志以什么级别监控哪个接口(此处接口为IPaymentFeignService)
         com.atguigu.springcloud.service.IPaymentFeignService: debug
     ```

     

