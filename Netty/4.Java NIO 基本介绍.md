# Java NIO 基本介绍

-   Java NIO：`Java non-blocking io`，是JDK提供的新的API，Java提供的一系列改进的输入输出的新特性，被统称为NIO，是同步非阻塞的
-   NIO相关的类放在`java.nio`包下，并且对原来的`java.io`  进行了改写
-   NIO有三大核心组件：`Channel`（通道）、`Buffer`（缓冲）、`Selector`（选择器）
-   NIO是面向缓冲区、或者是面向块编程的。数据读取到一个它稍后要处理的缓冲区中，需要时数据可以在缓冲区中前后移动，这就增加了处理过程中的灵活性，使用它可以提供非阻塞式的高伸缩性网络
-   Java NIO的非阻塞模式，一个线程从通道中获取数据或者发送请求，它只能获取当前可用的数据，如果当前没有可用的数据，什么都不会获取，不会阻塞线程，线程可以去做其他的事情，非阻塞写也是如此，不必等待数据写完，它也可以去做其他的事情
-   HTTP2.0采用了多路复用技术，使得同一个连接可以发送多个请求

### NIO和BIO的比较

-   BIO是以流的方式处理数据，NIO是以块的方式处理数据。块IO的效率要比流IO高
-   BIO是阻塞的，NIO是非阻塞
-   BIO是基于字节流或者字符流进行操作，NIO是基于channel和buffer进行操作，从通道中写入数据到缓冲，或者从缓冲中读取数据到通道，selector用于监听多个channel的事件。这样就是的一个线程就可以处理多个client的请求

### NIO的三大核心原理示意图

解释

1.  每一个channel都会对应一个buffer
2.  channel和buffer的数据流向都是双向的，只不过buffer的读写都需要flip进行转换
3.  多个channel会注册到selector上
4.  selector会根据不同的事件调度处理不同的channel
5.  一个线程对应一个selector，一个线程可以处理来自多个客户端的请求

图例

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_Di6IvGrq57.png)

### 缓冲区（Buffer）

-   缓冲区本质上是一个可以进行读写的内存块，可以理解为一个容器对象（含有数组），该对象提供了一组方法可以更轻松地使用内存块，缓冲区对象内置了一些机制，能够跟踪记录缓冲区的状态变化情况，channel提供从文件、网络读取数据的渠道，但是读取或写入的数据必须经过Buffer

#### Buffer类及其子类

概要

-   在NIO中，Buffer是一个顶层父类，他是一个抽象类
-   除了Boolean类型为java的所有基本数据类型都有其对应的Buffer，其中最常用的就是ByteBuffer

类结构图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_lGtNKfpvNu.png)

-   Buffer提供了四个基本属性控制缓冲区数据的读写

源码

```java
 // Invariants: mark <= position <= limit <= capacity
    private int mark = -1;
    private int position = 0;
    private int limit;
    private int capacity;
```

解释

-   mark：标记
-   position：位置，下一次对缓冲区读写的索引
-   limit：当前读写缓冲区的极限位置，每次读写不能超过它
-   capacity：缓冲区的最大容量，创建时就指定

<!---->

-   Buffer类的主要方法

```java
public abstract class Buffer { 
   //IDK1.4时，引入的api 
   public final int capacity();//返回此缓冲区的容量 
   public final int position();//返回此缓冲区的位置 
   public final Buffer position (int newPositio);//设置此缓冲区的位置 
   public final int limit();//返回此缓冲区的限制 
   public final Buffer limit (int newLimit);//设置此缓冲区的限制 
   public final Buffer mark();//在此缓冲区的位置设置标记 
   public final Buffer reset();//将此缓冲区的位置重置为以前标记的位置 
   public final Buffer clear();//清除此缓冲区,即将各 1 标记恢复到初始状态，但是数据并没有 
   public final Buffer flip();//反转此缓冲区 
   public final Buffer rewind();//重绕此缓冲区 
   public final int remaining();//返回当前位置与限制之间的元素数 
   public final boolean hasRemaining();//告知在当前位置和限制之间是否有元素 
   public abstract boolean isReadOnly();//告知此缓冲区是否为只读缓冲区 
   //JDK1.6时引入的api 
   public abstract boolean hasArray();//告知此缓冲区是否具有可访问的底层实现数组 
   public abstract Object array();//返回此缓冲区的底层实现数组 
   public abstract int arrayOffset();//返回此缓冲区的底层实现数组中第一个缓冲区元素的偏移量 
   public abstract boolean isDirect();//告知此缓冲区是否为直接缓冲区
}
```

-   ByteBuffer

```java
public abstract class ByteBuffer {
  //缓冲区创建相关api 
  public static ByteBuffer allocateDirect(int capacity)//创建直接缓冲区 
  public static ByteBuffer allocate(int capacity)//设置缓冲区的初始容量 
  public static ByteBuffer wrap(byte[] array)//把1个数组放到缓冲区中使用 
  //构造初始化位置offset和上界length的缓冲区 
  public static ByteBuffer wrap(byte[] array, int offset, int length)
  // 缓存区存取相关API 
  public abstract byte get();//从当前位置position上get，get之后，position会自动+1 
  public abstract byte get(int index);//从绝对位置get 
  public abstract ByteBuffer put(byte b);//从当前位置上添加， put之后, position会自动+1 
  public abstract ByteBuffer put(int index, byte b);//从绝对位置上put
}
```

### 通道（Channel）

描述

-   NIO的通道与流类似，但是有所区别
    -   通道可以同时进行读写，而流只能读或者写
    -   通道可以实现异步读写数据
    -   通道可以对缓冲区进行读写数据
-   Channel是一个接口，定义了统一的规范。常用的Channel有：`FileChannel`（对文件进行读写）、`DatagramChannel`（对UDP进行读写）、`ServerSocketChannel`、`SocketChannel`（对TCP进行读写）
-   Channel可以通过流获取

channel层次结构

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_TGtPvO8bYt.png)

#### FileChannel 类

-   FileChannel 是对本地文件进行IO操作，主要的方法有

```java
public int read(ByteBuffer dst)//从通道读取数据并放到缓冲区中 
public int write(ByteBuffer src)//把缓冲区的数据写到通道中 
public long transferFrom(ReadableByteChannel src, long position, long count)//从目标通道中复制数据到当前通道 
public long transferTo(long position, long count, WritableByteChannel target)//把数据从当前通道复制给目标通道
```

### FileChannel 示例代码

```java
//将缓冲区中的数据写出
public class FileChannelWrite {
    public static void main(String[] args) throws Exception {
        String str = "将一个字符串通过FileChannel写入到文件中";
        //Channel需要借助Buffer
        ByteBuffer buffer = ByteBuffer.wrap(str.getBytes());
        //创建输出流
        FileOutputStream outputStream = new FileOutputStream("file01.txt");
        //获取输出流上的Channel
        FileChannel fileChannel = outputStream.getChannel();
        //将数据从Buffer中写入到Channel中
        fileChannel.write(buffer);
        //关闭资源
        fileChannel.close();
        outputStream.close();
    }
}
```

```java
//将数据读入缓冲区
public class FileChannelRead {
    public static void main(String[] args) throws Exception{
        //获取文件，方便获取文件的长度
        File file = new File("file01.txt");
        //输入流获取文件流
        FileInputStream inputStream = new FileInputStream(file);
        //获取流对应的Channel
        FileChannel channel = inputStream.getChannel();
        //开辟缓冲区
        ByteBuffer buffer = ByteBuffer.allocate((int) file.length());
        //数据读入缓冲区
        channel.read(buffer);
        //展示数据
        System.out.println(new String(buffer.array()));
        //关闭资源
        channel.close();
        inputStream.close();

    }
}

```

```java
//赋值文件代码
public class FileCopy {
    public static void main(String[] args) throws Exception {
        //打开输入输出流
        FileInputStream inputStream = new FileInputStream("file01.txt");
        FileOutputStream outputStream = new FileOutputStream("file02.txt");
        //获取对应的Channel
        FileChannel inputStreamChannel = inputStream.getChannel();
        FileChannel outputStreamChannel = outputStream.getChannel();
        //创建缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        while (inputStreamChannel.read(buffer) != -1) {
            //读写之间要进行翻转，对position和limit重新赋值
            buffer.flip();
            outputStreamChannel.write(buffer);
            //每次读写完毕要将标记初始化，否则出错
            buffer.clear();
        }
        //关闭资源
        inputStreamChannel.close();
        inputStream.close();
        outputStreamChannel.close();
        outputStream.close();

    }
}

public class FileCopy {
    public static void main(String[] args) throws Exception {
        //打开输入输出流
        FileInputStream inputStream = new FileInputStream("file01.txt");
        FileOutputStream outputStream = new FileOutputStream("file02.txt");
        //获取对应的Channel
        FileChannel inputStreamChannel = inputStream.getChannel();
        FileChannel outputStreamChannel = outputStream.getChannel();
        //使用transferFrom进行数据复制，，TransferTo同理
        outputStreamChannel.transferFrom(inputStreamChannel,0,inputStreamChannel.size());
        //关闭资源
        inputStreamChannel.close();
        inputStream.close();
        outputStreamChannel.close();
        outputStream.close();

    }
}


```

### 关闭Buffer和Channel的注意事项和细节

-   `ByteBuffer`支持类型化的put和get，put什么类型的数据，就要按照相同的顺序和类型get这个类型的数据否则会出现`BufferUnderflowException`

```java
public class BufferTest {
    public static void main(String[] args) {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        buffer.putInt(10);
        buffer.putFloat(10.1f);
        buffer.putLong(100L);
        //读写一定要翻转，否则报错
        buffer.flip();
        //按照顺序获取指定类型的元素
        System.out.println(buffer.getInt());
        System.out.println(buffer.getFloat());
        System.out.println(buffer.getLong());
    }

}

```

-   可以将一个普通的Buffer转换为一个只读Buffer

```java
public class BufferTest {
    public static void main(String[] args) {
        IntBuffer buffer = IntBuffer.allocate(10);
        buffer.put(10);
        //读写要转换
        buffer.flip();
        //变为只读Buffer
        IntBuffer intBuffer = buffer.asReadOnlyBuffer();
        System.out.println(intBuffer.get());
    }

}
```

-   NIO还提供了`MappedByteBuffer` 类，这个类可以实现文件在内存中（堆外内存）进行修改，大大提高文件的传输效率。

```java
public class MappedByteBufferTest {
    public static void main(String[] args) throws Exception{
        //随机存取文件
        RandomAccessFile accessFile = new RandomAccessFile("file01.txt", "rw");
        FileChannel channel = accessFile.getChannel();
        //直接在堆外内存修改从0开始到10的文件内容
        MappedByteBuffer buffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, 10);
        buffer.put(1,(byte) 'H');
        buffer.put(2,(byte) 23);
        accessFile.close();
        channel.close();
    }
}
```

-   NIO还支持通过多个Buffer进行数据的读写（Scattering（分散）和Gathering（聚集））
    -   Scattering：采用Buffer数组将数据依次写入
    -   Gathering：将数据从Buffer数组中依次读出

```java
public class FileCopy {
    public static void main(String[] args) throws Exception {
        //打开输入输出流
        FileInputStream inputStream = new FileInputStream("file01.txt");
        FileOutputStream outputStream = new FileOutputStream("file02.txt");
        //获取对应的Channel
        FileChannel inputStreamChannel = inputStream.getChannel();
        FileChannel outputStreamChannel = outputStream.getChannel();
        //创建Buffer数组
        ByteBuffer[] buffers = new ByteBuffer[2];
        buffers[0] = ByteBuffer.allocate(5);
        buffers[1] = ByteBuffer.allocate(3);
        while(true){
            //每一次都要初始化
            Arrays.stream(buffers).forEach(ByteBuffer::clear);
            //将数据一次读取到Buffer数组中
            long read = inputStreamChannel.read(buffers);
            if (read == -1){
                break;
            }
            //读写翻转
            Arrays.stream(buffers).forEach(ByteBuffer::flip);
            //将Buffer数组的数据一次写出去
            outputStreamChannel.write(buffers);
        }

        //关闭资源
        inputStreamChannel.close();
        inputStream.close();
        outputStreamChannel.close();
        outputStream.close();

    }
}

```

### Selector（选择器）

特点

-   多个不同的Channel可以注册到相同的Selector中进行统一管理
-   Selector能够检测多个注册的通道上是否有事件发生，如果有事件发生就获取事件，然后对事件进行相应对的处理，这样就可以使用单线程去管理多个客户端的请求。
-   有事件可会触发线程处理，没有事件什么都不会做，减少系统消耗
-   减少线程切换带来的开销

Selector示意图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_s5bMv_VvX_.png)

#### Selector类的相关方法

```java
public abstract class Selector implements Closeable {
  //得到一个选择器对象
  public static Selector open();
  //监控所有注册的通道，当其中有IO操作可以进行时，将对应的SelectionKey加入到内部集合中并返回，参数用来设置超时时间
  public int select(long timeout);
  //从内部集合中得到所有的SelectionKey,获取的是有事件发生的连接
  public Set<SelectionKey> selectedKeys();
  //获取注册到Selector上的所有的连接
  public abstract Set<SelectionKey> keys();
  //监控所有注册通道，立即返回不会阻塞，
  public abstract int selectNow() throws IOException;
  //监控所有注册通道，会阻塞
  public abstract int select() throws IOException;
  //唤醒阻塞的Selector
  public abstract Selector wakeup();
}
```

#### NIO非阻塞网络编程原理分析

原理图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_LSPiAIkgtU.png)

-   当客户端连接时，会通过ServerSocketChannel获取SocketChannel
-   将所有的Channel注册到Selector上并返回SelectionKey和Selector关联集合
-   Selector进行监听，select方法返回有事件的通道数，并将通道加入内部集合
-   当注册的Channel中有事件发生时，会得到该事件的SelectionKey
-   可以通过SelectionKey获取与其相关的Channel
-   可以通过Channel与客户端进行通信

#### SelectionKey的相关方法

```java
public abstract class SelectionKey {
  public abstract Selector selector();//得到与之关联的Selector 对象
  public abstract SelectableChannel channel();//得到与之关联的通道
  public final Object attachment();//得至到与之关联的共享数据
  public abstract SelectionKey interestOps(int ops);//设置或改变监听事件
  public final boolean isAcceptable();//是否可以accept
  public final boolean isReadable();//是否可以读
  public final boolean isWritable();//是否可以写
}
```

#### ServerSocketChannel的相关方法

```java
public abstract class ServerSocketChannel extends AbstractSelectableChannel implements NetworkChannel{
  public static ServerSocketChannel open()//得到一个 ServerSocketChannel 通道
  public final ServerSocketChannel bind(SocketAddress local)//设置服务器端端口号
  public final SelectableChannel configureBlocking(boolean block)//设置阻塞或非阻塞模式，取值 false表示采用非阻塞模式
  public SocketChannel accept()//接受一个连接，返回代表这个连接的通道对象
  public final SelectionKey register(Selector sel,int ops)//注册一个选择器并设置监听事件
}
```

#### SocketChannel的相关方法

```java
public abstract class SocketChannel extends AbstractSelectableChannel implements ByteChannel, ScatteringByteChannel, GatheringByteChannel,NetworkChannel{
  public static SocketChannel open();//得到一个SocketChannel 通道 
  public final SelectableChannel configureBlocking(boolean block);//设置1塞或非阻塞模式，取值 false 表示采用非阻塞模式
  public boolean connect(SocketAddress remote);//连接服务器
  public boolean finishConnect();//如果上面的方法连接失败，接下来就要通过该方法完成连接操作
  public int write(ByteBuffer src);//往通道里写数据
  public int read(ByteBuffer dst);//从通道里读数据
  public final SelectionKey register(Selector sel, int ops, Object att);//注册一个选择器并设置监听事件，最后一个参数可以设置共享数据
  public final void close();//关闭通道
}
```

#### 代码示例 &#x20;

[NIO网络编程代码示例](https://www.wolai.com/ntZkjB1BkYKMW11cdYnDDc "NIO网络编程代码示例")

### 零拷贝

-   零拷贝是网络编程的关键，很多性能优化都离不开
-   零拷贝从操作系统角度看就是没有CPU参与拷贝的拷贝，在内核缓存之中没有重复数据
-   在Java程序中常用的零拷贝技术有：mmap（内存映射）、sendFile函数
-   mmap适合小数据传输，sendFile适合大数据文件传输
-   在java中可以使用通道中的transferTo函数进行零拷贝

描述

-   传统的IO模型在进行网络传输时，需要将数据在用户态和内核态之间拷贝

1.  硬盘数据通过DMA拷贝到内核缓存
2.  内核缓存数据通过CPU拷贝到用户缓存
3.  用户缓存通过cpu拷贝到套接字缓存
4.  套接字缓存再通过DMA将数据交给网卡

传统的IO模型

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_foIORv__Uy.png)

DMA：直接内存访问（Direct Memory Access）

问题：需要经过多次CPU拷贝，和内核态和用户态的切换，效率不高

描述

-   mmap通过内存映射，将文件映射到内核缓存区，同时用户空间可以共享内核缓存区的数据，在网络传输时减少内核空间和用户空间的数据拷贝。
-   减少一次CPU拷贝

mmap优化

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_WAw8NOJs9b.png)

描述

-   在Linux2.1中提供了sendFile函数
-   数据根本就不需要经过用户态直接从内核缓冲区进入socket buffer中
-   减少了一次用户态和内核态的切换

sendFile函数，kernel:2.1

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_7r-Y1nACDo.png)

描述

-   Linux2.4 改进，避免了从kernel buffer 拷贝到socket buffer中，直接将数据从kernel buffer送到 协议栈上。
-   减少了一次数据拷贝

注：其实这里有一次CPU拷贝（kernel → socket），只是拷贝的信息很少，如：length，offset等等

sendFile函数，kernel 2.4

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_wlYK6YDcPU.png)
