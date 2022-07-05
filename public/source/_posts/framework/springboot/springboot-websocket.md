---
title: springboot-websocket
categories: springboot
tags:
  - springboot
  - websocket
abbrlink: 470abb14
date: 2022-02-02 12:00:00
---
## 基本实现

WebSocket协议是基于TCP的一种应用层协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。websocket 协议是在 http 协议上的一种补充协议，是 html5 的新特性，是一种持久化的协议。

使用 springboot 2.3.12 + thymeleaf 演示下：

主要的依赖：

```xm
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-websocket</artifactId>
	<version>2.7.0</version>
</dependency>
```

完整 POM

<details>
<summary> 完整 POM & logback.xml（点击展开）</summary>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.12.RELEASE</version>
    </parent>

    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.lichenghao</groupId>
    <artifactId>springboot-websocket</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.3.12.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
            <version>2.7.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
            <version>2.7.0</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.30</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.2.3</version>
        </dependency>
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>30.0-jre</version>
        </dependency>

    </dependencies>


</project>
```

logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <contextName>logback</contextName>
    <property name="log.path" value="logs"/>

    <!--输出到控制台-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>debug</level>
        </filter>
        <encoder>
            <!--<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} : %msg%n</pattern>-->
            <pattern>[%thread] %-5level %logger{36} : %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="console"/>
    </root>
</configuration>
```



</details>

配置文件 application.yaml

```yaml
server:
  port: 8099
spring:
  application:
    name: sb-websocke
  thymeleaf:
    mode: HTML
    encoding: utf-8
    servlet:
      content-type: text/html
    prefix: classpath:/templates/
    suffix: .html
    cache: false
```

配置

```java
/**
 * websocket config
 *
 * @author chenghao.li
 */
@Configuration
public class WebSocketConfig {

    /**
     * 注入ServerEndpointExporter，
     * 这个bean会自动注册使用了@ServerEndpoint注解声明的Websocket endpoint
     */
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

发布服务

```java
/**
 * 发布地址
 *
 * @author chenghao.li
 */
@Component
@ServerEndpoint("/ws")
@Slf4j
public class WsServerEndpoint {

    private Session session;

    /**
     * 存放每个客户端对应的MyWebSocket对象
     */
    private static final Map<String, WsServerEndpoint> WEB_SOCKET_MAP = new ConcurrentHashMap<>();

    /**
     * 连接成功
     *
     * @param session 会话
     */
    @OnOpen
    public void onOpen(Session session) {
        this.session = session;
        WEB_SOCKET_MAP.put(session.getId(), this);
        log.info("连接成功:{}", session.getId());
    }

    /**
     * 连接关闭
     *
     * @param session 会话
     */
    @OnClose
    public void onClose(Session session) {
        WEB_SOCKET_MAP.put(session.getId(), this);
        log.info("连接关闭:{}", session.getId());
    }

    /**
     * 接收到消息
     *
     * @param text 客户端的消息
     */
    @OnMessage
    public String onMsg(String text) {
        return "got it " + text;
    }

    /**
     * 实现服务器主动推送
     */
    public void sendMessage(String message) throws IOException {
        this.session.getBasicRemote().sendText(message);
    }

    public static void sendInfo(String message) throws IOException {
        if (!WEB_SOCKET_MAP.isEmpty()) {
            for (Map.Entry<String, WsServerEndpoint> next : WEB_SOCKET_MAP.entrySet()) {
                next.getValue().sendMessage(message);
            }
        }
    }
}
```

测试 controller

```java
@Controller
public class IndexController {

    @RequestMapping("/index")
    public String index() {
        return "index";
    }

    /**
     * 广播消息
     */
    @ResponseBody
    @GetMapping("/sendMsg/{msg}")
    public String push(@PathVariable("msg") String msg) throws IOException {
        WsServerEndpoint.sendInfo(msg);
        return "ok";
    }
}
```

主启动类

```java
/**
 * 入口
 *
 * @author chenghao.li
 */
@SpringBootApplication
public class AppApplication {
    public static void main(String[] args) {
        SpringApplication.run(AppApplication.class, args);
    }
}
```

测试页面

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>WebSocket测试</title>
</head>

<body>
<div id="main" style="width: 1200px;height:auto;"></div>
Welcome<br/>
<input id="text" type="text"/>
<button onclick="send()">发送消息</button>
<hr/>
<button onclick="closeWebSocket()">关闭WebSocket连接</button>
<hr/>
<div id="message"></div>
</body>
<script type="text/javascript">
    var websocket;
    //判断当前浏览器是否支持WebSocket
    if ('WebSocket' in window) {
        websocket = new WebSocket("ws://localhost:8099/ws");
    } else {
        alert('当前浏览器 Not support websocket')
    }

    //连接发生错误回调方法
    websocket.onerror = function () {
        setMessageInnerHTML("WebSocket连接发生错误");
    };

    //连接成功建立回调方法
    websocket.onopen = function () {
        setMessageInnerHTML("WebSocket连接成功");
    }

    // 接收消息回调方法
    websocket.onmessage = function (event) {
        console.log(event);
        setMessageInnerHTML(event.data);
    }

    //连接关闭回调方法
    websocket.onclose = function () {
        setMessageInnerHTML("WebSocket连接关闭");
    }

    //监听窗口关闭事件
    window.onbeforeunload = function () {
        closeWebSocket();
    }

    //将消息显示在网页上
    function setMessageInnerHTML(innerHTML) {
        document.getElementById('message').innerHTML += '服务器说：' + innerHTML + '<br/>';
    }

    //关闭WebSocket连接
    function closeWebSocket() {
        websocket.close();
    }

    //发送消息
    function send() {
        var message = document.getElementById('text').value;
        websocket.send('{"msg":"' + message + '"}');
    }
</script>
</html>
```

演示：

![](https://blog.lichenghao.cn/upload/2022/07/210256.gif)



## STOMP 

xx



## Netty实现

xx



## Socket.io实现

xx