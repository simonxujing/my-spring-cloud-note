# 一、Spring Cloud Alibaba

## 1、用途

- 服务限流降级：默认支持servlet、Feign、RestTemplate、Dubbo、RocketMQ限流降级功能的接入，可以运行时通过控制台实时修改限流降级规则，还支持查看限流降级Metrics监控
- 服务注册与发现：视频Spring Cloud服务注册与发现标准，默认继承了Ribbon的支持
- 分布式配置管理：支持分布式系统的外部化配置，配置更改时自动刷新
- 消息驱动能力：级域Spring Cloud Stream为微服务应用构建消息驱动能力
- 阿里云对象存储：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任务应用、任务时间、任务地点存储和访问任意类型的数据
- 分布式任务调度：提供妙计、精准、高可靠、高可用的定时（基于Cron表达式）任务调度服务。同时提供分布式的任务执行模式，如网格任务。网格任务支持海量子任务均匀分配到所有Worker（schedulerx-client）上分析

# 二、Nacos

## 1、简介

​	Naming Configuration Service

​	Nacos (official site: [http://nacos.io](http://nacos.io/)) is an easy-to-use platform designed for dynamic service discovery and configuration and service management. It helps you to build cloud native applications and microservices platform easily.

## 2、安装运行Nacos

​	下载安装后，运行bin/startup.cmd，访问localhost:8848/nacos。

# 三、Nacos消生产者

## 1、pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>

    <!--一般为通用配置-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>

    <dependency>
        <groupId>com.atguigu.springcloud</groupId>
        <artifactId>cloud-api-commons</artifactId>
        <version>${project.version}</version>
    </dependency>
</dependencies>
```

## 2、applicatiom.xml

```yml
server:
  port: 9003

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848

management:
  endpoints:
    web:
      exposure:
        include: "*"
```

## 3、启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ProviderNacosApplication9003 {
  public static void main(String[] args) {
	  SpringApplication.run(ProviderNacosApplication9003.class, args);
  }
}
```

# 四、Nacos消费者

## 1、pom同nacos生产者

​	需引入nacos-discovery

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

## 2、application.yml

```yml
server:
  port: 8083

spring:
  application:
    name: nacos-payment-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848

service-url:
  nacos-user-service: http://nacos-payment-provider
```

## 3、主启动类 `@EnableDiscoveryClient`

## 4、RestTemplate配置类

```java
@Configuration
public class ApplicationContextConfig {

	@Bean
	@LoadBalanced
	public RestTemplate getRestTemplate(){
		return new RestTemplate();
	}
}
```

## 5、controller

```java
@RestController
public class OrderNacosController {
  @Resource private RestTemplate restTemplate;

  @Value("${service-url.nacos-user-service}")
  private String serverUrl;

  @GetMapping(value = "/consumer/payment/nacos/{param}")
  public String paymentInfo(@PathVariable("param") String param) {
    return restTemplate.getForObject(serverUrl + "/payment/nacos/" + param, String.class);
  }
}
```

# 五、nacos linux集群

## 1、安装nacos

​	https://nacos.io/zh-cn/docs/quick-start.html

## 2、安装mysql

​	[https://www.cnblogs.com/kasnti/p/11929030.html](https://www.google.com/url?q=https://www.cnblogs.com/kasnti/p/11929030.html&sa=D&usd=2&usg=AOvVaw26abncxVAbG34r62p9dc18) 

​	安装完成后，create database nacos_config，执行source /nacos/conf/nacos-mysql.sql

### 3、修改application.properties

```properties
spring.datasource.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
db.url.0=jdbc:mysql://localhost:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=user
db.password=password
```

## 4、修改cluster.conf

```hostname -i``` 或 ```ip addr```查看ip，修改cluster.conf时，使用该ip地址 192.168.xxx.xxx:port

## 5、修改nacos脚本 startup.sh

​	希望修改为传递不同的端口号启动不同的nacos实例

```sh
while getopts ":m:f:s:p:" opt
do
	case $opt in
		m)
			MODE=$OPTARG;;
		f)
			FUNCTION_MODE=$OPTARG;;
		s)
			SERVER=$OPTARG;;
		p)
			PORT=$OPTARG;;
			
....
```

```sh
# start
...

nohup $JAVA -Dserver.port=${PORT} ${JAVA_OPT}

...
```

`./startup.sh -p 3333` 启动端口号为3333的nacos服务。

## 6、遇到的问题

### 1）bin目录中出现hs_er rxxx.log

```xml
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 1073741824 bytes for committing reserved memory.
# Possible reasons:
#   The system is out of physical RAM or swap space
#   The process is running with CompressedOops enabled, and the Java Heap may be blocking the growth of the native heap
# Possible solutions:
#   Reduce memory load on the system
#   Increase physical memory or swap space
#   Check if swap backing store is full
#   Decrease Java heap size (-Xmx/-Xms)
#   Decrease number of Java threads
#   Decrease Java thread stack sizes (-Xss)
#   Set larger code cache with -XX:ReservedCodeCacheSize=
# This output file may be truncated or incomplete.
#
#  Out of Memory Error (os_linux.cpp:2763), pid=30009, tid=0x00007fd480668700
#
# JRE version:  (8.0_242-b08) (build )
# Java VM: OpenJDK 64-Bit Server VM (25.242-b08 mixed mode linux-amd64 compressed oops)
# Core dump written. Default location: /opt/nacos/bin/core or core.30009
#
```

应该是内存太小了，我修改了vm的设置解决。

![image-20200408114454261](F:\Note\Java\my-spring-cloud-note\imgs\vmSettings.png)

### 2）Nacos 1.0.1内置的connector是 `mysql-connector-java-5.1.34` ，该connector无法连MySQL 8.0

下载支持MySQL 8.0的connector，例如：`mysql-connector-java-8.0.16` 。在Nacos的 `plugins` 目录下创建 `mysql` 目录，并将下载的connector扔到该目录即可。

# 六、nginx配置

```xml
upstream cluster{
    server 127.0.0.1:3333;
    server 127.0.0.1:4444;
    server 127.0.0.1:5555;
}

# limit_zone crawler $binary_remote_addr 10m;
# 下面是server虚拟主机的配置
server
{
    listen 1111; # 监听端口
    server_name localhost; #域名
    location / {
    proxy_pass http://cluster;
    }
    access_log off;
}
```



