---
title: Spring Cloud Bus
categories: springcloud
tags:
  - springcloud
  - bus
abbrlink: 1c50d52c
date: 2022-02-02 12:00:00
---
## 概述

作用：Spring Cloud Bus 能管理和传播分布式系统之间的消息，像一个分布式执行器，可用于广播状态更改，事件推动等，也可以当做微服务间的通信通道。

Bus 支持两种消息代理：RabbitMQ 和 Kafka。

> 如果我们把上面的手动刷新通过 Bus 变成自定刷新就很好理解了：就是把我们手动刷新的动作，通过消息中间件的功能（订阅通知），变成了自动刷新。



什么是消息总线？

在微服务架构中，通常会使用轻量级的消息代理构建一个共用的消息主题，并让系统中的所有微服务实例都连接上来。由于该主题中产生的消息会被所有的实例监听和消费，所以称之为消息总线。在总线上的各个实例，都可以方便的广播一些需要让其他连接该主题的实例都知道的消息。

## 配置 RabbitMQ

软件下载：https://wwb.lanzoul.com/b01vbnmhg    密码:26m9

见文档：[centos7-云服务器安 rabbitmq](https://note.lichenghao.cn/#/tool/linux/centos7-%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%89rabbitmq)



## 配置全局动态刷新

消息刷新有两种形式：

1. 利用消息总线触发一个客户端`/bus/refresh`，而刷新所有客户端的配置。

2. 利用消息总线触发一个服务端ConfigServer的`/bus/refresh`，而刷新所有客户端的配置。

通常选择 2 的方式，因为 1 违反了单一原则，不建议。

配置广播通知

### 服务端

pom，增加 spring-cloud-starter-bus-amqp

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

    <artifactId>cloud-config-center-3344</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- 消息总线RabbitMQ 依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
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

配置文件，增加 mq 配置 和 配置监控端点

```yaml
server:
  port: 3344

spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/lichenghao/spring-cloud-config.git
          search-paths:
            - spring-cloud-config
          username: lichlaughing
          password: lichenghao.88888
      label: master
  # rabbitmq
  rabbitmq:
    username: admin
    password: admin
    host: 172.16.127.3
    port: 5672

eureka:
  client:
    service-url:
      defaultZone: http://localhost:6001/eureka

# 配置监控端点
management:
  endpoints:
    web:
      exposure:
        include: "bus-refresh"
```



### 客户端

pom，增加 spring-cloud-starter-bus-amqp

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

    <artifactId>cloud-config-client-3355</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- 消息总线RabbitMQ 依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
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

配置文件

```yaml
server:
  port: 3355

spring:
  application:
    name: cloud-config-client
  cloud:
    config:
      label: master # 分支
      name: config # 配置文件名称
      profile: dev # 配置文件后缀名称。 config-dev.yml
      uri: http://localhost:3344 # 配置中心地址
  # rabbitmq
  rabbitmq:
    username: admin
    password: admin
    host: 172.16.127.3
    port: 5672

eureka:
  client:
    service-url:
      defaultZone: http://localhost:6001/eureka

# 配置监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```



修改git 上的配置文件内容，然后刷新配置中心的配置，客户端就会自动刷新。

刷新配置中心的请求

```bash
curl -X POST "http://配置中心地址:配置中心端口/actuator/bus-refresh"

curl -X POST "http://localhost:3344/actuator/bus-refresh"
```



## 配置单点刷新

如果只想要刷新其中的某一个节点，可以在全局刷新命令的基础上增加参数

```bash
curl -X POST "http://配置中心地址:配置中心端口/actuator/bus-refresh/{node:port}"
```

例如：cloud-config-client （微服务名称）：端口，这样就会只刷新该节点的配置信息

```bash
curl -X POST "http://配置中心地址:配置中心端口/actuator/bus-refresh/{cloud-config-client:3355}"
```





