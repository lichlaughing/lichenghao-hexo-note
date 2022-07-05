---
title: Netty 线程模型
categories: netty
tags: 
  - netty
  - thread
abbrlink: 2fe65e9f
cover: https://blog.lichenghao.cn/upload/2022/07/e444d1c7ly1gndk70ui44j20gg09oq45.jpg
date: 2022-03-01 12:00:00
---



## 概述

原生NIO类库和API相当复杂，需要熟练的掌握各种 channel 和 buffer，以及多线程等。除此之外，开发难度也很大因为你要处理很多情况例如：断连重连、网络拥塞等等各种异常的情况。所以对应的网络编程框架问世了。

官网Netty架构图



![](https://blog.lichenghao.cn/upload/2022/07/e444d1c7ly1gndk70ui44j20gg09oq45.jpg)

Netty 是异步的，基于事件驱动的高性能网络编程框架。

Netty 对 JDK 自带的 NIO 的 API 进行了封装，降低了网络编程的开发难度，并且拥有多种线程模型。



## 常见的线程模型

### BIO 模型

![](https://blog.lichenghao.cn/upload/2022/07/163655.png)

- 每个请求都需要独立的线程去处理，如果并发很大的话，线程会占用很大的系统资源。
- 处理线程创建后，如果没有处理的事件，那么该线程就阻塞。浪费线程资源。

### Reactor 模型

![](https://blog.lichenghao.cn/upload/2022/07/164358.png)

- 基 于I/O 复用模型：多个连接共用一个阻塞对象，程序只需要在一个阻塞对象等待，无需阻塞所有的连接。当有新的数据可以处理的时候，操作系统会通知程序，线程从阻塞状态返回，开始处理数据。
- 基于线程池复用线程资源，不必为每个请求创建线程，将连接完成后的业务处理交给线程池处理，一个线程可以处理多个连接的业务。

Reactor 模式分为如下三种：

- 单 Reactor 单线程
- 单 Reactor 多线程
- 主从 Reactor 多线程  `(Netty 线程模型基于主从 Reactor 多线程模型，并进行了相应的优化改进)`



#### 单 Reactor 单线程

模型示意图：

![](https://blog.lichenghao.cn/upload/2022/07/1184824.png)



- Reactor 通过 select 监控客户端请求，然后通过 dispatch 分发出去。

- 建立连接的请求，Acceptor 通过 accept 处理连接请求，然后创建对应 Handler 去处理业务。

- 如果不是连接请求，则 Reactor 会分发调用连接对应的 handler 进行响应。

  

?> **优点**：模型简单，没有复杂的多线程、进程通信等问题，只需要一个线程就可以完成。

?> **缺点：**服务器通过一个线程多路复用去处理多个客户端的请求，如果客户端较多处理会被阻塞，导致系统无法使用。一个线程无法发挥CPU的性能，客户端较多时候，性能问题很严重。如果线程意外终止导致系统不可用。

?> **场景**：客户端数量有限，业务不复杂



#### 单 Reactor 多线程

模型示意图：

![](https://blog.lichenghao.cn/upload/2022/07/195625.png)

- 通过 select 监控客户端请求事件，通过 dispatch 进行分发。
- 如果是建立连接的请求，由 Acceptor 通过 accept 处理连接请求，同时创建对应的 handler 对象处理完成连接后的事件。
- 如果不是连接请求，则由 Reactor 分发调用连接对应的 handler 来处理。
- handler 只负责响应事件，不做具体的业务处理。通过 read 读取数据后，分发给后面的 worker 线程池中的某个线程去处理。
- Worker 线程池会分配一个独立的线程完成真正的业务，同时把结果返回给 handler。
- handler 收到响应后，通过 send 将结果返回给客户端。



?>优点：相比单线程单线程，具体的业务处理交给线程池中的 worker 线程来处理，充分发挥cpu的性能。

?>缺点：多线程涉及到数据的共享的问题，Reactor 处理所有的事件监听和响应，在高并发下会有性能的瓶颈。



#### 主从 Reactor 多线程

模型示意图：

![](https://blog.lichenghao.cn/upload/2022/07/201840.png)

- Reactor 主线程 MainReactor 对象通过 select 监听连接事件，收到事件后，通过 Acceptor 处理连接事件。
- 当 Acceptor 处理连接事件后，MainReactor 将连接分配给 SubReactor 。
- SubReactor 将连接加入到连接队列中进行监听，并创建 handler 进行各种事件的处理。
- 当有新事件发生时，SubReactor 就会调用对应的 handler 进行处理。
- handler 只负责响应事件，不做具体的业务处理。通过 read 读取数据后，分发给后面的 worker 线程池中的某个线程去处理。
- Worker 线程池会分配一个独立的线程完成真正的业务，同时把结果返回给 handler。
- handler 收到响应后，通过 send 将结果返回给客户端。



?>针对单 Reactor 多线程中的瓶颈，主从Reactor中，MainReactor 主要负责客户端连接，而 SubReactor 负责读取数据，转发任务等。这样的话一个 MainReactor 可以带着多个 SubReactor，处理性能将会大大提升。

?> 相关的文档：《Scalable IO in Java》  地址：https://www.aliyundrive.com/s/hCuvHftJ5hM 

## Netty 的线程模型

工作流程示意图：

![](https://blog.lichenghao.cn/upload/2022/07/G9Cwow.png)



## Netty 版本 Server - Client

测试代码：

服务端：

```java
/**
 * 服务
 */
public class Server {

    private static final int PORT = 6666;

    public static void main(String[] args) throws InterruptedException {

        EventLoopGroup bossGroup = null;
        NioEventLoopGroup workerGroup=null;
        try {
            // 负责客户端连接的工作组
            bossGroup = new NioEventLoopGroup();
            // 负责处理任务的工作组
            workerGroup = new NioEventLoopGroup();
            // 服务端配置
            ServerBootstrap bootstrap = new ServerBootstrap();

            bootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childOption(ChannelOption.SO_KEEPALIVE,true)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel){
                            socketChannel.pipeline().addLast(new ServerHandler());
                        }
                    });

            // 绑定端口
            ChannelFuture channelFuture = bootstrap.bind(PORT).sync();
            // 监听通道关闭
            channelFuture.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}


/**
 * 服务端处理器
 */
public class ServerHandler extends ChannelInboundHandlerAdapter {


    /**
     * 读取消息
     * @param ctx 上下文对象
     * @param msg   客户端发送的消息
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf byteBuf = (ByteBuf) msg;
        System.out.println("客户端:"+ctx.channel().remoteAddress()+"-说："+byteBuf.toString(CharsetUtil.UTF_8));
    }

    /**
     * 数据读取完毕，可以回复消息
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ByteBuf replyMsg =
                Unpooled.copiedBuffer("你好,客户端:" + ctx.channel().remoteAddress(), CharsetUtil.UTF_8);
        ctx.writeAndFlush(replyMsg);
    }

    /**
     * 发生异常
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}

```

客户端：

```java

public class Client {

    private static final String HOST = "127.0.0.1";
    private static final int PORT = 6666;

    public static void main(String[] args) throws InterruptedException {

        EventLoopGroup eventExecutors=null;
        try {
            eventExecutors = new NioEventLoopGroup();
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(eventExecutors)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new ClientHandler());
                        }
                    });
            ChannelFuture channelFuture = bootstrap.connect(HOST, PORT).sync();
            channelFuture.channel().closeFuture().sync();
        } finally {
            eventExecutors.shutdownGracefully();
        }
    }
}


/**
 * 客户端处理器
 */
public class ClientHandler extends ChannelInboundHandlerAdapter {

    // 客户端准备完毕
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.copiedBuffer("你好服务器", CharsetUtil.UTF_8));
    }

    // 读取数据
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf info = (ByteBuf) msg;
        System.out.println("服务器说："+info.toString(CharsetUtil.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}

```

## 三种任务场景

### 用户自定义任务

当服务器端读取比较耗时的事件的时候，为了避免阻塞可以使用自定义任务，将任务直接提交到Channle的eventLoop中的taskQueue中

```java
/**
     * 读取消息
     *
     * @param ctx 上下文对象
     * @param msg 客户端发送的消息
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ctx.channel().eventLoop().execute(() -> {
            try {
                Thread.sleep(5 * 1000);
                ByteBuf byteBuf = (ByteBuf) msg;
                System.out.println("客户端:"
                        + ctx.channel().remoteAddress() + "-说：" + byteBuf.toString(CharsetUtil.UTF_8));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        System.out.println("继续任务...");
    }
```



### 定时任务

```java
 /**
     * 读取消息
     *
     * @param ctx 上下文对象
     * @param msg 客户端发送的消息
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 自定义定时任务，提交到scheduledTaskQueue，5s之后执行
        ctx.channel().eventLoop().schedule(() -> {
            ByteBuf byteBuf = (ByteBuf) msg;
            System.out.println("客户端:"
                    + ctx.channel().remoteAddress() + "-说：" + byteBuf.toString(CharsetUtil.UTF_8));
        }, 5, TimeUnit.SECONDS);

        System.out.println("继续任务...");
    }
```

### 非当前Reactor线程调用Channel的方法

可以用来处理不同的Channel之间的传递消息

演示代码：广播客户端的消息,修改server端代码，记录下所有的客户端对应的Channel，在处理器中广播客户端的消息

```java


/**
 * 服务
 */
public class Server {

    private static final int PORT = 6666;

    public static Set<SocketChannel> channelList = new HashSet<>();

    public static void main(String[] args) throws InterruptedException {

        EventLoopGroup bossGroup = null;
        NioEventLoopGroup workerGroup = null;
        try {
            // 负责客户端连接的工作组
            bossGroup = new NioEventLoopGroup();
            // 负责处理任务的工作组
            workerGroup = new NioEventLoopGroup();
            // 服务端配置
            ServerBootstrap bootstrap = new ServerBootstrap();

            bootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) {
                            channelList.add(socketChannel);
                            socketChannel.pipeline().addLast(new ServerHandler());
                            System.out.println(channelList);
                        }
                    });

            // 绑定端口
            ChannelFuture channelFuture = bootstrap.bind(PORT).sync();
            // 监听通道关闭
            channelFuture.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}


/**
 * 服务端处理器
 */
public class ServerHandler extends ChannelInboundHandlerAdapter {


    /**
     * 读取消息
     *
     * @param ctx 上下文对象
     * @param msg 客户端发送的消息
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 自定义普通任务提交到taskQueue中
        ctx.channel().eventLoop().execute(() -> {
            try {
                Thread.sleep(5 * 1000);
                ByteBuf byteBuf = (ByteBuf) msg;
                System.out.println("客户端:"
                        + ctx.channel().remoteAddress() + "-说：" + byteBuf.toString(CharsetUtil.UTF_8));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        // 自定义定时任务，提交到scheduledTaskQueue，5s之后执行
        ctx.channel().eventLoop().schedule(() -> {
            ByteBuf byteBuf = (ByteBuf) msg;
            System.out.println("客户端:"
                    + ctx.channel().remoteAddress() + "-说：" + byteBuf.toString(CharsetUtil.UTF_8));
        }, 5, TimeUnit.SECONDS);

        // 调用其他的channel去发送消息
        ByteBuf byteBuf = (ByteBuf) msg;
        ByteBuf broadcastMsg =
                Unpooled.copiedBuffer(ctx.channel().remoteAddress() + "-说：" + byteBuf.toString(CharsetUtil.UTF_8), CharsetUtil.UTF_8);
        Set<SocketChannel> channelList = Server.channelList;
        channelList.stream().forEach(socketChannel -> {
            if (socketChannel != null && socketChannel != ctx.channel()) {
                socketChannel.writeAndFlush(Unpooled.copiedBuffer(broadcastMsg));
            }
        });
    }

    /**
     * 数据读取完毕，可以回复消息
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ByteBuf replyMsg =
                Unpooled.copiedBuffer("你好,客户端:" + ctx.channel().remoteAddress(), CharsetUtil.UTF_8);
        ctx.writeAndFlush(replyMsg);


    }

    /**
     * 发生异常
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}

```

































