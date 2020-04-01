# SpringCloud Config

## 一、简介

SpringCloud Config为微服务架构中的微服务提供了集中化的外部配置支持，配置服务器为各种不同微服务应用的所有环境提供了一个中心化的外部配置。

分为服务端和客户端两部分。服务端也称为分布式配置中心，它是一个独立的微服务应用。用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口。

## 二、用途

- 集中管理配置文件
- 不同环境不同配置，动态化的配置更新，分环境部署比如dev/test/prod/beta/release
- 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉去配置自己的信息
- 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置
- 将配置信息以REST接口形式暴露

## 三、实操

### 1、在Github上新建名为springcloud-config的新Repository

### 2、本地硬盘clone

​	使用https地址：https://github.com/simonxujing/springcloud-config.git

### 3、新建config-dev.yml文件

### 4、commit、push config-dev.yml

### 5、new module & modify pom

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

### 6、application.yml

```yml
server:
  port: 3344

spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri: https://github.com/simonxujing/springcloud-config.git # github上的仓库名
          # 搜索目录
          search-paths:
            - spring-config
      # 分支名
      label: master

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

### 7、主启动类上添加`@EnableConfigServer`

### 8、修改hosts后访问：[http://config-3344.com:3344/master/config-test.yml](http://config-3344.com:3344/master/config-test.yml) 可获取到github上的配置信息

## 四、Config客户端配置与测试

### 1、application.yml 和 bootstrap.yml

​	SpringCloud会创建一个 Bootstrap Context作为Spring应用的Application Context的父上下文。初始化的时候，Bootstrap Context负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的Environment。 bootstrap.yml更先加载。

1. application.yml 是用户级的资源配置项
2. bootstrap.yml 是系统级，优先级更高

### 2、改pom

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

### 3、建bootstrap.yml

```yml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    # config 客户端配置
    config:
      label: master # 分支名称
      name: config  # 配置文件名称
      profile: dev # 读取后缀名称 上述3个综合：master分支上config-dev.yml 的配置文件被读取 http://config-3344.com:3344/master/config-dev.yml
      uri: http://localhost:3344 # 配置中心地址

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

### 4、主启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class ConfigClientMain3355 {
  public static void main(String[] args) {
    SpringApplication.run(ConfigClientMain3355.class, args);
  }
}
```

### 5、业务类

```java
@RestController
public class ConfigClientController {
    // 读取服务端的config.info配置
	@Value("${config.info}")
	private String configInfo;

	@GetMapping(value = "/configInfo")
	public String getConfigInfo(){
		return configInfo;
	}
}
```

## 五、config动态刷新

### 1、改pom，引入监控spring-boot-starter-actuator

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 2、修改yml，暴露监控端口

```yml
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

### 3、业务类上添加`@RefreshScope` 

```java
@RestController
@RefreshScope
public class ConfigClientController {
	@Value("${config.info}")
	private String configInfo;

	@GetMapping(value = "/configInfo")
	public String getConfigInfo(){
		return configInfo;
	}
}
```

### 4、需运维人员发送post请求，刷新3355

```
curl -X POST "http://localhost:3355/actuator/refresh"
```









