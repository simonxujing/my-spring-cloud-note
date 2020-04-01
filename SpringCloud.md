#### 一、热部署 Devtools

1. Adding devtools to your project

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-devtools</artifactId>
       <scope>runtime</scope>
       <optional>true</optional>
   </dependency>
   ```

2. Adding plugin to your pom.xml

   ```xml
    <build>
       <plugins>
         <plugin>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-maven-plugin</artifactId>
           <configuration>
             <fork>true</fork>
             <addResources>true</addResources>
           </configuration>
         </plugin>
       </plugins>
     </build>
   ```

   

3. Enabling automatic build
   ![image-20200313111136038](F:\Note\Java\SrpingCloud\imgs\devtools_settings_compiler.png)

4. Update the Value of

   ```
   ctrl+shift+Alt+/
   ```

   Click *Registry* , then click to check 

   - compiler.automake.allow.when.app.running
   - actionSystem.assertFocusAccessFormEdt

5. restart idea

#### 二、开发新module的步骤

1. new module
2. edit pom.xml
3. new application.yml
4. new SpringBootApplication main class
5. service: entities, service, controller
6. test

#### 三、RestTemplate

- 概述：RestTemplate提供了多种便捷访问远程http服务的方法，一种简单便捷的访问restful服务模板类，是Spring提供的用于访问Rest服务的客户端模板工具集

  ```java
  // 新建配置类
  @Configuration
  public class ApplicationContextConfig {
  	@Bean
  	public RestTemplate getRestTemplate(){
  		return new RestTemplate();
  	}
  }
  ```

  ```java
  @RestController
  @Slf4j
  public class OrderController {
  	public static final String PAYMENT_URL = "http://localhost:8001";
  
  	@Resource
  	private RestTemplate restTemplate;
  
  	@GetMapping("/consumer/payment/create")
  	public CommonResult<Payment> create(Payment payment){
          // 调用restTemplate的请求方法
  		return restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
  	}
  
  	@GetMapping("/consumer/payment/get/{id}")
  	public CommonResult<Payment> getPayment(@PathVariable("id") Long id){
  		return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
  	}
  }
  ```

#### 四、工程重构

1. 观察问题：系统中有重复的部分，重构

2. 新建module ‘cloud-api-commons’

3. 修改cloud-api-commons 的 pom.xml

4. 将重复的entities放到新的cloud-api-commons中

5. maven命令：clean 、install

6. 删除重复的entities，修改pom.xml文件，引用cloud-api-commons

   ```xml
   <dependency>
       <groupId>com.atguigu.springcloud</groupId>
       <artifactId>cloud-api-commons</artifactId>
       <version>${project.version}</version>
   </dependency>
   ```


#### 五、Eureka

1. 服务治理：Spring Cloud封装了Netflix公司开发的Eureka模块来实现服务治理。
   在传统的RPC远程调用框架中，管理每个服务与服务之间的依赖关系比较复杂，所以需要使用服务治理，管理服务与服务之间的依赖关系，可以实现服务调用、负载均衡、容错等功能，实现服务的发现与注册。
   
   - **RPC(远程过程调用)**：Remote Procedure Call。RPC就是从一台机器（客户端）上通过参数传递的方式调用另一台机器（服务器）上的一个函数或方法（可以统称为服务）并得到返回的结果
   
2. **Eureka服务搭建**：

   1. 新建module：cloud-eureka-server7001

   2. 改pom.xml，引入eureka-server

      ```xml
      <!-- eureka-server -->
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
      </dependency>
      ```

      

   3. 新建application.yml

      ```yml
      server:
        port: 7001
      
      eureka:
        instance:
          hostname: localhost # eureka服务端实例名称
        client:
          # false 表示不向注册中心注册自己
          register-with-eureka: false
          # 表示自己端就是注册中心，职责就是维护服务实例，并不需要检索服务
          fetch-registry: false
          service-url:
            defaultZone: http://${eureka.instace.hostname}:${server.port}/eureka/
      ```

      

   4. 新建SpringBootApplication启动类，在类上添加`@EnableEurekaServer`标签

   5. EurekaClient端cloud-provider-payment8001注册进EurekaServer成为服务提供者provider

      1. 修改pom.xml，新增dependency: spring-cloud-starter-netflix-eureka-client

         ```xml
         <dependency>
             <groupId>org.springframework.cloud</groupId>
             <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
         </dependency>
         ```

         

      2. 修改applicaiton.yml，新增eureka配置

         ```yml
         eureka:
           client:
             # 表示注册进EurekaServer
             register-with-eureka: true
             # 是否从EurekaServer中抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配置ribbon使用负责均衡
             fetch-registry: true
             service-url:
               defaultZone: http://localhost:7001/eureka
         ```

         

      3. 在SpringBootApplication启动类上添加`@EnableEurekaClient`标签

   6. EurekaClient端cloud-consumer-order80注册进EurekaServer成为服务消费者consumer。与第5步操作相同

3. Eureka集群环境配置

   1. 新建EurekaServer module `cloud-eureka-server7002`

   2. pom.xml与之前的EurekaServer相同

   3. application.yml需做修改，两个eureka server 7001与7002相互注册，修改hostname、defaultZone

      ```yml
      eureka:
        instance:
          hostname: eureka7001.com # eureka服务端实例名称
        client:
          # false 表示不向注册中心注册自己
          register-with-eureka: false
          # 表示自己端就是注册中心，职责就是维护服务实例，并不需要检索服务
          fetch-registry: false
          service-url:
            defaultZone: http://eureka7002.com:7002/eureka/
      ```

   4. consumer、provider的application.yml修改defaultZone

      ```yml
      eureka:
        client:
          # 表示注册进EurekaServer
          register-with-eureka: true
          # 是否从EurekaServer中抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配置ribbon使用负责均衡
          fetch-registry: true
          service-url:
            defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
      ```

4. provider 集群后，consumer调用的地址需修改为http:// + provider的Application名(eg.http://CLOUD-PAYMENT-SERVICE)，RestTemplate上需添加`@LoadBalanced`以保证负载均衡。

   ```java
   @Configuration
   public class ApplicationContextConfig {
   	@Bean
   	// 使用@LoadBalanced注解赋予了RestTemplate负载均衡的能力
   	@LoadBalanced
   	public RestTemplate getRestTemplate(){
   		return new RestTemplate();
   	}
   }
   ```
