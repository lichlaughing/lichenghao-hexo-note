---
title: spring cloud sleuth 服务链路跟踪
categories: springcloud
tags:
  - springcloud
  - zipkin
  - sleuth
abbrlink: b4c691a9
date: 2022-02-02 12:00:00
---


## Spring Cloud Sleuth 概述

提供了一套完整的服务跟踪的解决方案，并且兼容了支持了 zipkin。

开源地址：https://github.com/spring-cloud/spring-cloud-sleuth

文档：https://spring.io/projects/spring-cloud-sleuth



## 链路监控搭建

### 监控平台

下载 zipkin，下载地址：https://search.maven.org/ 搜索 zipkin server 即可，然后下载可执行 jar 文件。(或者：https://wwb.lanzoul.com/b01vbrvjg  密码:bqku)

`java -jar zipkin-server-2.23.16-exec.jar` 测试运行。

> 通常写个启动脚本( start.sh ) 后台启动，内容如下：
>
> ```sh
> nohup java -jar /opt/module/zipkin-server/zipkin-server-2.23.16-exec.jar > log.file 2>&1 &
> ```
>
> 有启动就有关闭，同样写个关闭脚本： [Shell脚本-关闭一个 java 进程](http://localhost:3000/#/tool/linux/shell?id=%e5%85%b3%e9%97%ad%e4%b8%80%e4%b8%aa-java-%e8%bf%9b%e7%a8%8b)

运行后，访问 http://localhost:9411/zipkin/

![](https://blog.lichenghao.cn/upload/2022/07/30085442.png)



一条调用链路通过父Id关联起来如下图所示：

![](https://blog.lichenghao.cn/upload/2022/07/parents.png)

### 调整微服务的生产和消费者

生产者：

调整 pom，增加 spring-cloud-starter-zipkin

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
    <artifactId>cloud-provider-payment8001</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- 链路跟踪 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
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
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
</project>
```

调整配置文件 application.yml ，增加 zipkin 和 sleuth 注意对齐格式。

```yaml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service
  zipkin:
    base-url: http://127.0.0.1:9411
  sleuth:
    sampler:
      # 采样率在 0-1 之间，1 表示全部采集
      probability: 1
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://172.16.127.3:3306/lichcloud?useUnicode=true&useSSL=false&characterEncoding=utf-8
    username: root
    password: Root.123456

eureka:
  client:
    # 注册进中心
    register-with-eureka: true
    # 从 Eureka Server 抓取自己的注册信息。单机无所谓，集权环境下必须为ture。
    fetch-registry: true
    service-url:
      # defaultZone: http://eureka7001.com:7001/eureka,http://eureka7001.com:7002/eureka
      defaultZone: http://localhost:6001/eureka
  instance:
    instance-id: payment8001
    prefer-ip-address: true

mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: cn.lichenghao.springcloud.entity
```



调整消费者

pom，增加 spring-cloud-starter-zipkin 

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
    <artifactId>cloud-consumer-order80</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- 链路跟踪 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
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
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
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
  port: 80

spring:
  application:
    name: cloud-order-service
  zipkin:
    base-url: http://127.0.0.1:9411
  sleuth:
    sampler:
      # 采样率在 0-1 之间，1 表示全部采集
      probability: 1
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://172.16.127.3:3306/lichcloud?useUnicode=true&useSSL=false&characterEncoding=utf-8
    username: root
    password: Root.123456
eureka:
  client:
    # 注册进中心
    register-with-eureka: true
    # 从 Eureka Server 抓取自己的注册信息。单机无所谓，集权环境下必须为ture。
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:6001/eureka

mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: cn.lichenghao.springcloud.entity
```

然后消费者请求下，在zipkin 的监控页面就能看到链路信息，如下所示：

![](https://blog.lichenghao.cn/upload/2022/07/30093252.png)