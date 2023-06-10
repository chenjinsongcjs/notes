# Netty案例—TCP服务

#### 服务器端

```java
public class NettyServer {
    public static void main(String[] args) throws Exception {
        //创建BossGroup和WorkerGroup
        //默认含有线程数为cpu核心数的两倍
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            //启动客户端配置
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup)//设置两个线程组
                    .channel(NioServerSocketChannel.class)//设置服务器的通道
                    .option(ChannelOption.SO_BACKLOG, 128)//对服务器通道设置
                    .childOption(ChannelOption.SO_KEEPALIVE, true)//对连接通道设置
                    .childHandler(new ChannelInitializer<SocketChannel>() {//对workerGroup的事件设置通道
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            //在管道中加入事件
                            socketChannel.pipeline().addLast(new NettyServerHandler());
                        }
                    });
            //绑定端口，异步操作
            ChannelFuture future = serverBootstrap.bind(8888).sync();
            System.out.println("服务器启动。。。");
            //对关闭通道进行监听
            future.channel().closeFuture().sync();
        } finally {
            //优雅关闭线程组
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}

```

```java
public class NettyServerHandler extends ChannelInboundHandlerAdapter {
    //通道读取事件
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("收到来自客户端的消息：" + buf.toString(CharsetUtil.UTF_8));
    }

    //读取完成事件
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        //向客户端发送消息
        ctx.channel().writeAndFlush(Unpooled.copiedBuffer("hello 这里是服务器。。。", CharsetUtil.UTF_8));
    }

    //异常处理事件
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
        System.out.println(cause.getCause().getMessage());
    }
}

```

#### 客户端

```java
public class NettyClient {
    public static void main(String[] args) throws Exception {
        NioEventLoopGroup group = new NioEventLoopGroup();

        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new NettyClientHandler());
                        }
                    });
            ChannelFuture future = bootstrap.connect("127.0.0.1", 8888).sync();
            System.out.println("客户端准备就绪。。。");
            future.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully();
        }

    }
}

```

```java
public class NettyClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("from Server："+ buf.toString(CharsetUtil.UTF_8));
    }
    //有通道就会触发
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.channel().writeAndFlush(Unpooled.copiedBuffer("hello ；服务器。。。", CharsetUtil.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}

```
