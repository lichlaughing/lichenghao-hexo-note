---
title: spring cloud gateway 服务网关
categories: springcloud
tags:
  - springcloud
  - gateway
abbrlink: '53e64191'
date: 2022-02-02 12:00:00
---
## 简介



文档地址：https://spring.io/projects/spring-cloud-gateway

SpringCloud Gateway作为 SpringCloud生态系统中的网关，目标是代替 Zuul，在 Spring Cloud2.0 以上的版本没有对新版本的 Zuul2.0以上的最新版本进行集成。为了提高网关的性能，SpringCloud Gateway基于 WebFlux 框架实现，而 WebFlux框架底层则使用了高性能的 Reactor模式通信框架 Netty。

SpringCloud Gateway的目标提供统一的路由方式且基于 Filter 链的方式提供了网关的基本功能，如鉴权，监控，限流等。



## 核心概念

路由（Route）：构建网关的基本模块，它由 ID，目标 URL，一些列的断言和过滤组成，如果断言为真在匹配路由

断言（Predicate）：java.util.function.Predicate。如果请求和断言匹配则进行路由

过滤（Filter）：使用过滤器，可以对请求路由前和之后对请求进行修改



## 工作流程

客户端向网关发出请求。

如果网关处理程序映射确定请求与路由匹配，则将其发送到网关Web处理程序。此处理程序通过特定于请求的筛选器链运行请求。

过滤器被虚线分隔的原因是，过滤器可以在代理请求发送之前和之后运行逻辑。执行所有“预”过滤器逻辑。然后发出代理请求。

发出代理请求后，将运行“post”过滤器逻辑。

![](https://blog.lichenghao.cn/upload/2022/07/spring_cloud_gateway_diagram.png)



## 入门配置

### 使用网关

pom 增加 spring-cloud-starter-gateway。

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

    <artifactId>cloud-gateway-gateway9527</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
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
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>

</project>
```

注意，引入 gateway，需要去掉 spring-boot-starter-web 依赖

配置文件，注意 Path 中的 P大写！

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_route
          uri: http://localhost:8001
          predicates:
            - Path=/payment/info/**
        - id: payment_route2
          uri: http://localhost:8002
          predicates:
            - Path=/payment/lib/**

eureka:
  client:
    # 注册进中心
    register-with-eureka: true
    # 从 Eureka Server 抓取自己的注册信息。单机无所谓，集权环境下必须为ture。
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:6001/eureka
  instance:
    hostname: cloud-gateway-service
```

主启动类

```java
/**
 * 网关
 */
@SpringBootApplication
@EnableEurekaClient
public class Gateway9527 {
    public static void main(String[] args) {
        SpringApplication.run(Gateway9527.class, args);
    }
}
```



启动后，服务注册成功

访问 http://localhost:8001/payment/info/1  就可以替换为：http://localhost:9527/payment/info/1

### 配置路由的两种方式

#### XML

如上面的入门配置

```yaml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_route
          uri: http://localhost:8001
          predicates:
            - Path=/payment/info/**
        - id: payment_route2
          uri: http://localhost:8002
          predicates:
            - Path=/payment/lib/**
```

#### Bean

通过注入 Bean的形式，注入路由规则

```java
/**
 * 自定义路由
 */
@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator getRoute(RouteLocatorBuilder builder) {
        RouteLocatorBuilder.Builder routes = builder.routes();
        routes.route("payment_route", r -> r.path("/payment/info/**").uri("http://localhost:8001")).build();
        routes.route("payment_route2", r -> r.path("/payment/lb/**").uri("http://localhost:8002")).build();
        return routes.build();
    }
}
```

## 动态路由

上面的基础配置的路由中，我们的地址都是写死的。这样不能负载均衡。以前我们通过 Ribbon 实现，这里通过 gateway的动态路由实现。

修改服务配置文件即可。增加如下所示：

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  # 开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_route
          #  uri: http://localhost:8001
          uri: lb://CLOUD-PAYMENT-SERVICE  # 匹配后提供服务的路由地址
          predicates:
            - Path=/payment/info/**
        - id: payment_route2
          #  uri: http://localhost:8002
          uri: lb://CLOUD-PAYMENT-SERVICE
          predicates:
            - Path=/payment/lib/**
eureka:
  client:
    # 注册进中心
    register-with-eureka: true
    # 从 Eureka Server 抓取自己的注册信息。单机无所谓，集权环境下必须为ture。
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:6001/eureka
  instance:
    hostname: cloud-gateway-service
```



## 断言 Predicate

提供了十多种断言的类型，如官方截图

https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#gateway-request-predicates-factories

![](https://blog.lichenghao.cn/upload/2022/07/28112100.png)



## 过滤器 Filter

### 内置过滤器

路由筛选器允许以某种方式修改传入的HTTP请求或传出的HTTP响应。路由筛选器的作用域为特定路由。Spring Cloud Gateway包括许多内置网关过滤器工厂。

分类：

- 按照生命周期：过滤器分为前置过滤器和后置过滤器；

- 按照种类：指定路由过滤器和全局过滤器；

官网给的好多指定路由过滤器

https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#gatewayfilter-factories

![](https://blog.lichenghao.cn/upload/2022/07/28113716.png)



全局过滤器也有十多个

https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#global-filters

![](https://blog.lichenghao.cn/upload/2022/07/28113945.png)

### 自定义过滤器

通常通过自定义过滤器：全局日志记录，统一网关鉴权等



配置一个自定义过滤器

```java
/**
 * 自定义过滤器
 */
@Component
@Slf4j
public class LogGatewayFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("LogGatewayFilter:" + LocalDateTime.now());
        String name = exchange.getRequest().getQueryParams().getFirst("name");
        if (name == null) {
            log.error("用户名为空!");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    /**
     * 越小，优先级越高
     *
     * @return
     */
    @Override
    public int getOrder() {
        return 0;
    }
}
```





## 其他

### 跨域配置

```java
/**
 * 服务跨域问题
 *
 */
@Configuration
public class CorsConfig {

    @Bean
    public CorsWebFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        config.addAllowedMethod("*");
        config.addAllowedOrigin("*");
        config.addAllowedHeader("*");
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource(new PathPatternParser());
        source.registerCorsConfiguration("/**", config);
        return new CorsWebFilter(source);
    }
}
```



















