---
title: srping cloud OpenFeign
categories: springcloud
tags:
  - springcloud
  - OpenFeign
abbrlink: e37cdf78
date: 2022-02-02 12:00:00
---
## 资料地址

OpenFeign

开源地址：https://github.com/OpenFeign/feign

文档地址：https://spring.io/projects/spring-cloud-openfeign



## 定义及功能

是一个声明式的Web 服务客户端，只需要创建一个接口并在接口上添加注解就可以去使用。

在 Feign 的实现下我们只需要创建和服务提供方一样的接口并用注解来配置它，即可完成对服务提供方的服务的绑定，简化了代码的开发量。

同时和集成了 Ribbon 的功能，能实现客户端的负载均衡。

## 使用

### POM 文件

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
    <artifactId>cloud-consumer-openfeign-order80</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- openfeign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
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

### yml

```yaml
server:
  port: 80

eureka:
  client:
    # 注册进中心
    register-with-eureka: false
    # 从 Eureka Server 抓取自己的注册信息。单机无所谓，集权环境下必须为ture。
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```



### 增加一个业务接口

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE") // 注册的服务
public interface PaymentService {

    @GetMapping("/payment/info/{id}") // 调用的服务路径
    CommentResult<Payment> info(@PathVariable("id") Long id);
}
```

调用

```java
package cn.lichenghao.springcloud.controller;

import cn.lichenghao.springcloud.entity.CommentResult;
import cn.lichenghao.springcloud.entity.Payment;
import cn.lichenghao.springcloud.service.PaymentService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@RequestMapping("/order")
public class OrderController {

    @Resource
    private PaymentService paymentService;

    @GetMapping("/consumer/payment/info/{id}") 
    public CommentResult<Payment> info(@PathVariable("id") Long id) {
        return paymentService.info(id);
    }
}
```

### 启动类

```java
package cn.lichenghao.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration;
import org.springframework.cloud.openfeign.EnableFeignClients;

/**
 * 演示OpenFeign服务调用
 *
 * @author chenghao.li
 */
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class
        , DataSourceTransactionManagerAutoConfiguration.class})
@EnableFeignClients
public class OpenFeignOrder {
    public static void main(String[] args) {
        SpringApplication.run(OpenFeignOrder.class, args);
    }
}
```

## 超时控制

如果消费者消费的时间和调用服务的时间不一样，可能存在时间差，这样消费者可能认为超时了，实际上还在服务调用中。OpenFeign默认 1s 超时，如果服务调用超过 1s 的话就会报错。

因为OpenFeign支持 Ribbon 所以在配置文件中增加配置即可：

连接等待和超时等待

```java
server:
  port: 80

eureka:
  client:
    # 注册进中心
    register-with-eureka: false
    # 从 Eureka Server 抓取自己的注册信息。单机无所谓，集权环境下必须为ture。
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka

ribbon:
  # 连接等待时间
  ConnectTimeout: 5000
  # 超时时间
  ReadTimeout: 5000
```

## 日志打印

Feign 提供了日志打印的功能，通过修改日志级别可以打印出 HTTP 调用的细节日志，也就是接口的调用情况。

级别包括：

- NONE：默认，不显示任何日志
- BASIC：仅记录请求的方法，URL，响应状态码及时间
- HEADERS：BASIC+请求和结果的头信息
- FULL：HEADERS+请求和响应的元数据信息

增加配置类，设置日志级别

```java
package cn.lichenghao.springcloud.config;

import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 自定义Feign日志级别
 */
@Configuration
public class FeignConfig {

    @Bean
    Logger.Level logLevel() {
        return Logger.Level.FULL;
    }
}
```

修改配置文件，指定日志输出

```yaml
server:
  port: 80

eureka:
  client:
    # 注册进中心
    register-with-eureka: false
    # 从 Eureka Server 抓取自己的注册信息。单机无所谓，集权环境下必须为ture。
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka

ribbon:
  # 连接等待时间
  ConnectTimeout: 5000
  # 超时时间
  ReadTimeout: 5000

logging:
  level:
    # 指定日志的输出级别
    cn.lichenghao.springcloud.service.PaymentService: debug
```













