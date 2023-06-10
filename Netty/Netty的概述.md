# Netty的概述

### 原生NIO存在的问题

-   NIO的类库和API繁杂，使用麻烦
-   开发工作量和难度非常大
-   JDK NIO的bug：Epoll Bug，它会导致Selector空轮询，最终导致CPU100%，还没有得到根本性的解决

### Netty的优点

-   Netty对JDK自带的NIO进行了封装，解决了上述NIO的缺点问题
-   设计优雅：适用于各种传输类型的统一API阻塞和非阻塞Socket；基于灵活可扩展的事件模型，可以清晰地分离关注点；高度可制定的线程模型：单线程，一个或多个线程池
-   使用方便：详细记录的Javadoc，用户指南和示例
-   高性能、高吞吐量、低延迟、减少资源消耗，最小化不必要的内存复制
-   安全：完整的SSL/TLS和StartTLS支持
-   社区活跃、不断更新。

### Netty官网

<https://netty.io/>

### Netty的版本说明

-   netty 版本分为netty3.x和netty4 x、netty5.x
-   因为Netty5 出现重大 bug，已经被官网废弃了，目前推荐使用的是Netty4.x的稳定版本
-   目前在官网可下载的版本 netty3.x netty4.0.x 和 netty4.1.x
