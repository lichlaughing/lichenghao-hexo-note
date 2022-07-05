---
title: Netty 心跳机制
categories: netty
tags: 
  - netty
abbrlink: b6a23e89
cover: https://blog.lichenghao.cn/upload/2022/07/e444d1c7ly1gndk70ui44j20gg09oq45.jpg
date: 2022-03-01 12:00:00
---



通过 IdleStateHandler 处理器来实现心跳的检测，然后交给管道的下一个处理器，在userEventTriggered方法中处理

```java
/**
 * 服务端
 */
public class ChatServer {

    private int port;

    public ChatServer() {
    }

    public ChatServer(int port) {
        this.port = port;
    }

    private void start() {

        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup, workGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .handler(new LoggingHandler(LogLevel.INFO))
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            pipeline.addLast("stringEncoder", new StringEncoder());
                            pipeline.addLast("stringDecoder", new StringDecoder());
                            /**
                             * 心跳检测处理器,参数依次表示，多久没有读操作，多久没有写操作，多久没有读写操作
                             * 结果会传递给管道的下个处理器的userEventTriggered方法中处理
                             */
                            pipeline.addLast(new IdleStateHandler(2, 4, 8, TimeUnit.SECONDS));
                            pipeline.addLast("chatServerHandler", new ChatServerHandler());
                        }
                    });
            ChannelFuture channelFuture = bootstrap.bind(port).sync();
            channelFuture.addListener(future -> {
                if (future.isSuccess()) {
                    System.out.println("聊天室启动成功！");
                }
            });
            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) {
        new ChatServer(6668).start();
    }
}

/**
 * 服务端处理器
 */
public class ChatServerHandler extends ChannelInboundHandlerAdapter {

    private static ChannelGroup channels =
            new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        SocketAddress socketAddress = ctx.channel().remoteAddress();
        channels.writeAndFlush("[客户端：" + socketAddress + "] 上线！");
        channels.add(ctx.channel());
    }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        channels.remove(ctx.channel());
        channels.writeAndFlush("[客户端：" + ctx.channel().remoteAddress() + "] 下线！");
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String getMsg = "[客户端：" + ctx.channel().remoteAddress() + "] 说：" + msg.toString();
        System.out.println(getMsg);
        channels.stream().forEach(channel -> {
            if (channel != ctx.channel()) {
                channel.writeAndFlush(getMsg);
            }
        });
    }

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof IdleStateEvent) {
            IdleStateEvent stateEvent = (IdleStateEvent) evt;
            switch (stateEvent.state()) {
                case READER_IDLE:
                    System.out.println("读空闲！");
                    break;
                case WRITER_IDLE:
                    System.out.println("写空闲");
                    break;
                case ALL_IDLE:
                    System.out.println("读写空闲！");
                    break;
                default:
                    System.out.println("其他空闲！");
            }
        }
    }
}

```





