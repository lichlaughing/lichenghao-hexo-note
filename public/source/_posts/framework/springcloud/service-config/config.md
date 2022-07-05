---
title: srping cloud config 服务配置中心
categories: springcloud
tags:
  - springcloud
  - config
abbrlink: 61a5fc19
date: 2022-02-02 12:00:00
---
文档：https://spring.io/projects/spring-cloud-config



SpringCloud Config 为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。



作用

- 集中管理配置文件
- 分环境部署 dev/test/prod
- 运行期间动态调整
- 配置发生改变，服务不需要重启即可感知新的配置
- 配置信息以 REST 接口形式暴露
- 支持服务端和客户端



## config 服务端配置

pom 增加，spring-cloud-config-server

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



增加配置文件，以 gitee 来存放统一的配置文件，配置文件名称格式：application-profile.yml 

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
          uri: https://gitee.com/lichenghao/spring-cloud-config.git  # 仓库地址
          search-paths:
            - spring-cloud-config  # 仓库名称
          username: xxxx
          password: xxxx
      label: master  # 分支

eureka:
  client:
    service-url:
      defaultZone: http://localhost:6001/eureka
```



主启动类

```java
package cn.lichenghao.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigCenter {
    public static void main(String[] args) {
        SpringApplication.run(ConfigCenter.class, args);
    }
}
```



启动后访问：http://localhost:3344/master/config-dev.yml

结果

```properties
config:
  info: SpringCloud config-dev.yml version=1
```





## config 客户端配置

### 基础配置

pom 增加，spring-cloud-starter-config

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



增加配置文件，bootstrap.yml

> application.yml 是用户级的资源配置；bootstrap.yml是系统级别的配置文件，优先级更高；SpringCloud 会创建一个"Bootstrap Context"作为 Spring 应用的"Application Context"的父上下文。初始的化的时候Bootstrap Context负责从外部源加载配置属性并解析。这两个上下文共享一个外部获取的"Enviroment"。



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

eureka:
  client:
    service-url:
      defaultZone: http://localhost:6001/eureka
```

业务类

```java
/**
 * 测试获取配置信息
 */
@RestController
public class ClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

主启动类

```java
/**
 * 配置中心演示，客户端
 */
@SpringBootApplication
@EnableEurekaClient
public class ConfigClient {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClient.class, args);
    }
}
```



启动后访问：http://localhost:3355/config

```properties
SpringCloud config-dev.yml version=1
```

### 动态刷新配置（手动）

我们手动修改仓库中的配置文件中的配置信息，3344 可以感知到，但是 3355并不能感知到。如果重启 3355 可以重新拉取，但是如果不想重新启动项目的话，可以使用手动刷新。



客户端增加监控。pom 增加下：

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

暴露监控的信息

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

eureka:
  client:
    service-url:
      defaultZone: http://localhost:6001/eureka
      
# 配置监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"  # 监控所有
```



增加刷新注解，@RefreshScope

```java
/**
 * 测试获取配置信息
 */
@RestController
@RefreshScope
public class ClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

手动刷新配置信息请求：

```bash
curl -X POST "http://localhost:3355/actuator/refresh"
```

![](https://blog.lichenghao.cn/upload/2022/07/28164446.png)

这个时候在获取配置文件信息，即可获得最新的数据。





















