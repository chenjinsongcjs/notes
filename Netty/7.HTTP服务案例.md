# HTTP服务案例

### 启动器

```java
public class HttpServer {
    public static void main(String[] args) throws Exception{
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            //启动netty
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup,workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .handler(new LoggingHandler(LogLevel.INFO))
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            //添加编解码器
                            pipeline.addLast("httpCodec",new HttpServerCodec());
                            //添加业务处理器
                            pipeline.addLast(new HttpServerHandler());
                        }
                    });
            ChannelFuture future = serverBootstrap.bind(8888).sync();
            //监听通道关闭事件
            future.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}

```

### 处理器

```java
public class HttpServerHandler extends SimpleChannelInboundHandler<HttpObject> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, HttpObject httpObject) throws Exception {
        if (httpObject instanceof HttpRequest) {
            System.out.println("消息类型：" + httpObject.getClass());
            System.out.println("客户端地址："+ ctx.channel().remoteAddress());
            ByteBuf buf = Unpooled.copiedBuffer("hello this is server", CharsetUtil.UTF_8);
            //构造http请求
            DefaultFullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, buf);
            response.headers().set(HttpHeaderNames.CONTENT_TYPE,"text/plain");
            response.headers().set(HttpHeaderNames.CONTENT_LENGTH,buf.readableBytes());
            response.headers().set(HttpHeaderNames.ACCEPT_CHARSET,"utf-8");
            ctx.writeAndFlush(response);
        }
    }
}

```
