---
title: Netty Handler
categories: netty
tags: 
  - netty
abbrlink: 2d8720e5
cover: https://blog.lichenghao.cn/upload/2022/07/e444d1c7ly1gndk70ui44j20gg09oq45.jpg
date: 2022-03-01 12:00:00
---



## 出入站

以客户端程序为例，如果事件的运动方向是从客户端到服务端，我们称这些事件为`出站`的，即客户端发送给服务端的数据会通过pipeline中的一系列ChannelOutboundHandler处理；反之，客户端接收服务端的数据会通过pipeline中的ChannelInboundHandler处理，称这些事件为`入站`的。

Netty发送或者接收一个消息的时候，就会发生一次数据转换。入站的消息被解码，出站的消息被编码。也就是说ChannelOutboundHandler处理的时候需要编码，ChannelInboundHandler处理的时候需要解码。示意图如下所示：

![](https://blog.lichenghao.cn/upload/2022/07/2WROEJ.png)





