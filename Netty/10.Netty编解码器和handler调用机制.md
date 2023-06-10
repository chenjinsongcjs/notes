# Netty编解码器和handler调用机制

## 基本说明

-   netty的组件设计：Netty的基本组件有：Channel、ChannelFuture、ChannelPipeline、ChannelHandler、EventLoop等等。
-   ChannelHandler充当了处理出战入站数据应用程序逻辑的容器。例如：实现ChannelInboundHandler接口（或者ChannelInboundHandlerAdapter），你就可以接收入站数据，这些数据会被业务逻辑处理，当要个客户端发送消息时也可以从ChannelInboundHandler中刷出数据。业务逻辑通常是写在一个或者多个Channel InboundHandler中的。ChannelOutboundHandler也是同样的道理，只是他是处理出战数据的。
-   ChannelPipeline提供了ChannelHandler链的容器。以客户端应用程序为例，如果事件的运动方向是从客户端到服务端的，那么我们称这些事件为出站的，即客户端发送给服务端的数据会通过pipeline中的一系列ChannelOutboundHandler，并被这些Handler处理，反之则称为入站的

## 编解码器

-   Netty发送或者接受一个消息的时候就会发生一次数据转换。入站消息就会解码，从字节码转换为另外一种格式。出战消息就会编码，从一种格式转换为字节码
-   Netty提供一系列实用的编解码器，他们都实现了ChannelInboundHadnler或者ChannelOutboundHandler接口。在这些类中，channelRead 方法已经被重写了。以入站为例，对于每个从入站Channel 读取的消息，这个方法会被调用。随后，它将调用由解码器所提供的 decode()方法进行解码，并将已经解码的字节转发给ChannelPipeline中的下一个ChannelInboundHandler。
-   当提供的类型和指定的类型不匹配的时候，编码器会被跳过执行。

## 解码器：ByteToMessageDecoder

描述

-   由于不知道客户端是否会一次性的发送完整信息，tcp可能会出现粘包和拆包问题，这个类会对入站的数据进行缓存，直到可以被处理为止

类图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image__KaQw6XQ_X.png)

```java
//自定义解码器
public class MyDecoder extends ByteToMessageDecoder {
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> list) throws Exception {
        //当缓冲中的可读数据到达一个完整的int时才可以读取
        if (in.readableBytes() >= 4){
            list.add(in.readInt());
        }
        /**
         * 这个例子，每次入站从ByteBuf中读取4字节，将其解码为一个int，
         * 然后将它添加到下一个List中。当没有更多元素可以被添加到该List中时，
         * 它的内容将会被发送给下一个ChannelInboundHandler。
         * int在被添加到List中时，会被自动装箱为Integer。
         * 在调用readInt()方法前必须验证所输入的ByteBuf是否有足够的数据
         */
    }
}

```

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6_1pXCZgaD-g.png)

## Netty的Handler链调用机制

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_eXmB5NzPof.png)

-   不论是编码器还是解码器，接收的消息类型必须和待处理的类型保持一致，否则编码器或者解码器不会被执行
-   在进行解码的时候要判断缓冲区的数据是否足够，否则可能会接收到与期望不一致的数据

## 解码器：ReplayingDecoder

```java
public abstract class ReplayingDecoder<S> extends ByteToMessageDecoder 
```

-   ReplayingDecoder 扩展了 ByteToMessageDecoder类，使用这个类，我们不必调用`readableBytes()`方法。参数`S`指定了用户状态管理的类型，其中`Void`代表不需要状态管理

```java
public class MyDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf, List<Object> list) throws Exception {
        //不需要判断数据是否则狗，这个类在内部进行了判断
        list.add(byteBuf.readInt());
    }
}

```

-   `ReplayingDecoder` 使用方便，但它也有一些局限性：
    -   并不是所有的`ByteBuf`操作都被支持，如果调用了一个不被支持的方法，将会抛出一个`UnsupportedOperationException`。
    -   `ReplayingDecoder`在某些情况下可能稍慢于 `ByteToMessageDecoder`，例如网络缓慢并且消息格式复杂时，消息会被拆成了多个碎片，速度变慢

## 其他的解码器

-   LineBasedFrameDecoder：这个类在Netty内部也有使用，它使用行尾控制字符（\n或者\r\n）作为分隔符来解.析数据。
-   &#x20;DelimiterBasedFrameDecoder：使用自定义的特殊字符作为消息的分隔符。
-   HttpObjectDecoder：一个HTTP数据的解码器
-   LengthFieldBasedFrameDecoder：通过指定长度来标识整包消息，这样就可以自动的处理黏包和半包消息。

## 其他的编码器

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_bQqOb8ihPA.png)

## Log4j日志整合

-   导入依赖
-   书写配置文件
