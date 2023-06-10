# redis数据类型及其应用

## 九大类型

-   String类型
-   list类型
-   hash类型
-   set类型
-   zset类型
-   bitmap类型
-   hpyerloglog类型
-   GEO类型
-   stream（了解）

> Redis命令不区分大小写，但是Redis的键是区分大小写的
> help @类型名词  （查询该类型的命令）

## String类型

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_H7MDd9wnJS.png)

### Hash类型

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_kqTNBZ0W-J.png)

```java
//京东早期商品购物车的实现
新增商品 → hset shopcar:uid1024 334488 1
新增商品 → hset shopcar:uid1024 334477 1
增加商品数量 → hincrby shopcar:uid1024 334477 1
商品总数 → hlen shopcar:uid1024
全部选择 → hgetall shopcar:uid1024
```

### list类型

-   一个**双端链表的结构**，容量是2的32次方减1个元素，大概40多亿，主要功能有push/pop等，一般用在栈、队列、消息队列等场景。
-   lrange 可以做分页的效果

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_uewl5pHDpU.png)

### set类型

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_lis-72yIEq.png)

### zset类型

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_tsr-F12q_R.png)

## 亿级系统中常见的四种统计

#### 聚合统计

-   统计多个集合元素的聚合结果，就是集合的差交并等集合统计
-   交差并集和聚合函数的应用

#### 排序统计

-   在⾯对需要展示最新列表、排行榜等场景时，如果数据更新频繁或者需要分页显示，建议使⽤ZSet

#### 二值统计（Bitmap可用）

-   集合元素的取值就只有0和1两种。在钉钉上班签到打卡的场景中，我们只用记录有签到(1)或没签到(0)

#### 基数统计（hpyerloglog可用）

-   指统计⼀个集合中不重复的元素个数

### Bitmap类型

-   是以String作为底层数据结构对二值进行统计的一种数据类型

第1列

位图本质是数组，它是基于String数据类型的按位的操作。该数组由多个二进制位组成，每个二进制位都对应一个偏移量(我们可以称之为一个索引或者位格)。Bitmap支持的最大位数是2^32位，它可以极大的节约存储空间，使用512M内存就可以存储多大42.9亿的字节信息(2^32 = 4294967296)

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_MN22CHAVAp.png)

-   使用案例
    -   日活统计（UV）
    -   连续签到打卡
    -   最近一周的活跃用户
    -   统计指定用户一年之内的登录天数
    -   某用户按照一年365天，哪几天登陆过？哪几天没有登陆？全年中登录的天数共计多少？
-   基本命令

第1列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_OyWBsHhObg.png)

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_rh6XG43Fnq.png)

### hyperloglog类型

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_MuTy7_GKds.png)

-   是什么

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_zO7mkiQjTk.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_7CqYIqGsQu.png)

-   原理

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_dqwPfmafqb.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_9hfk5iA-Rm.png)

-   为什么redis集群的最大槽数是16384个？

```java
Redis集群并没有使用一致性hash而是引入了哈希槽的概念。Redis 集群有16384个哈希槽，每个key通过CRC16校验后对16384取模来决定放置哪个槽，集群的每个节点负责一部分hash槽。但为什么哈希槽的数量是16384（2^14）个呢？

(1)如果槽位为65536，发送心跳信息的消息头达8k，发送的心跳包过于庞大。
在消息头中最占空间的是myslots[CLUSTER_SLOTS/8]。 当槽位为65536时，这块的大小是: 65536÷8÷1024=8kb 
因为每秒钟，redis节点需要发送一定数量的ping消息作为心跳包，如果槽位为65536，这个ping消息的消息头太大了，浪费带宽。
 
(2)redis的集群主节点数量基本不可能超过1000个。
集群节点越多，心跳包的消息体内携带的数据越多。如果节点过1000个，也会导致网络拥堵。因此redis作者不建议redis cluster节点数量超过1000个。 那么，对于节点数在1000以内的redis cluster集群，16384个槽位够用了。没有必要拓展到65536个。
 
(3)槽位越小，节点少的情况下，压缩比高，容易传输
Redis主节点的配置信息中它所负责的哈希槽是通过一张bitmap的形式来保存的，在传输过程中会对bitmap进行压缩，但是如果bitmap的填充率slots / N很高的话(N表示节点数)，bitmap的压缩率就很低。 如果节点数很少，而哈希槽数量很多的话，bitmap的压缩率就很低。

```

-   基本命令

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_h9kxruwJa5.png)

### GEO类型

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_MjAhUSjC8R.png)
