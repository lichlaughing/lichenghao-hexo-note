---
title: Netty 异步机制
categories: netty
tags:
  - netty
abbrlink: e1356c33
cover: https://blog.lichenghao.cn/upload/2022/07/e444d1c7ly1gndk70ui44j20gg09oq45.jpg
---



```java
ChannelFuture channelFuture = bootstrap.bind(PORT).sync();
```

Future表示异步的结果，通过方法可以验证执行是否执行完毕

Future-Listener机制

```java
// 绑定端口
ChannelFuture channelFuture = bootstrap.bind(PORT).sync();
channelFuture.addListener(future -> {
  if(future.isSuccess()){
    System.out.println("端口监听成功");
  }
});
```

