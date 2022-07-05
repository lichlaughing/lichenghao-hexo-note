---
title: spring cloud stream
categories: springcloud
tags:
  - springcloud
  - stream
abbrlink: 2c847fdd
date: 2022-02-02 12:00:00
---
## 什么要用 Spring Cloud Stream ？

上面利用 Bus 整合了 rabbitmq 实现了基于消息的配置刷新，但是消息中间件不止 rabbitmq，还有 ActiveMq、RocketMq、Kafka 等，我们不可能针对每种中间件都进行一次编码，尤其是如果我们切换了中间件还要再重新编码一次，这样耦合性很高，很难受！

那么 Spring Cloud Stream 解决了这个问题，我们不需要关注 MQ的具体细节了，只需要一种适配器的绑定，就可以使用任何的 MQ。也就是说：屏蔽底层中间件的差异，降低切换成本，统一消息的编程模型。通过 Binder来实现消息的适配。如下所示的**RabbitMQ Binder**。

![](https://blog.lichenghao.cn/upload/2022/07/rabbit-binder.png)

Stream中的消息通信方式遵循了发布-订阅模式（Topic 主题广播，在 Rabbitmq 中就是 Exchange，Kafka中就是 Topic）。

文档：https://spring.io/projects/spring-cloud-stream

## 如何使用？

准备好 RabbitMq ；三个模块：一个生产者，两个消费者。



### 生产者

pom，增加 spring-cloud-starter-stream-rabbit

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>lichcloud</artifactId>
        <groupId>cn.lichenghao.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-stream-rabbitmq-provider8801</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <!-- eureka client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- 自定义通用 -->
        <dependency>
            <groupId>cn.lichenghao.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
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
        </dependency>
    </dependencies>
</project>
```



配置文件 application.yml

```yaml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders:
        defaultMq: # 定义的名称，用于 bingdings
          type: rabbit # 消息中间件类型
          environment: # 设置mq的基础信息
            spring:
              rabbitmq:
                host: 172.16.127.3
                port: 5672
                username: admin
                password: admin
      bindings:
        output: # 表示生产者
          destination: studyExchange # 通道名称
          content-type: application/json # 消息类型
          binder: defaultMq # 绑定上面的定义信息
  rabbitmq:
    host: 172.16.127.3
    port: 5672
    username: admin
    password: admin
eureka:
  client:
    # 注册进中心
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:6001/eureka
  instance:
    instance-id: cloud-stream-provider-8801
    prefer-ip-address: true
```

业务类，定义一个发送消息的接口

```java
public interface MessageProvider {
    String send();
}

/**
 * 消息发送
 */
@Component
@EnableBinding(Source.class) // 定义发送管道
@Slf4j
public class MessageProviderImpl implements MessageProvider {

    @Resource
    @Output(Source.OUTPUT)
    private MessageChannel messageChannel;

    @Override
    public String send() {
        String uuid = UUID.randomUUID().toString();
        log.info("MessageProvider send :{}", uuid);
        // 发送消息
        messageChannel.send(MessageBuilder.withPayload(uuid).build());
        return uuid;
    }
}
```



启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class StreamProvider8801 {
    public static void main(String[] args) {
        SpringApplication.run(StreamProvider8801.class, args);
    }
}
```



写个 controller 测试下

```java
@RestController
public class StreamProviderController {

    @Resource
    private MessageProvider messageProvider;

    @GetMapping("/sendMsg")
    public String sendMsg() {
        return messageProvider.send();
    }
}
```



发送请求，http://localhost:8801/sendMsg  后台就会打印输出，同时可以看到 Rabbitmq 控制台的变化

![](https://blog.lichenghao.cn/upload/2022/07/29145627.png)

### 消费者

pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>lichcloud</artifactId>
        <groupId>cn.lichenghao.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-stream-rabbitmq-consumer8802</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <!-- eureka client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- 自定义通用 -->
        <dependency>
            <groupId>cn.lichenghao.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
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
        </dependency>
    </dependencies>
</project>
```

配置文件 application.yml

```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders:
        defaultMq: # 定义的名称，用于 bingdings
          type: rabbit # 消息中间件类型
          environment: # 设置mq的基础信息
            spring:
              rabbitmq:
                host: 172.16.127.3
                port: 5672
                username: admin
                password: admin
      bindings:
        input: # 表示消费者
          destination: studyExchange # 通道名称
          content-type: application/json # 消息类型
          binder: defaultMq # 绑定上面的定义信息
  rabbitmq:
    host: 172.16.127.3
    port: 5672
    username: admin
    password: admin
eureka:
  client:
    # 注册进中心
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:6001/eureka
  instance:
    instance-id: cloud-stream-consumer-8802
    prefer-ip-address: true
```

接收消息业务类

````java
@Component
@EnableBinding(Sink.class)
@Slf4j
public class ReceiveService {

    @Value("${server.port}")
    private String port;

    @StreamListener(Sink.INPUT)
    public void receiveMsg(Message<String> msg) {
        String payload = msg.getPayload();
        log.info("ReceiveService port:{} receive:{}", port, payload);
    }
}
````

主启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class StreamConsumer8802 {
    public static void main(String[] args) {
        SpringApplication.run(StreamConsumer8802.class, args);
    }
}
```

按照上面的配置，搞多个微服务消费者，cloud-stream-rabbitmq-consumer8802 ，cloud-stream-rabbitmq-consumer8803

启动生产者和这两个消费者，然后生产者产生多个消息后，这俩消费者都会接收到消息！

```java
15:33:56.408  INFO 9980 --- [nio-8801-exec-5] c.l.s.service.impl.MessageProviderImpl   : MessageProvider send :0c5350db-680f-47ed-849d-333873719fe4
15:33:57.398  INFO 9980 --- [nio-8801-exec-6] c.l.s.service.impl.MessageProviderImpl   : MessageProvider send :20459405-3b70-47f3-bd51-6261f7be0c2b
```

消费者8002

```java
15:33:56.421  INFO 10178 --- [NOUKEY8t_yWSA-1] c.l.springcloud.service.ReceiveService   : ReceiveService port:8802 receive:0c5350db-680f-47ed-849d-333873719fe4
15:33:57.401  INFO 10178 --- [NOUKEY8t_yWSA-1] c.l.springcloud.service.ReceiveService   : ReceiveService port:8802 receive:20459405-3b70-47f3-bd51-6261f7be0c2b
```

消费者8003

```java
15:33:56.421  INFO 10201 --- [VuHdf0IcF09Fw-1] c.l.springcloud.service.ReceiveService   : ReceiveService port:8803 receive:0c5350db-680f-47ed-849d-333873719fe4
15:33:57.400  INFO 10201 --- [VuHdf0IcF09Fw-1] c.l.springcloud.service.ReceiveService   : ReceiveService port:8803 receive:20459405-3b70-47f3-bd51-6261f7be0c2b
```



## 分组消费&持久化

通过上面两个消费者发现，同一个消息被这两个消费者都消费了，出现了重复消费的问题。

可以通过 Stream 的 group（分组）消费来解决。

- 处于同一个分组中的消费者是竞争关系，消息只会被其中一个消费者消费。
- 不同的组的话，就会重复消费

默认情况下，就是不同的组，如下 rabbitmq 所示：

可以看出重复消费的问题在于默认的分组是不同的，只需要将分组名称设置为一样即可。

![](https://blog.lichenghao.cn/upload/2022/07/29160918.png)



采用分组的方式解决重复消费的问题，也特别的简单，只需要需改配置文件增加`group`参数即可

```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders:
        defaultMq: # 定义的名称，用于 bingdings
          type: rabbit # 消息中间件类型
          environment: # 设置mq的基础信息
            spring:
              rabbitmq:
                host: 172.16.127.3
                port: 5672
                username: admin
                password: admin
      bindings:
        input: # 表示消费者
          destination: studyExchange # 通道名称
          content-type: application/json # 消息类型
          binder: defaultMq # 绑定上面的定义信息
          group: lichcloud
  rabbitmq:
    host: 172.16.127.3
    port: 5672
    username: admin
    password: admin
eureka:
  client:
    # 注册进中心
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:6001/eureka
  instance:
    instance-id: cloud-stream-consumer-8802
    prefer-ip-address: true
```

将两个消费者设置相同的组后，这个时候就不会出现重复消费了，变成了轮询消费。

同理如果把分组设置成不一样了，就和默认的一样了，会出现重复消费。

![](https://blog.lichenghao.cn/upload/2022/07/29161852.png)



如果增加了`group`属性后，就会自动支持消息的持久化。那么只要消息没有被消费那么消息就会永久保留，指导被消费。即便服务宕机没有正确消费消息，待重后依然可以消费对应的消息。







