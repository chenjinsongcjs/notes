# 单点登录SSO

-   单点登录（Signal Sign On）：就是再多系统中，用户只需要在一个系统中登录其他系统就能感知到，当前用户已经登录。

### 单系统的登录设计

-   登录：将用户的信息存储在Session中
    -   如果Session中有用户的信息，那么用户处于登录状态
    -   如果Session中没有用户信息，那么用户处于注销状态
-   注销：就是将用户的信息从Session中删除
-   记住我：Cookie中记录当前SessionID不过期或者设置过期时间，可以保证一段时间的登录状态

### 多系统登录问题

登录依靠Session来实现，多个系统之间Session不共享，不满足高可用的特性。

#### Session不共享的解决方案

-   集群之间Session全局同步复制，（会影响集群的性能，不建议）
-   负载均衡时使用IP通过hash函数映射到指定的服务器，同一个用户IP只能访问同一台主机（如果该主机宕机会丢失大量的Session信息，而且用户不一定是在同一个IP下访问）
-   Session放在第三方缓存Redis中

### Cookie跨域问题

针对Cookie存在跨域问题，有几种解决方案：

1.  服务端将Cookie写到客户端后，客户端对Cookie进行解析，将Token解析出来，此后请求都把这个Token带上就行了
2.  多个域名共享Cookie，在写到客户端的时候设置Cookie的domain。
3.  将Token保存在SessionStroage中（不依赖Cookie就没有跨域的问题了）

### 单点系统

1.  用户第一次登录时，生成一个token写回到Cookie中
2.  并把这个token和用户信息存放在Redis中，
3.  当用户访问其他系统时，通过拦截器，判断是否存在token，进行判断是否登录

### 认证中心模式

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_O0GmFDXwot.png)
