# 一、Stream简介

​	Spring Cloud Stream 是一个构建消息驱动微服务的框架。屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型。目前只支持RabbitMQ、kafka

# 二、标准流程套路

## 1、Binder

​	连接中间件，屏蔽差异

## 2、Channel

​	通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel对队列进行配置。

## 3、Source 、Sink

​	从Stream发布消息就是输出，接收消息就是输入。

# 三、Stream消息驱动的生产者

## 1、pom

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
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
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
</dependencies>
```

## 2、application.yml

```yml
server:
  port: 8801
spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息
        defaultRabbit: # 表示顶一的名称，用于binding整合
          type: rabbit # 消息组件的类型
          environment:
            string:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务整合处理
        output: # 通道名称
          destination: studyExchange # 要使用的exchange名称定义
          content-type: application/json # 消息类型，本次为json，文本可设置"text/plain"
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳时间间隔（默认为30秒）
    lease-expiration-duration-in-seconds: 5
    instance-id: send-8801.com # 在信息列表时显示主机名称
    prefer-ip-address: true # 访问的路径变为ip地址
```

## 3、service实现类

```java
import com.atguigu.springcloud.service.IMessageProvider;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.support.MessageBuilder;

import javax.annotation.Resource;
import java.util.UUID;

/**
 * @author: Isalpha
 * @date: 2020-4-1 10:50
 * @description:
 */
@EnableBinding(Source.class) // 定义消息的推送管道
public class MessageProviderImpl implements IMessageProvider {

  @Resource private MessageChannel output; // 消息发送管道

  @Override
  public String send() {
    String serial = UUID.randomUUID().toString();
    output.send(MessageBuilder.withPayload(serial).build());
    System.out.println("serial:" + serial);
    return null;
  }
}
```

# 四、Stream消息驱动之消费者

## 1、pom 同生产者的pom

## 2、application.yml

```yml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitMQ的服务信息
        defaultRabbit: # 表示定义的名称，用于binding的整合
          type: rabbit # 消息中间件类型
          environment: # 设置rabbitMQ的相关环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设为text/plain
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
#          group: spectrumrpcA

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的间隔时间，默认30
    lease-expiration-duration-in-seconds: 5 # 超过5秒间隔，默认90
    instance-id: receive-8802.com #主机名
    prefer-ip-address: true # 显示ip
```

## 3、实现类

 ```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Component;

/**
 * @author: Isalpha
 * @date: 2020-4-1 14:27
 * @description:
 */
@Component
@EnableBinding(Sink.class)
public class ReceiveMessageController {
  @Value("${server.port}")
  private String serverPort;

  @StreamListener(Sink.INPUT)
  public void input(Message<String> message) {
    System.out.println("消费者1号，接收到消息：" + message.getPayload() + "\t port:" + serverPort);
  }
}
 ```

# 五、Stream重复消费

## 1、案例分析：

​	订单系统是集群部署，都会从RabbitMQ中获取订单信息。同一个订单被两个服务获取到，那么就会造成数据错误。可使用Stream消息分组解决。

## 2、解决方法

### 	自定义分组

```yml
spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitMQ的服务信息
        defaultRabbit: # 表示定义的名称，用于binding的整合
          type: rabbit # 消息中间件类型
          environment: # 设置rabbitMQ的相关环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设为text/plain
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
          group: spectrumrpcA 
```





