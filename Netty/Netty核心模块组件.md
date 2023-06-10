# Netty核心模块组件

## Bootstrap、ServerBootstrap

-   这两个组件主要是启动配置netty程序，串联各个组件。Bootstrap是客户端的启动引导类、ServerBootstrap是服务器端的启动引导类
-   常用的方法有

```java
public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup)，该方法用于服务器端，用来设置两个EventLoop
public B group(EventLoopGroup group)， 该方法用于客户端，用来设置一个 EventLoop
public B channel(Class<？extends C> channelClass)，该方法用来设置一个服务器端的通道实现
public <T> B option(ChannelOption<T> option, T value)，用来给 ServerChannel 添加配置
public <T> ServerBootstrap childOption(ChannelOption<T> childOption,T value)，用来给接收到的通道添加配置
public B handler(ChannelHandler handler),用来给bossgroup条件处理器
public ServerBootstrap childHandler(ChannelHandler childHandler)，该方法用来设置业务处理类（自定义的handler）
public ChannelFuture bind(int inetPort)，该方法用于服务器端，用来设置占用的端口号
public ChannelFuture connect(String inetHost,int inetPort)，该方法用于客户端，用来连接服务器端
```

## Future、ChannelFuture

-   Netty 中所有的 IO 操作都是异步的，不能立刻得知消息是否被正确处理。但是可过一会等它执行完成或者直接注册一个监听，具体的实现就是通过 Future 和 ChannelFutures，他们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件
-   常用方法

```java
Channel channel();//获取当前正在进行IO操作的通道
ChannelFuture addListener(GenericFutureListener<? extends Future<? super Void>> var1);//添加监听器
ChannelFuture sync()//等待异步操作完成

```

## Channel

-   &#x20;Netty网络通信的组件，能够用于执行网络I/O操作。
-   &#x20;通过Channel可获得当前网络连接的通道的**状态**
-   &#x20;通过Channel 可获得 网络连接的**配置参数**（例如接收缓冲区大小）
-   &#x20;Channel 提供异步的网络 I/O 操作(如建立连接，读写，绑定端口)，异步调用意味着任何 I/O 调用都将立即返回，并且不保证在调用结束时所请求的 I/O 操作已完成
-   &#x20;调用立即返回一个ChannelFuture实例，通过注册监听器到ChannelFuture上，可以I/O操作成功、失败或取消时回调通知调用方
-   &#x20;支持关联 I/O 操作与对应的处理程序
-   &#x20;不同**协议**、不同的**阻塞类型**的连接都有不同的 Channel 类型与之对应
-   常用的 Channel 类型:

```java
NioSocketChannel，异步的客户端 TCP Socket 连接。
NioServerSocketChannel，异步的服务器端TCPSocket连接。
NioDatagramChannel，异步的UDP连接。
NioSctpChannel，异步的客户端 Sctp 连接。
NioSctpServerChannel，异步的Sctp服务器端连接，这些通道涵盖了UDP和TCP网络IO以及文件IO。
```

## Selector

-   Netty基于Selector对象实现I/O多路复用，通过Selectorr 一个线程可以监听多个连接的 Channel 事件。
-   当向一个Selector中注册Channel后，Selector内部的机制就可以自动不断地查询(Select)这些注册的Channel 是否有已就绪的 I/O 事件（例如可读，可写，网络连接完成等），这样程序就可以很简单地使用一个线程高效地管理多个 Channel

## ChannelHandler及其实现类

-   ChannelHandler 是一个接口，处理 I/O 事件或拦截 I/O 操作，并将其转发到其 ChannelPipeline(业务处理链)中的下一个处理程序。
-   &#x20;ChannelHandler 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，可以继承它的子类
-   ChannelHandler及其实现类一览

描述

-   Inbound：处理数据入站事件
-   OutBound：处理数据出站事件
-   ChannelDuplexHandler：处理出站或者入站事件
-   我们一般是继承或者实现这些类或者接口，重写他们的方法来进行业务处理

类图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image__ZBnwPU0TJ.png)

```java
public interface ChannelInboundHandler extends ChannelHandler {
    //通道注册事件
    void channelRegistered(ChannelHandlerContext var1) throws Exception;
    void channelUnregistered(ChannelHandlerContext var1) throws Exception;
    //通道一建立就会触发
    void channelActive(ChannelHandlerContext var1) throws Exception;
    void channelInactive(ChannelHandlerContext var1) throws Exception;
    //读取通道事件
    void channelRead(ChannelHandlerContext var1, Object var2) throws Exception;
    //读取完成事件
    void channelReadComplete(ChannelHandlerContext var1) throws Exception;

    void userEventTriggered(ChannelHandlerContext var1, Object var2) throws Exception;
    void channelWritabilityChanged(ChannelHandlerContext var1) throws Exception;
    //处理异常
    void exceptionCaught(ChannelHandlerContext var1, Throwable var2) throws Exception;
}
```

## Pipeline、ChannelPipeline

-   ChannelPipeline是一个Handler的集合，它负责处理和拦截inbound或者outbound的事件和操作，相当于一个贯穿 Netty 的链。(也可以这样理解：ChannelPipeline 是 保存 ChannelHandler 的 List，用于处理或拦截Channel 的入站事件和出站操作)
-   &#x20;ChannelPipeline 实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及 Channel中各个的 ChannelHandler 如何相互交互
-   &#x20;在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应，它们的组成关系如下：

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_7g2e_hmbr7.png)

-   常用方法

```java
ChannelPipeline addFirst(ChannelHandler.. handlers)，把一个业务处理类（handler）添加到链中的第一个位置
ChannelPipeline addLast(ChannelHandler... handlers)，把一个业务处理类（handler）添加到链中的最后一个位置
```

## ChannelHandlerContext

-   保存Channel相关的所有上下文信息，同时关联一个 ChannelHandler 对象
-   即ChannelHandlerContext中包含一个具体的事件处理器ChannelHandler，同 时ChannelHandlerContext 中也绑定了对应的 pipeline 和 Channel 的信息，方便对ChannelHandler进行调用.
-   常用方法

```java
Channel channel();//获取通道
ChannelHandlerContext flush();//刷新通道
ChannelPipeline pipeline();//获取管道
ChannelFuture writeAndFlush(Object var1);//将数据写入到当前ChannelPipeline的当前ChannelHandler的
//下一个ChannelHandler进行处理

```

## ChannelOption

-   Netty在创建Channel实例后一般都要设置ChannelOption&#x20;
-   常用参数：

```java
ChannelOption.SO_BACKLOG
对应TCP/IP协议 Iisten 函数中的 backlog 参数，用来初始化服务器可连接队列大小。服务端处理客户端连接请求是顺序处理的，所以同一时间只能处理一个客户端连接。多个客户端来的时候，服务端将不能处理的客户端连接请求放在队列中等待处理，backlog 参数指定了队列的大小。

ChannelOption.SO_KEEPALIVE
保持连接
```

## EventLoopGroup及其实现类NioEventLoopGroup

-   &#x20;EventLoopGroup 是一组 EventLoop 的抽象，Netty 为了更好的利用多核 CPU 资源，一般会有多个 EventLoop同时工作，每个 EventLoop 维护着一个 Selector 实例。
-   EventLoopGroup 提供 next 接口，可以从组里面按照一定规则获取其中一个 EventLoop来处理任务。在 Netty服务 器端编程中，我们一般都需要提供两个EventLoopGroup，例如：BossGroup 和workerEventLoopGroup。
-   &#x20;通常一个服务端口即一个ServerSocketChannel对应一个Selector和一个EventLoop线程。BossEventLoop负责接收客户端的连接并将SocketChannel交给WorkerEventLoopGroup来进行IO处理，如下图所示

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_8GHT3HgGLu.png)

-   常用方法

```java
public NioEventLoopGroup()，构造方法
public Future<？> shutdownGracefully()，断开连接，关闭线程
```

## Unpooled类

-   Netty提供了一个专门操作缓冲区的工具类（即Netty的数据容器）

说明

-   0 → readerIndex：表示已读数据
-   readerIndex→writerIndex：表示可读数据
-   writerIndex→capacity：表示可写数据区

图示

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_0ORSSmXN18.png)

-   常用的方法

```java
public static ByteBuf buffer(int initialCapacity)//创建指定大小的ByteBuf
public static ByteBuf copiedBuffer(CharSequence string, Charset charset) //创建指定字符编码的缓冲区

```

-   ByteBuf的常用方法

```java
  public abstract int capacity();
  public abstract boolean isReadOnly();
  public abstract int readerIndex();
  public abstract int writerIndex();
  public abstract byte getByte(int var1);
  public abstract ByteBuf getBytes(int var1, ByteBuf var2);
  public abstract ByteBuf writeByte(int var1);
  public abstract ByteBuf writeBytes(ByteBuf var1);
```

## Netty心跳检测机制

```java
public class NettyServer {
    public static void main(String[] args) throws Exception {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup,workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .handler(new LoggingHandler(LogLevel.INFO))
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            //通道中放入IdleStateHandler
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            //使用IdleStateHandler空闲状态处理器进行心跳检测
                            //使用userEventTrigger进行处理
                            //参数顺序：
                            // readerIdleTimeSeconds，读空闲，触发事件
                            // writerIdleTimeSeconds，写空闲触发事件
                            // allIdleTimeSeconds，读写空闲触发事件
                            pipeline.addLast(new IdleStateHandler(2,5,7));
                            //加入真正的业务事件处理器
                            pipeline.addLast(null);
                        }
                    });
            ChannelFuture future = serverBootstrap.bind(8888).sync();
            future.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}

```

```java
public class NettyHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof IdleStateEvent){
            IdleStateEvent idleStateEvent = (IdleStateEvent) evt;
            //在不同空闲状态进行处理
            switch (idleStateEvent.state()){
                case READER_IDLE:
                    System.out.println("读空闲");
                    break;
                case WRITER_IDLE:
                    System.out.println("写空闲");
                    break;
                case ALL_IDLE:
                    System.out.println("读写空闲");
                    break;
            }
        }
    }
}

```

## Netty群聊系统

[Netty群聊系统案例](https://www.wolai.com/cL3rDPzqRN2y8LyCbfQjri "Netty群聊系统案例")

## Netty通过WebSocket实现客户端与服务器端的长链接

[Netty结合WebSocket实现长链接](https://www.wolai.com/a1NYXsR1FGkJe2gwqFnrCY "Netty结合WebSocket实现长链接")
