# Netty的介绍和应用场景

### Netty介绍

介绍

-   Netty是JBOSS提供的一个java开源框架
-   Netty是一个异步、基于事件驱动的网络应用框架，用于开发高性能和高可用性的网络IO程序
-   Netty是基于TCP协议的、面向客户端的高并发应用，或者是p2p场景下的大量数据持续传输的应用
-   Netty本质上是一个NIO框架，适用于服务器通信相关的应用场景

### Netty的应用场景

Netty的位置

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_ikzffPoW8E.png)

分布式系统中

分布式系统中，各个节点之间需要远程调用，高性能的RPC框架必不可少，Netty作为异步高性能的通信框架，往往被作为基础的通信组件被这些RPC框架使用。

游戏行业中

Netty作为高性能的基础通信组件，提供了TCP/UDP和HTTP协议栈，方便开发和定制私有协议栈、账号登录服务器

地图服务器之间可以通过Netty进行高效的通信

大数据行业

经典的Hadoop的高性能和序列化组件AVRO的RPC框架，默认采用Netty进行跨节点通信
