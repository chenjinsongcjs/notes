# Redis分布式锁

-   锁的种类
    -   单机锁，同一个JVM内的synchronized和lock锁
    -   分布式锁，不同的JVM内，单机锁不起作用，资源类在不同的机器间共享。
-   一个分布式锁需要满足的条件
    -   独占性：任何时刻只有一个线程持有锁
    -   高可用：在Redis集群中，不能因为某台主机宕机之后，线程不能正常的获取锁和释放锁
    -   防死锁：杜绝死锁，必须有超时控制机制或者撤销操作
    -   不乱抢：防止张冠李戴，一个获取锁的线程不能去释放其他线程的锁，只能自己释放自己的锁
    -   重入性：同一个节点的同一个线程如果获得锁，它可以再次获得这个锁
-   Redis分布式锁的实现命令
    -   获取锁和设置锁的过期时间要保证原子性
        ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_uypdTtuRkK.png)

### 自己实现Redis分布式锁要注意的问题

-   获取锁和设置锁的过期时间要保证原子性
-   获取锁后要实现释放锁的逻辑
    -   释放锁时要比较当前锁的内容和线程对应锁的内容是否一致，避免释放错其他线程的锁
    -   获取锁的内容和释放锁要保证原子性，使用lua脚本
-   锁的过期时间要大于业务执行的时间，不好确定，使用锁续期的策略
-   考虑锁的高可用性，Redis集群是AP的保证高可用，但是不保证强一致性，异步复制可能导致锁的丢失
-   单节点的分布式锁实现
-   自己实现分布式锁比较困难，使用官方推荐
-   自己实现锁的历程
    -   synchronized单机版OK，上分布式
    -   nginx分布式微服务单机锁不行
    -   取消单机锁，上redis分布式锁setnx
    -   只加了锁，没有释放锁，出异常的话，可能无法释放锁，必须要在代码层面finally释放锁
    -   宕机了，部署了微服务代码层面根本没有走到finally这块，没办法保证解锁，这个key没有被删除，需要有lockKey的过期时间设定
    -   为redis的分布式锁key，增加过期时间此外，还必须要setnx+过期时间必须同一行
    -   必须规定只能自己删除自己的锁，你不能把别人的锁删除了,防止张冠李戴，1删2,2删3
    -   redis集群环境下，我们自己写的也不OK直接上RedLock之Redisson落地实现

#### **redis集群环境下，我们自己写的也不OK,直接上RedLock之Redisson落地实现**

-   Redis官方实现的分布式锁

<https://redis.io/topics/distlock>

<https://github.com/redisson/redisson>

-   RedLock 释放锁的时候也要判断是不是当前线程的锁，不能释放错锁

### Redis分布式锁-Redlock算法Distributed locks with Redis

-   使用场景：
    -   多个服务间保证同一时刻同一时间段内同一用户只能有一个请求(防止关键业务出现并发攻击)

#### RedLock理念

<http://redis.cn/topics/distlock.html>

#### Redisson实现

-   Redisson是java操作Redis的客户端之一，提供了大量的API方便操作Redis
-   Redisson官网

<https://redisson.org/>

-   redisson之Github

<https://github.com/redisson/redisson/wiki/1.-Overview>

-   redisson之解决分布式锁

<https://github.com/redisson/redisson/wiki/8.-Distributed-locks-and-synchronizers>

## 单机案例

#### 加锁

-   加锁实际上就是给Redis中的一个key设置值，为了避免死锁，设置过期时间

#### 解锁

-   解锁就是将key删除，但是不能乱删只能删除自己设置的的那个key
-   lua脚本删除
    -   为了保证解锁操作的原子性，我们用LUA脚本完成这一操作。先判断当前锁的字符串是否与传入的值相等，是的话就删除Key，解锁成功
    ```lua
    if redis.call('get',KEYS[1]) == ARGV[1] then 
       return redis.call('del',KEYS[1]) 
    else
       return 0 
    end
    ```

#### 超时

-   锁key要注意过期时间，不能长时间占有锁

#### 关键

-   加锁关键逻辑

```java
/**
 * 
 * @param key
 * @param uniqueId
 * @param seconds
 * @return
 */
public static boolean tryLock(String key, String uniqueId, int seconds) {
    return "OK".equals(jedis.set(key, uniqueId, "NX", "EX", seconds));
}
```

-   解锁关键逻辑

```java
/**
 *
 * @param key
 * @param uniqueId
 * @return
 */
public static boolean releaseLock(String key, String uniqueId) {
    String luaScript = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
            "return redis.call('del', KEYS[1]) else return 0 end";
    
    return jedis.eval(
            luaScript,
            Collections.singletonList(key),
            Collections.singletonList(uniqueId)
    ).equals(1L);
}
```

## 多机案例

#### 基于setnx的分布式锁有什么问题？（单主模式）

第1列

线程 1 首先获取锁成功，将键值对写入 redis 的 master 节点；
在 redis 将该键值对同步到 slave 节点之前，master 发生了故障；
redis 触发故障转移，其中一个 slave 升级为新的 master；
此时新的 master 并不包含线程 1 写入的键值对，因此线程 2 尝试获取锁也可以成功拿到锁；
此时相当于有两个线程获取到了锁，可能会导致各种预期之外的情况发生，例如最常见的脏数据

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_UDSkzMJRu_.png)

#### redis之父提出了Redlock算法解决这个问题（采用多主模式）

-   Redis也提供了Redlock算法，用来实现基于多个实例的分布式锁。**多master锁变量由多个实例维护**，即使有实例发生了故障，锁变量仍然是存在的，客户端还是可以完成锁操作。Redlock算法是实现高可靠分布式锁的一种有效解决方案，可以在实际开发中使用。

## Redlock算法设计理念

-   Zookeeper集群的CAP

    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_tnuyXe5NxO.png)

    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_hdgtnpmpn1.png)
-   Eruaka集群是AP的

    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_FGJTbQ4IcW.png)
-   Redis集群的AP
    -   redis异步复制造成的锁丢失，
    -   比如：主节点没来的及把刚刚set进来这条数据给从节点，就挂了。
-   单机Redis是CP

### 设计理念

该方案也是基于（set 加锁、Lua 脚本解锁）进行改良的，所以redis之父antirez 只描述了差异的地方，大致方案如下。
假设我们有N个Redis主节点，例如 N = 5这些节点是完全独立的，我们不使用复制或任何其他隐式协调系统，
为了取到锁客户端执行以下操作：

```java
1  获取当前时间，以毫秒为单位；
2  依次尝试从5个实例，使用相同的 key 和随机值（例如 UUID）获取锁。当向Redis 请求获取锁时，客户端应该设置一个超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为 10 秒，则超时时间应该在 5-50 毫秒之间。这样可以防止客户端在试图与一个宕机的 Redis 节点对话时长时间处于阻塞状态。如果一个实例不可用，客户端应该尽快尝试去另外一个 Redis 实例请求获取锁；
3  客户端通过当前时间减去步骤 1 记录的时间来计算获取锁使用的时间。当且仅当从大多数（N/2+1，这里是 3 个节点）的 Redis 节点都取到锁，并且获取锁使用的时间小于锁失效时间时，锁才算获取成功；
4  如果取到了锁，其真正有效时间等于初始有效时间减去获取锁所使用的时间（步骤 3 计算的结果）。
5  如果由于某些原因未能获得锁（无法在至少 N/2 + 1 个 Redis 实例获取锁、或获取锁的时间超过了有效时间），客户端应该在所有的 Redis 实例上进行解锁（即便某些Redis实例根本就没有加锁成功，防止某些节点获取到锁但是客户端没有得到响应而导致接下来的一段时间不能被重新获取锁）。

```

-   该方案为了解决数据不一致的问题，**直接舍弃了异步复制只使用 master 节点**，同时由于舍弃了 slave，为了保证可用性，引入了 N 个节点，官方建议是 5。

客户端只有在满足下面的这两个条件时，才能认为是加锁成功。
条件1：客户端从超过半数（大于等于N/2+1）的Redis实例上成功获取到了锁；
条件2：客户端获取锁的总耗时没有超过锁的有效时间。

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_eVfFgIAJWN.png)

## Redisson源码解析

-   Redis 分布式锁过期了，但是业务逻辑还没处理完怎么办
    -   额外起一个线程，定期检查线程是否还持有锁，如果有则延长过期时间。
    -   Redisson 里面就实现了这个方案，使用“看门狗”定期检查（每1/3的锁时间检查1次），如果线程还持有锁，则刷新过期时间；
    -   在获取锁成功后，给锁加一个 watchdog，watchdog 会起一个定时任务，在锁没有被释放且快要过期的时候会续期

#### 缓存续命

-   通过redisson新建出来的锁key，默认是30秒

    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_sHbPE4Bzwk.png)

    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_pGx8imX7cT.png)
-   这里面初始化了一个定时器，dely 的时间是 internalLockLeaseTime/3。在 Redisson 中，internalLockLeaseTime 是 30s，也就是每隔 10s 续期一次，每次 30s。

    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_uITz6qBVgH.png)

    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_xpzW-eN2rQ.png)
-   客户端A加锁成功，就会启动一个watch dog看门狗，他是一个后台线程，会每隔10秒检查一下，如果客户端A还持有锁key，那么就会不断的延长锁key的生存时间，默认每次续命又从30秒新开始

### 重入锁

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_LbTWsWgcDZ.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_JZn_acFq7q.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_fmFN6GlpN8.png)

### 解锁

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_A-mDCA-MYd.png)
