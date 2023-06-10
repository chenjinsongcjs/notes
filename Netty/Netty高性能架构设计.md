# Netty高性能架构设计

### 线程模型的基本介绍

-   目前存在的线程模型
    -   传统的阻塞IO线程模型
    -   Reactor模式
        -   单Reactor单线程
        -   单Reactor多线程
        -   主从Reactor多线程
-   Netty的线程模型主要基于主从Reactor多线程模型做了一定的改进，其中主从Reactor多线程模型有多个Reactor

### 传统阻塞IO线程模型

描述

-   采用阻塞IO获取输入数据
-   每一个连接都需要独立线程完成数据的输入，业务处理，数据返回
-   当并发量大的时候就会创建大量的线程，占用很大的系统资源
-   连接创建后如果没有数据可读，那么线程就会阻塞，导致线程浪费

图示

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_wQFB5LjC7b.png)

### Reactor模式

#### 针对传统阻塞IO线程模型的不足主要有以下两个方法进行解决：

1.  基于IO复用模型：多个连接共用一个阻塞对象，应用程序只需要在阻塞对象等待，无需阻塞等待所有的连接，当某个连接有数据处理的时候，操作系统通知应用程序，线程从阻塞状态返回，进行业务处理。
2.  基于线程池复用线程资源：不需要为每个连接创建一个单独的线程进行处理，连接完成之后，将任务交由线程池处理，每个线程可以处理多个连接的任务

描述

-   IO复用结合线程池就是Reactor模式的基本思想
-   多个客户端将请求发送给服务器端的处理程序（一个阻塞对象），处理程序收到请求之后，将任务转发给线程池处理。

图示

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_-EyS-aUYp0.png)

#### Reactor模式的核心组成

-   Reactor：在一个**独立的线程**中运行，负责监听和分发事件，将IO事件分发给适当的应用程序进行处理
-   Handlers：处理程序执行IO事件实际要完成的任务，处理程序执行的是非阻塞操作

#### 单Reactor单线程

描述

-   select 是前面 I/O 复用模型介绍的标准网络编程 API,可以实现应用程序通过一个阻塞对象监听多路连接请求
-   Reactor 对象通过 Select 监控客户端请求事件，收到事件后通过 Dispatch 进行分发
-   如果是建立连接请求事件，则由 Acceptor 通过 Accept 处理连接请求，然后创建一个 Handler 对象处理连接完成后的后续业务处理
-   如果不是建立连接事件，则 Reactor 会分发调用连接对应的 Handler 来响应
-   Handler 会完成 Read→业务处理→Send 的完整业务流程

图示

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_0N7mp0VZQ3.png)

优点：模型简单，没有多线程、进程通信、竞争的问题，全部都在一个线程中完成

缺点：性能问题，只有一个线程，无法完全发挥多核 CPU 的性能。Handler 在处理某个连接上的业务时，整个进程无法处理其他连接事件，很容易导致性能瓶颈

缺点：可靠性问题，线程意外终止，或者进入死循环，会导致整个系统通信模块不可用，不能接收和处理外部消息，造成节点故障

#### 单Reactor多线程

描述

-   Reactor对象通过select监控客户端请求事件,收到事件后，通过dispatch进行分发
-   如果建立连接请求，则右Acceptor通过accept处理连接请求,然后创建一个Handler对象处理完成连接后的各种事件
-   如果不是连接请求，则由 reactor 分发调用连接对应的 handler 来处理
-   handler只负责响应事件，不做具体的业务处理，通过read读取数据后分发给worker线程池的某个线程处理业务
-   worker 线程池会分配独立线程完成真正的业务，并将结果返回给handler
-   &#x20;handler收到响应后，通过 send 将结果返回给 client

图示

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_63WOiQ-Gm6.png)

优点：可以充分的利用多核cpu的处理能力

缺点：多线程数据共享和访问比较复杂， reactor 处理所有的事件的监听和响应，在单线程运行， 在高并发场景容易出现性能瓶颈.

#### 主从Reactor多线程

描述

-   Reactor 主线程 MainReactor 对象通过select监听连接事件,收到事件后，通过Acceptor 处理连事件
-   当Acceptor处理连接事件后，MainReactor将连接分配给SubReactor
-   subreactor将连接加入到连接队列进行监听,并创建handler进行各种事件处理
-   当有新事件发生时，subreactor 就会调用对应的 handler 处理
-   handler 通过 read 读取数据，分发给后面的 worker 线程处理
-   worker 线程池分配独立的 worker 线程进行业务处理，并返回结果
-   handler收到响应的结果后，再通过send将结果回给client
-   Reactor主线程可以对应多个Reactor子线程，即MainRecator可以关联多个SubReactor

图示

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_h6I3EqWC69.png)

优点：父线程与子线程的数据交互简单职责明确，父线程只需要接收新连接，子线程完成后续的业务处理。

优点：父线程与子线程的数据交互简单，Reactor 主线程只需要把新连接传给子线程，子线程无需返回数据。

缺点：编程复杂度较高

#### Reactor模式的优点

-   响应快，不必为单个同步时间所阻塞，虽然 Reactor 本身依然是同步的
-   可以最大程度的避免复杂的多线程及同步问题，并且避免了多线程/进程的切换开销
-   扩展性好，可以方便的通过增加 Reactor 实例个数来充分利用 CPU 资源
-   复用性好，Reactor 模型本身与具体事件处理逻辑无关，具有很高的复用性

### Netty模型

描述

-   Netty 抽象出两组线程池 BossGroup 专门负责接收客户端的连接, WorkerGroup专门负责网络的读写
-   BossGroup和WorkerGroup类型都是NioEventLoopGroup
-   NioEventLoopGroup 相当于一个事件循环组，这个组中含有多个事件循环，每一个事件循环是NioEventLoop
-   NioEventLoop 表示一个不断循环的执行处理任务的线程， 每个NioEventLoop 都有一个selector，用于监听绑定在其上的socket的网络通讯
-   NioEventLoopGroup 可以有多个线程，即可以含有多个NioEventLoop每个BossNioEventLoop循环执行的步骤有3步
    -   轮询accept
    -   事件处理accept事件，与client建立连接，生成NioScocketChannel，并将其注册到某个workerNIOEventLoop上的 selector
    -   处理任务队列的任务 ， 即runAllTasks

图示

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_ZcCDrrd11G.png)

-   每个WorkerNIOEventLoop循环执行的步骤
    -   轮询read,write事件
    -   处理 i/o 事件，即 read , write 事件，在对应 NioScocketChannel 处理
    -   处理任务队列的任务，即 runAllTasks
-   每个Worker NIOEventLoop处理业务时，会使用pipeline(管道), pipeline 中包channel，即通过pipeline.可以获取到对应通道,管道中维护了很多的 处理器

### Netty案例—TCP服务

[Netty案例—TCP服务](https://www.wolai.com/gbMHtwDK3TmXjdhRVrF55m "Netty案例—TCP服务")

### 任务队列中的task有三种典型的应用场景

-   用户程序自定义的普通任务
-   用户定义的定时任务
-   非当前Reactor调用Channel的各种方法
    -   例如在推送系统中，根据用户的标识获取用户的Channel然后调用write方法向该用户推送消息，。最终的write会提交到队列中被异步消费

```java
 @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ctx.channel().eventLoop().execute(()->{
            System.out.println("用户自定义的普通任务");
        });
        ctx.channel().eventLoop().schedule(()->{
            System.out.println("用户自定义的定时任务");
        },5, TimeUnit.SECONDS);
    }
```

### 总结—以上

-   Netty抽象出两组线程池，BossGroup专门负责接收来自客户端的连接，WorkGroup专门负责网络的读写
-   NioEventLoop表示一个不断循环执行的线程，每一个NioEventLoop都有一个Selector负责监听绑定在其上面的Socket网络通道
-   NioEventLoop内部采用了串行化的设计，从消息的**读取→解码→处理→编码→发送**，始终由IO线程EventLoop负责

***

-   NioEventLoopGroup下面包含多个NioEventLoop
    -   每个NioEventLoop都含有一个Selector和TaskQueue
    -   每个NioEventLoop的Selector上面可以注册监听多个NioChannel
    -   每个NioChannel都会绑定一个唯一的NioEventLoop上
    -   每个NioChannel都有一个绑定自己的ChannelPipeline

### 异步模型

-   异步是和同步相对的概念，当一个异步调用过程发出后，调用者并不会立即得到结果。实际在处理这个调用的组件在执行完毕之后，通过状态、通知和回调通知调用者
-   Netty中的IO操作是异步的，包括write、bind、connect等等操作会先简单的返回一个ChannelFuture，调用者并不会立即获得结果，而是通过Future-Listen机制，主动的获取异步执行IO操作的结果
-   Netty的异步模型是建立在Future和callback的基础之上的，callback就是回调函数。future的核心思想就是：如果一个处理过程相当的耗时，并不会等待这个处理过程的执行完毕，立即返回一个future对象，可以通过这个future对象监控这个处理过程的状态。

#### future对象的说明：

-   future对象是异步执行的结果，可以通过future提供的方法检测异步执行是否结束
-   ChannelFuture是一个接口，可以在其上添加监听器，当有事件发生时，就会通知这个监听器。

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_6hdEQDrvhH.png)

-   可以在Handler中使用future或者callback实现异步操作：在TaskQueue中添加任务

### Future-Listen机制

-   当Future对象刚刚创建的时候，处于非完成状态，调用者可以通过ChannelFuture来获取操作执行的状态，可以注册监听函数来执行一些操作
-   常用操作如下：

```java
isDone()//判断当前操作是否完成
isSuccess()//判断已完成的操作是否成功
getCause()//获取已完成操作失败的原因
isCanceled()//判断已完成操作是否取消
addListener()//添加监听器

 future.addListener(new ChannelFutureListener() {
                @Override
                public void operationComplete(ChannelFuture channelFuture) throws Exception {
                    if (future.isSuccess()){
                        System.out.println("异步事件成功");
                    }else{
                        System.out.println("异步事件失败");
                    }
                }
            });

```

### HTTP服务案例

[HTTP服务案例](https://www.wolai.com/m4WkJpbNuoq3YFY2NYDv4X "HTTP服务案例")
