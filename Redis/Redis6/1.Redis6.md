# Redis6

## Redis官网介绍和基本配置

-   英文官网

<https://redis.io/>

-   中文官网

<http://redis.cn/>

-   Redis安装

<https://redis.io/download>

-   Redis命令大全

<http://doc.redisfans.com/>

<http://www.redis.cn/commands.html>

-   `redis-server -v` 查看当前redis版本
-   配置文件修改

```bash
redis.conf配置文件
   2.1 修改daemonize 改为  daemonize yes   开启守护线程，
   2.2 修改protected-mode  yes 改为  protected-mode no  关闭安全模式
   2.3 注释掉 #bin 127.0.0.1  允许外网连接
```

# 进入主题

[Redis单线程 VS 多线程](https://www.wolai.com/fB5CQxsQdQVWmB4ajPtxNj "Redis单线程 VS 多线程")

[redis数据类型及其应用](https://www.wolai.com/kCPZ6nrRgzQP5hsTHX8b4a "redis数据类型及其应用")

[BloomFilter(布隆过滤器)](https://www.wolai.com/tLmLK1PWmC6Nk3XL9SiuFr "BloomFilter(布隆过滤器)")

[缓存的各种问题](https://www.wolai.com/oUAPx9LePxpf1ofaMoxE48 "缓存的各种问题")

[Redis分布式锁](https://www.wolai.com/4yW9U77X8bxDqwuVkPYk3w "Redis分布式锁")

[Redis的缓存过期淘汰策略](https://www.wolai.com/wueAYeKPrrBUVZ48JJ9pjE "Redis的缓存过期淘汰策略")

[redis经典五种数据类型及底层实现](https://www.wolai.com/uSLs4bGzwt34H64urjBbEi "redis经典五种数据类型及底层实现")

[Redis与MySQL数据双写一致性工程落地案例](https://www.wolai.com/tXb4soF6tzt3npf88f8GCa "Redis与MySQL数据双写一致性工程落地案例")

[高性能设计之epoll和IO多路复用深度解析](https://www.wolai.com/2EHfAuZmeTg33RBYdoxHCZ "高性能设计之epoll和IO多路复用深度解析")
