# Java IO模型

## I/O模型

-   Java一共支持三种网络I/O的编程模型：BIO、NIO、AIO

### Java BIO

描述

Java BIO：同步阻塞IO（传统的IO）服务的实现模式为一个线程对应一个连接，即当客户端有连接请求时服务器就会启动一个线程进行处理，如果这个连接没有进行数据传输，那么当前线程就会阻塞，造成不必要的线程开销。

图形

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_4lJTOdmBjJ.png)

### Java NIO

描述

Java NIO：同步非阻塞IO，服务器的实现为一个线程可以处理来自多个客户端的请求，客户端发送的请求会注册到多路复用器上，多路复用器轮询这些连接时，如果有请求就会将这些请求交由线程处理

图形

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_EXR8FReVjW.png)

### Java AIO

-   Java AIO（NIO.2）：异步非阻塞IO，AIO引入了异步通信的概念，采用Proactor模式，简化程序的编写，有效的请求才启动线程，它的特点是现有操作系统完成后才通知服务端程序启动线程去，一般适用于连接数较多连接时间较长的应用。

### BIO、NIO、AIO的使用场景分析

-   BIO适用于连接数较小且固定的架构，这种方式对服务器资源要求比较高，并且局限于应用中，JDK1.4以前的唯一选择，但程序简单移动
-   NIO适用于连接数较多且连接时间较小的架构中（轻操作），比如：聊天服务器，弹幕系统，服务器间通讯等
-   AIO适用于连接数较多且连接时间较长的架构中（重操作），比如：相册服务器，充分调用OS参与并发操作
