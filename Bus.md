# 一、Spring Cloud Bus简介

​	Srping Cloud Bus 是用来将分布式系统的节点与轻量级消息系统连接起来的框架。整合了Java的事件处理机制和消息中间件的功能，目前支持RabbitMQ、Kafka。能管理和传播分布式系统间的消息，就像一个分布式执行器，用于广播状态、事件推送等，也可作为微服务见的通信通道。

# 二、RabbitMQ

## 1、下载安装Erlang（RabbitMQ运行需要）

## 2、下载安装RabbitMQ

## 3、进去RabbitMQ安装目录的sbin目录，执行命令

### 1）启动插件

```
rabbitmq-plugins enable rabbitmq_management
```

### 2）访问地址

```
http://localhost:15672
```

### 3) 登录系统

默认账号密码为guest / guest

# 三、Bus动态刷新全局广播配置实现

## 1、config server、client中pom引入消息总线RabbitMQ支持

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

## 2、server的application.yml中添加RabbitMQ配置

```yml
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
  # rabbitmq 相关配置
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

# rabbitmq 相关配置，暴露bus刷新配置的端点
management:
  endpoints:
    web:
      exposure:
        include: "bus-refresh"
```

## 3、client的bootstrap.yml修改

```yml
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
  # rabbitmq 相关配置
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

## 4、post请求至server的bus-refresh

```
curl -X POST "http://localhost:配置中心端口号/actuator/bus-refresh"
```

## 四、定点通知

## 1、destination指定需要更新的服务或实例

```
curl -X POST "http://localhost:配置中心端口号/actuator/bus-refresh/{destination}"
```

