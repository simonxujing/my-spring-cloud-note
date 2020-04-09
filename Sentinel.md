# 一、简介

​	Sentinel is a powerful flow control component that takes "flow" as the breakthrough point and covers multiple fields including flow control, concurrency limiting, circuit breaking, and adaptive system protection to guarantee the reliability of microservices.

# 二、下载、安装控制台程序

​	https://github.com/alibaba/sentinel/releases 下载到本地sentinel-dashboard-1.7.1.jar

​	运行命令 java -jar sentinel-dashboard-1.7.1.jar (注意端口为8080)

# 三、sentinel初始化监控

## 1、新建module，改pom

```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!--     sentinel-datasource-nacos 后续持久化用   -->
    <dependency>
        <groupId>com.alibaba.csp</groupId>
        <artifactId>sentinel-datasource-nacos</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
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
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.atguigu.springcloud</groupId>
        <artifactId>cloud-api-commons</artifactId>
        <version>${project.version}</version>
    </dependency>
</dependencies>
```

## 2、application.yml

```yml
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: http://192.168.56.101:1111
        # server-addr: localhost:8848
    sentinel:
      transport:
        # 配置sentinel dashboard地址
        dashboard: localhost:8080
        # 默认8719端口，假如被占用则从8719+1，直至找到未被占用的端口
        port: 8719
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

# 四、规则持久化

## 1、pom引入

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

## 2、修改application.yml

```yml
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.56.101:1111
        # server-addr: localhost:8848
    sentinel:
      transport:
        # 配置sentinel dashboard地址
        dashboard: localhost:8080
        # 默认8719端口，假如被占用则从8719+1，直至找到未被占用的端口
        port: 8719
      datasourc:
        ds1:
          nacos:
            server-addr: 192.168.56.101:1111
            dataId: cloudalibaba-sentinel-service
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
```

## 3、nacos中新增配置

```json
[
    {
        // 资源名称
        "resource": "rateLimit/byUrl",
        // 来源应用
        "limitApp": "default",
        // 阈值类型， 0表示线程数，1表示QPS
        "grade": 1,
        // 单机阈值
        "count": 1,
        // 流控模式，0表示直接，1表示关联，2表示链路
        "strategy": 0,
        // 流控效果，0表示快速关闭，1表示warm up，2表示排队等待
        "controlBehavior": 0,
        // 是否集群
        "clusterMode": false
    }
]
```

