# HTTPS

HTTP（Hypertext Transfer Protocol）超文本传输协议，是一种用于分布式、协作式和超媒体信息系统的应用层协议，可以说 HTTP 是当代互联网通信的基础。

但是，HTTP 有着一个致命的缺陷，那就是内容是**明文传输**的，没有经过任何加密，而这些明文数据会经过**WiFi、路由器、运营商、机房**等多个物理设备节点，如果在这中间任意一个节点被监听，传输的内容就会完全暴露，，这一攻击手法叫做MITM（Man In The Middle）**中间人**攻击。

## HTTPS实现原理

-   HTTPS其实就是将HTTP的报文信息，经过SSL/TLS加密后传输
-   SSL（Secure Sockets Layer）安全套接层和TLS（Transport Layer Security）传输层安全协议其实是**一套东西**。

### 工作流程

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_UZ_A6qWJJq.png)

1.  用户在浏览器发起HTTPS请求（如 [https://www.mogu.com/），默认使用服务端的443端口进行连接；](https://www.mogu.com/），默认使用服务端的443端口进行连接； "https://www.mogu.com/），默认使用服务端的443端口进行连接；")
2.  HTTPS需要使用一套**CA数字证书**，证书内会附带一个**公钥Pub**，而与之对应的**私钥Private**保留在服务端不公开；
3.  服务端收到请求，返回配置好的包含**公钥Pub**的证书给客户端；
4.  客户端收到**证书**，校验合法性，主要包括是否在有效期内、证书的域名与请求的域名是否匹配，上一级证书是否有效（递归判断，直到判断到系统内置或浏览器配置好的根证书），如果不通过，则显示HTTPS警告信息，如果通过则继续；
5.  客户端生成一个用于对称加密的**随机Key**，并用证书内的**公钥Pub**进行加密，发送给服务端；
6.  服务端收到**随机Key**的密文，使用与**公钥Pub**配对的**私钥Private**进行解密，得到客户端真正想发送的**随机Key**；
7.  服务端使用客户端发送过来的**随机Key**对要传输的HTTP数据进行对称加密，将密文返回客户端；
8.  客户端使用**随机Key**对称解密密文，得到HTTP数据明文；
9.  后续HTTPS请求使用之前交换好的**随机Key**进行对称加解密。

#### 对称加密与非对称加密

-   对称加密是指有一个密钥，用它可以对一段明文加密，加密之后也只能用这个密钥来解密得到明文。
-   非对称加密有两个密钥，一个是公钥，另一个是私钥。一般来说，公钥用来加密，这时密文只能用私钥才能解开。
-   对称加密的密钥在传输过程中由被监听的风险。
-   非对称加密相对与对称加密来所比较耗时

#### CA颁发机构

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_7cNCLh5gxc.png)

-   为了保证公钥传输中不被篡改，又使用了非对称加密的数字签名功能，借助CA机构和系统根证书的机制保证了HTTPS证书的公信力。
-   为了兼顾性能和安全性，使用了非对称加密+对称加密的方案。
