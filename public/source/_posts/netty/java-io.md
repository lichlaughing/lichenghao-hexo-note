---
title: JAVA中常见的 IO 模型 BIO、NIO、AIO
categories: netty
tags:
  - BIO
  - NIO
  - AIO
abbrlink: 67ef9522
---



JAVA中常见的IO模型BIO、NIO、AIO

## BIO

BIO（Blocking I/O）：同步阻塞IO模型,一个请求需要等待响应结果才能继续下一个请求

生活的例子：比如你去银行的ATM机上取钱，如果前面有人你需要等待前面的人取完钱，你才可以取钱

使用的场景：适用于客户端连接数量不多的情况，对服务器资源要求比较高

## NIO

NIO（Non Blocking I/O）: 同步非阻塞IO模型

生活的例子：在银行办理业务，取个号等着就行，这期间你可以不断的询问工作人员你前面还有几个人

使用的场景：它支持面向缓冲的，基于通道的 I/O 操作方法。对于高负载、高并发的（网络）应用应使用 NIO，例如聊天服务器、弹幕 。

### 缓冲区（Buffer）

缓存区本质是一个可以读写数据的内存块，可以看成是一个容器对象，它提供了一组方法去使用这个内存块。（通过源码可以看到Buffer的子类中都维护了一个数组来处理数据）除了boolean以外的其他基本数据类型都有对应的Bufffer子类,例如IntBuffer等

### 通道（Channel）

常用的Channel类：FileChannel、ServerSocketChannel 、SocketChannel。

### 选择器（Selector）

Selector能够检测多个注册的通道上是否有事件发生。多个Channel可以注册在一个Selector上。如果有事件发生，就能获取到事件，然后对这个事件进行处理。



## AIO

AIO（Asynchronous I/O）：异步IO

生活的例子：让你的助手帮你去银行取钱，你可以去干别的，助手取完钱会回来告诉你

使用的场景：适用于连接数较多，且连接时间较长的应用，目前来说 AIO 的应用还不是很广泛

