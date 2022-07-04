---
title: Netty 示例JAVA 程序
categories: netty
tags:
  - netty
abbrlink: c9031c38
cover: https://blog.lichenghao.cn/upload/2022/07/e444d1c7ly1gndk70ui44j20gg09oq45.jpg
---



## Netty WeSocket

代码地址：[——>Netty WebSocket](https://github.com/lichlaughing/lichenghao-study/tree/master/nettystudytest/netty/src/main/java/websocket)

Netty 通过WebSocket编程实现客户端和服务端的长连接

服务端

```java

/**
 * 服务端
 */
public class SocketServer {

    private int port;

    public SocketServer() {
    }

    public SocketServer(int port) {
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
                            // http编码解码器
                            pipeline.addLast(new HttpServerCodec());
                            pipeline.addLast(new ChunkedWriteHandler());
                            // 聚合多段的数据
                            pipeline.addLast(new HttpObjectAggregator(5000));
                            // 将http请求协议升级成ws协议 ws://localhost:ip/socket
                            pipeline.addLast(new WebSocketServerProtocolHandler("/socket"));
                            // 自定义处理器
                            pipeline.addLast(new SocketServerHandler());

                        }
                    });
            ChannelFuture channelFuture = bootstrap.bind(port).sync();
            channelFuture.addListener(future -> {
                if (future.isSuccess()) {
                    System.out.println("服务启动成功！");
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
        new SocketServer(7777).start();
    }
}


/**
 * 服务端处理器
 */
public class SocketServerHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {

    // 读取并回复消息
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame text) throws Exception {
        ctx.channel().writeAndFlush(
                new TextWebSocketFrame(LocalDateTime.now() + "-接收客户端消息：" + text.text()));
    }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        // 唯一ID
        System.out.println("客户端：" + ctx.channel().id().asLongText() + "-加入！");
    }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        System.out.println("客户端：" + ctx.channel().id().asLongText() + "-退出！");
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}

```

客户端：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form onsubmit="return false">
    <textarea name="message" style="height: 300px;width: 300px"></textarea>
    <input type="button" value="send" onclick="send(this.form.message.value)">
    <textarea id="responseText" style="height: 300px;width: 300px"></textarea>
    <input type="button" value="clear" onclick="document.getElementById('responseText').value=''">
</form>
<script>
    var socket;
    if (window.WebSocket) {
        socket = new WebSocket("ws://localhost:7777/socket");
        // 接收服务端消息
        socket.onmessage = function (ev) {
            var res = document.getElementById("responseText");
            res.value = res.value + "\n" + ev.data;
        }
        // 开启连接
        socket.onopen = function (ev) {
            var res = document.getElementById("responseText");
            res.value = "已连接！";
        }
        socket.onclose = function (ev) {
            var res = document.getElementById("responseText");
            res.value = res.value + "\n" + "已关闭连接！";
        }
    }

    function send(msg) {
        if (!window.socket) {
            return;
        } else {
            if (socket.readyState == WebSocket.OPEN) {
                socket.send(msg);
            } else {
                var res = document.getElementById("responseText");
                res.value = res.value + "\n" + "连接未开启！";
            }
        }
    }
</script>
</body>
</html>
```



## Http 服务

代码地址：[——>Http服务](https://github.com/lichlaughing/lichenghao-study/tree/master/nettystudytest/netty/src/main/java/httpServer)

浏览器发出请求，服务器给出回应

```java
public class Server {
    public static void main(String[] args) {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup, workGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ServerChannelInitializer());
            ChannelFuture channelFuture = bootstrap.bind(7777).sync();
            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }
    }
}


public class ServerChannelInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {
        // 加入netty提供的处理HTTP的编解码器
        socketChannel.pipeline().addLast("httpServerCodec", new HttpServerCodec());
        // 加入自己的处理器
        socketChannel.pipeline().addLast("serverHandler", new HttpHandler());
    }
}


public class HttpHandler extends SimpleChannelInboundHandler<HttpObject> {

    /**
     * 读取消息
     *
     * @param ctx        上下文
     * @param httpObject 客户端的消息
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, HttpObject httpObject) throws Exception {
        if (httpObject instanceof HttpRequest) {
            HttpRequest request = (HttpRequest) httpObject;
            URI uri = new URI(request.uri());
            if ("/favicon.ico".equals(uri.getPath())) {
                return;
            }
            // 返回的消息内容
            ByteBuf responseContent =
                    Unpooled.copiedBuffer("This is Server ~", CharsetUtil.UTF_8);
            DefaultFullHttpResponse response =
                    new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, responseContent);
            // 设置头
            response.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/plain");
            response.headers().set(HttpHeaderNames.CONTENT_LENGTH, responseContent.readableBytes());
            // 返回消息
            ctx.writeAndFlush(response);
        }
    }
}

```



## Netty 聊天室

代码地址：[——>Netty 聊天室](https://github.com/lichlaughing/lichenghao-study/tree/master/nettystudytest/netty/src/main/java/nettyChat)

服务端：

```java
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
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            pipeline.addLast("stringEncoder", new StringEncoder());
                            pipeline.addLast("stringDecoder", new StringDecoder());
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
}

```

客户端

```java
public class ChatClient {
    private String host;
    private int port;

    public ChatClient() {
    }

    public ChatClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    private void start() {

        EventLoopGroup workGroup = new NioEventLoopGroup();

        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(workGroup)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            pipeline.addLast("stringEncoder", new StringEncoder());
                            pipeline.addLast("stringDecoder", new StringDecoder());
                            pipeline.addLast("chatClientHandler", new ChatClientHandler());
                        }
                    });
            ChannelFuture channelFuture = bootstrap.connect(host, port).sync();
            Channel channel = channelFuture.channel();
            // 发送消息
            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNext()) {
                String toMsg = scanner.nextLine();
                channel.writeAndFlush(toMsg + "\r");
            }
            // 监听关闭通道
            channel.closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            workGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) {
        new ChatClient("127.0.0.1", 6668).start();
    }
}


public class ChatClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println(msg.toString());
    }
}
```

