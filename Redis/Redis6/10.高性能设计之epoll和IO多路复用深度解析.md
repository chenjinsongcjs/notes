# 高性能设计之epoll和IO多路复用深度解析

-   Redis为什么快?
    -   IO多路复用模型
-   Redis单线程如何处理那么多并发客户端连接，为什么单线程，为什么快

```java
Redis 是跑在单线程中的，所有的操作都是按照顺序线性执行的，但是由于读写操作等待用户输入或输出都是阻塞的，所以 I/O 操作在一般情况下往往不能直接返回，这会导致某一文件的 I/O 阻塞导致整个进程无法对其它客户提供服务，而 I/O 多路复用就是为了解决这个问题而出现
 
所谓 I/O 多路复用机制，就是说通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或写就绪），能够通知程序进行相应的读写操作。这种机制的使用需要 select 、 poll 、 epoll 来配合。多个连接共用一个阻塞对象，应用程序只需要在一个阻塞对象上等待，无需阻塞等待所有连接。当某条连接有新的数据可以处理时，操作系统通知应用程序，线程从阻塞状态返回，开始进行业务处理。
 
Redis 服务采用 Reactor 的方式来实现文件事件处理器（每一个网络连接其实都对应一个文件描述符） 
Redis基于Reactor模式开发了网络事件处理器，这个处理器被称为文件事件处理器。它的组成结构为4部分：
多个套接字、
IO多路复用程序、
文件事件分派器、
事件处理器。
因为文件事件分派器队列的消费是单线程的，所以Redis才叫单线程模型
```

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_s4i27Rt0RJ.png)

-   Redis利用epoll来实现IO多路复用，将连接信息和事件放到队列中，一次放到文件事件分派器，事件分派器将事件分发给事件处理器。

第1列

IO多路复用是什么

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_8lORrie-c3.png)

### 同步、异步、阻塞、非阻塞

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_50rw49FsnR.png)

### Unix网络编程中的五种IO模型

-   Blocking IO - 阻塞IO
-   NoneBlocking IO - 非阻塞IO
-   IO multiplexing - IO多路复用
-   signal driven IO - 信号驱动IO
-   asynchronous IO - 异步IO
