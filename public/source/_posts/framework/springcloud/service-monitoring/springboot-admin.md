---
title: springcloud 服务监控 springboot admin
categories: springcloud
tags:
  - springcloud
  - springboot-admin
abbrlink: a50213ab
date: 2022-02-02 12:00:00
---
## 服务监控配置

添加主要依赖

因为使用的 nacos 注册中心，springboot admin 会主动去注册中心查询服务。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springboot-admin-monitoring</artifactId>
        <groupId>cn.lichenghao</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>admin</artifactId>
    <name>admin</name>

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
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-server</artifactId>
            <version>2.2.3</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>
</project>
```

配置文件配置下 nacos 的地址

```yaml
server:
  port: 8000
  servlet:
    context-path: /
spring:
  application:
    name: ADMIN-SERVICE
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.1.240
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

主启动类，开启服务监控

```java
/**
 * 服务监控
 *
 * @author chenghao.li
 */
@EnableAdminServer
@SpringBootApplication
@EnableDiscoveryClient
public class AdminServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(AdminServerApplication.class, args);
    }
}
```

访问：http://ip:8000/  

![](https://blog.lichenghao.cn/upload/2022/07/12142123.png)



如果想要完整的监控信息，还需要微服务开放端点监控。在需要监控的微服务上增加配置：

服务添加依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

配置文件

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

然后在 Admin中就可以监控注册到 nacos 中的服务。

## 访问权限

使用用户名密码登录控制台。





