---
title: JAVA 消息服务
categories: java
tags:
  - jms
  - java
date: 2022-01-01 00:11:00
abbrlink: 636e69ca
---
Java Message Service (JMS)

> JMS介绍文档：[Getting Started with Java Message Service (JMS)](https://www.oracle.com/technical-resources/articles/java/intro-java-message-service.html)

Java消息服务 (JMS) API是一个消息传递标准，它允许基于Java平台的应用程序组件创建、发送、接收和读取消息。它支持松散耦合、可靠和异步的分布式通信。就像各种的DB的链接驱动，各个消息中间件的厂家也对JMS有对应的实现。所以JMS不是一种通信协议，而是一种编程的规范。

比如把一个客户端对消息的生产和消费展示如下：

![](https://blog.lichenghao.cn/upload/2022/07/1577107.png)



市面上现在有很多消息中间件，那么都有哪些消息的协议呢？

消息协议则是指用于实现消息队列功能时所涉及的协议。按照是否向行业开放消息规范文档，可以将消息协议分为**开放协议**和**私有协议**。

常见的开放协议有 **AMQP**、**MQTT**、 **STOMP**、**XMPP**等。有些特殊框架（如Redis、Kafka、ZeroMQ）根据自身需要未严格遵循MQ规范，而是基于TCP/IP自行封装了一套协议，通过网络Socket接口进行传输，实现了MQ的功能。

| 开放协议 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| AMQP     | AMQP 即 Advanced Message Queuing Protocol，一个提供统一消息服务的应用层标准高级消息队列协议,是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同开发语言等条件的限制。**优点：可靠、通用。** |
| MQTT     | MQTT(Message Queuing Telemetry Transport，消息队列遥测传输)是IBM 开发的一个即时通讯协议，有可能成为物联网的重要组成部分。该协议支持所有平台，几乎可以把所有联网物品和外部连接起来，被用来当做传感器和致动器(比如通过Twitter 让房屋联网)的通信协议。**优点：格式简洁、占用带宽小、移动端通信、PUSH、嵌入式系统。** |
| STOMP    | STOMP(Streaming Text Orientated Message Protocol)是流文本定向消息协议，是一种为MOM(Message Oriented Middleware，面向消息的中间件)设计的简单文本协议。STOMP 提供一个可互操作的连接格式，允许客户端与任意STOMP 消息代理(Broker)进行交互。**优点：命令模式(非topic\queue 模式)。** |
| XMPP     | XMPP(可扩展消息处理现场协议，Extensible Messaging and Presence Protocol)是基于可扩展标记语言(XML)的协议，多用于即时消息(IM)以及在线现场探测。适用于服务器之间的准即时操作。核心是基于XML 流传输，这个协议可能最终允许因特网用户向因特网上的其他任何人发送即时消息，即使其操作系统和浏览器不同。**优点：通用公开、兼容性强、可扩展、安全性高，但XML 编码格式占用带宽大。** |

