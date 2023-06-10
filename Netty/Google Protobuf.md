# Google Protobuf

## Netty本身的编解码器和问题分析

-   Netty自身提供了一系列的编解码器，比如：StringEncoder/StringDecoder  ObjectEncode/ObjectDecoder等等，底层是使用java的序列化和反序列化机制完成的，效率不是很高

## Protobuf

-   Protobuf 是 Google 发布的开源项目，全称 Google Protocol Buffers，是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 RPC\[远程过程调用remote procedurecall 数据交换格式(目前很多公司 http+jsontcp+protobuf）
-   支持跨平台、跨语言，即\[客户端和服务器端可以是不同语言编写的]（支持目前绝大多）C＋+、C#、Java、python等）

## Protobuf官方网站

<https://developers.google.cn/protocol-buffers?hl=zh-cn>

### java相关

<https://developers.google.cn/protocol-buffers/docs/javatutorial?hl=zh-cn>
