---
title: TCP的粘包和拆包
categories: netty
tags: 
  - netty
  - 粘包
abbrlink: 13caedd3
cover: https://blog.lichenghao.cn/upload/2022/07/e444d1c7ly1gndk70ui44j20gg09oq45.jpg
date: 2022-03-01 12:00:00
---



## 粘包拆包原理

TCP是面向连接，面向流的，提供高可用的服务。收发两端，客户端和服务端都要有一一成对的socket,因此发送端为了将多个发送给接收端包更有效的发送出去，使用了优化算法（Nagle算法），将多次间隔较小且数量较小的数据，合并成一个较大的数据，进行封包，然后在发送出去。这样做提高了效率但是接收端就很短分辨出完整的数据包了。这样面向流的通信消息是没有边界的。所以拆包和粘包的问题其实就是解决消息的边界的问题。

粘包拆包示意图：客户端向服务端发送Msg1和Msg2两个数据包

![](https://blog.lichenghao.cn/upload/2022/07/FfTKQ1.png)

1. 服务端分别读取到两个独立的数据包Msg1和Msg2
2. 服务端一次接收了两个数据包Msg1和Msg2粘合在一起，成为TCP粘包
3. 服务端两次读取了数据，第一次读取了Msg1数据和包和部分Msg2数据包Msg2-1,第二次读取了Msg2剩下的部分Msg2-2
4. 服务端两次读取了数据，第一次读取了Msg1部分数据包Msg1-1,第二次读取了Msg1剩下的部分Msg1-2和Msg2数据包
5. 3和4称为TCP拆包

## 演示代码

代码地址：[TCP粘包拆包演示代码地址](https://github.com/lichlaughing/lichenghao-study/tree/master/nettystudytest/netty/src/main/java/tcppackage/demo)

服务端：

```java
/*
演示TCP粘包拆包服务端
 */
public class Server {
    private static final int PORT = 6666;

    public static void main(String[] args) throws InterruptedException {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap bootstrap = new ServerBootstrap()
                    .group(bossGroup, workGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            pipeline.addLast(new ServerHandler());
                        }
                    });
            ChannelFuture channelFuture = bootstrap.bind(PORT).sync();
            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }
    }
}


public class ServerHandler extends SimpleChannelInboundHandler<ByteBuf> {
    private int sum;

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf byteBuf) throws Exception {
        // 读取客户端的消息
        byte[] bytes = new byte[byteBuf.readableBytes()];
        byteBuf.readBytes(bytes);
        String msg = new String(bytes, CharsetUtil.UTF_8);
        System.out.println("接收消息：" + msg);
        System.out.println("接收的消息数据的次数：" + (++this.sum));

        // 回应客户端消息
        ByteBuf response =
                Unpooled.copiedBuffer(UUID.randomUUID().toString() + " ", CharsetUtil.UTF_8);
        ctx.writeAndFlush(response);
    }
}

```

客户端：

```java
/*
演示TCP粘包拆包客户端
 */
public class Client {
    private static final String HOST = "127.0.0.1";
    private static final int PORT = 6666;

    public static void main(String[] args) throws InterruptedException {
        NioEventLoopGroup eventExecutors = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap()
                    .group(eventExecutors)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            pipeline.addLast(new ClientHandler());
                        }
                    });
            ChannelFuture channelFuture = bootstrap.connect(HOST, PORT).sync();
            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            eventExecutors.shutdownGracefully();
        }
    }
}


public class ClientHandler extends SimpleChannelInboundHandler<ByteBuf> {

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        // 连续发送5条消息
        for (int i = 0; i < 5; i++) {
            ByteBuf byteBuf = Unpooled.copiedBuffer("hello world ", CharsetUtil.UTF_8);
            ctx.writeAndFlush(byteBuf);
        }
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf byteBuf) throws Exception {
        // 读取服务端返回的消息
        byte[] bytes = new byte[byteBuf.readableBytes()];
        byteBuf.readBytes(bytes);
        String msg = new String(bytes, CharsetUtil.UTF_8);
        System.out.println("接收消息：" + msg);
    }
}

```

## 解决方法

使用自定义的协议和编码器解决TCP粘包拆包问题，其根本就是让接收端知道一次要读取多长的数据。

演示代码地址：[解决TCP粘包拆包问题演示代码](https://github.com/lichlaughing/lichenghao-study/tree/master/nettystudytest/netty/src/main/java/tcppackage/protocol)

自定义协议以及编解码器

```java

/**
 * 消息协议
 */
public class MessageProtocol {
    private int len;
    private byte[] content;

    public int getLen() {
        return len;
    }

    public void setLen(int len) {
        this.len = len;
    }

    public byte[] getContent() {
        return content;
    }

    public void setContent(byte[] content) {
        this.content = content;
    }
}


/**
 * 编码器
 */
public class MessageEncoder extends MessageToByteEncoder<MessageProtocol> {
    @Override
    protected void encode(ChannelHandlerContext ctx, MessageProtocol msg, ByteBuf byteBuf) throws Exception {
        System.out.println("编码器执行~");
        byteBuf.writeInt(msg.getLen());
        byteBuf.writeBytes(msg.getContent());
    }
}


/**
 * 解码器
 */
public class MessageDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(ChannelHandlerContext cgx, ByteBuf byteBuf, List<Object> list) throws Exception {
        System.out.println("解码器执行~");
        int i = byteBuf.readInt();
        byte[] bytes = new byte[i];
        byteBuf.readBytes(bytes);
        // 解析成自定义协议
        MessageProtocol messageProtocol = new MessageProtocol();
        messageProtocol.setLen(i);
        messageProtocol.setContent(bytes);
        // 交给下个处理器
        list.add(messageProtocol);
    }
}

```



服务端

```java
/*
演示TCP粘包拆包服务端
 */
public class Server {
    private static final int PORT = 6666;

    public static void main(String[] args) throws InterruptedException {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap bootstrap = new ServerBootstrap()
                    .group(bossGroup, workGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            // 自定义协议解码器
                            pipeline.addLast(new MessageDecoder());
                            // 自定义编码器
                            pipeline.addLast(new MessageEncoder());
                            // 自定义处理器
                            pipeline.addLast(new ServerHandler());
                        }
                    });
            ChannelFuture channelFuture = bootstrap.bind(PORT).sync();
            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }
    }
}


public class ServerHandler extends SimpleChannelInboundHandler<MessageProtocol> {
    private int sum;

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MessageProtocol messageProtocol) throws Exception {
        // 读取客户端的消息
        String msg = new String(messageProtocol.getContent(), CharsetUtil.UTF_8);
        System.out.println("接收消息：" + msg);
        System.out.println("接收的消息数据的次数：" + (++this.sum));

        // 回应客户端消息
        MessageProtocol response = new MessageProtocol();
        String responseMsg = UUID.randomUUID().toString() + " ";
        response.setContent(responseMsg.getBytes(CharsetUtil.UTF_8));
        response.setLen(responseMsg.getBytes(CharsetUtil.UTF_8).length);
        ctx.writeAndFlush(response);
    }
}


```

客户端

```java

/*
演示TCP粘包拆包客户端
 */
public class Client {
    private static final String HOST = "127.0.0.1";
    private static final int PORT = 6666;

    public static void main(String[] args) throws InterruptedException {
        NioEventLoopGroup eventExecutors = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap()
                    .group(eventExecutors)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            // 自定义协议编码器
                            pipeline.addLast(new MessageEncoder());
                            // 自定义协议解码器
                            pipeline.addLast(new MessageDecoder());
                            // 自定义处理器
                            pipeline.addLast(new ClientHandler());
                        }
                    });
            ChannelFuture channelFuture = bootstrap.connect(HOST, PORT).sync();
            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            eventExecutors.shutdownGracefully();
        }
    }
}

public class ClientHandler extends SimpleChannelInboundHandler<MessageProtocol> {

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        // 连续发送5条消息
        for (int i = 0; i < 5; i++) {
            String msg = "老子吃火锅~ " + i;
            MessageProtocol messageProtocol = new MessageProtocol();
            messageProtocol.setLen(msg.getBytes(CharsetUtil.UTF_8).length);
            messageProtocol.setContent(msg.getBytes(CharsetUtil.UTF_8));
            ctx.writeAndFlush(messageProtocol);
        }
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MessageProtocol messageProtocol) throws Exception {
        // 读取服务端返回的消息
        byte[] content = messageProtocol.getContent();
        String msg = new String(content, CharsetUtil.UTF_8);
        System.out.println("接收消息：" + msg);
    }
}

```

