---
title: Spring Cloud Alibaba Sentinel
categories: springcloud-alibaba
tags:
  - springcloud-alibaba
  - Sentinel
abbrlink: '5884e01'
date: 2022-02-02 12:00:00
---
Sentinel

开原地址：https://github.com/alibaba/Sentinel

下载：https://github.com/alibaba/Sentinel/releases

中文文档：https://sentinelguard.io/zh-cn/index.html

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

特性：

![](https://blog.lichenghao.cn/upload/2022/07/11e9-9ffc.png)



## 安装控制台

下载运行 jar 包启动即可

```sh
java -jar sentinel-dashboard-1.7.0.jar
```

访问地址：http://localhost:8080/    账号/密码   sentinel/sentinel

![](https://blog.lichenghao.cn/upload/2022/07/01081327.png)



## 初始化监控

增加一个新的模块用来测试 sentinel，cloudalibaba-sentinel-service8401

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

    <artifactId>cloudalibaba-sentinel-service8401</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- 服务注册发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--持久化用-->
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
    </dependencies>
</project>
```

配置文件 application.yml

```yaml
server:
  port: 8401
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: 172.16.127.3:8848
    sentinel:
      transport:
        dashboard: 172.16.127.3:8080
        # 默认 8719，如果被占用就+1
        port: 8719

management:
  endpoints:
    web:
      exposure:
        include: "*"
```

测试请求

```java

@RestController
public class SentinelController {

    @GetMapping("/testa")
    public String testA() {
        return "test a";
    }

    @GetMapping("/testb")
    public String testB() {
        return "test b";
    }
}
```

主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class SentinelService8401 {
    public static void main(String[] args) {
        SpringApplication.run(SentinelService8401.class, args);
    }
}
```

启动服务后，查看Nacos 注册中心和 Sentinel 监控中心，需要发送请求后，Sentinel 监控中心才会有数据

![](https://blog.lichenghao.cn/upload/2022/07/01084053.png)



## 流量规则

XX





