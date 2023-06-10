# Redis基础

## 1、在分布式数据库中CAP原理+BASE

-   传统的ACID属性：
    -   原子性：一组SQL或一个任务要么全部成功要么全部失败，不可以分割
    -   一致性：数据库的操作时从一种一致性向另一种一致性过度的。
    -   隔离性：多线程间各个事务之间是相互隔离的，互不干扰。
    -   持久性：一旦事务操作成功对数据库的改变是持久的。
-   CAP
    -   Consistency：强一致性
    -   Availability：高可用性
    -   Partition tolerance：分区容错性
    -   由于当前的网络硬件一定会出现丢包的情况，分区容错性是我们必须需要实现的。
    -   CAP  NOSQL只能实现其中的两个。
    -   CA：传统Oracle数据库
    -   AP：大多数网站架构的选择
    -   CP：Redis、Mongodb
    -   CA：单点集群，满足一致性，可用性的系统
    -   CP：满足一致性，分区容错性的系统，通常性能不是很高
    -   AP：满足可用性，分区容错性的系统。
    -
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623471636451-02ee68f7-dac1-4923-a061-aa89a885b503.png#align=left\&display=inline\&height=299\&margin=\[object Object]\&name=image.png\&originHeight=597\&originWidth=695\&size=52035\&status=done\&style=none\&width=347.5>)
-   BASE理论
    -   BASE就是为了解决关系型数据库强一致性引起的问题而引起的可用性降低而提出的解决方法。
    -   BASE其实是三个术语的缩写：
        -   基本可用:Basically Available
        -   软状态：Softstate
        -   最终一致性：Eventually consistent
    -   它的思想是通过让系统放松对某一时刻数据一致性的要求来换取系统整体伸缩性和性能上改观。为什么这么说呢，缘由就在于大型系统往往由于地域分布和极高性能的要求，不可能采用分布式事务来完成这些指标，要想获得这些指标，我们必须采用另外一种方式来完成，这里BASE就是解决这个问题的办法

## 2、Redis基本知识

-   单进程：单进程模型来处理客户端的请求。对读写等时间的响应是通过对epoll函数的包装来做到的，Redis的实际处理速度完全依靠主进程的执行效率。
-   Redis默认16个数据库，类似数组下标从零开始，初始默认使用零号库。
-   Select 命令可以用来切换数据库：select 数据库编号
-   dbsize：查看当前数据库的key的数量
-   flushdb：清空当前库
-   flushall：清空所有库
-   16个库都是使用同样的密码。
-   Redis索引都是从零开始
-   默认端口是6379

## 3、Redis的数据类型

### 3.1、Redis的五大数据类型

-   String 字符串
    -   String是Redis最基本的数据类型。
    -   String类型是二进制安全的。redis的String可以包含任何数据。比如jpg图片或者序列化的对象。
    -   一个redis中的字符串最多可以是512M
-   hash 类似java中的map
    -   Redis的hash是一个键值对集合。
    -   hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。
-   List 列表
    -   Redis列表是一个简单的字符串列表，按照插入顺序排序，可以在元素列表的头部或尾部插入一个元素。
    -   Redis列表的底层实际是一个链表
-   set 集合
    -   Redis的set是一个String类型的无序集合，他是通过HashTable实现的。
-   Zset 有序集合
    -   zset的每个元素都会关联一个double类型的分数
    -   redis通过分数为集合中的成员进行从小到大的排序，zset的成员是唯一的，但是分数（socore） 可以重复。

### 3.2、Redis键(key)

-   keys \* 查看当前数据库所有的key
-   exists key  判断某个key是否存在
-   move key db  将指定的key移动到指定的数据库，当前数据库就不存在key
-   expire key 秒钟：为给定的key设置过期时间。
-   ttl key 查看指定key还有多少秒过期，-1表示永不过期。-2代表已过期
-   type key 查看key是什么类型
-   del key 删除key
-   dump key 序列化指定的key，并返回序列化的值
-   persist key 移除key的过期时间，持久化key
-   randomkey  从数据库中随机返回一个key
-   rename key newkey 修改key的名字

### 3.3、Redis字符串  （单值单value）

-   set key value 设置指定key的值
-   get key 获取指定key的值
-   getrange key start end 返回key中字符串值的子字符串
-   getset key value 设置指定key的值为value，并返回旧的值
-   mget k1 k2 ... 获取多个key所对应的值
-   setex key seconds value 将value绑定到key上并设置过期时间
-   setnx key value 当key不存在是才设置key的值
-   setrange key offset value 设置key对应的value从指定偏移量开始
-   strlen key 返回key对应的字符串的长度
-   mset k v \[k v...] 设置多个key-value
-   msetnx k v k v 设置多个k v当不存在k的时候
-   incr key 将key中存储的数字加1
-   incrby key increment 增加指定增量
-   incrbyfloat key increment 增加浮点数
-   decr key 将key中的数字减一
-   decrby key decrement 减去指定数字
-   append key value 追加字符串到末尾

### 3.4、Redis列表（list 单值多value）

-   lpush k v1 v2...从链表头插入多个值
-   rpush k v1 v2...从链表为插入多个值
-   lrange key start end 查看key对应list指定范围的值 0 -1 代表所有
-   lpop key 从list的头取出元素
-   rpop key 从list的尾部取出元素
-   lindex index 按照索引获取元素，从上到下
-   llen key 获取list的长度
-   lrem key count value 删除指定数量的值，0表示全部删除
-   ltrim key start end 截取list中指定的值赋值给当前key
-   rpoplpush 源list 目的list 第一个list删除一个元素 加入到后一个list中 尾出头入。
-   lset key index value 设置list中指定索引的值
-   linsert key Before/after v1 v2 值指定的值的前或后设置值

### 3.5、Redis集合(set 单值多value)

-   sadd key v1 v2 ... 向集合中添加元素
-   smembers key  查看集合中的所有元素
-   sismember key value 判断value是否是集合中的元素
-   scard k 获取集合中元素的个数
-   srem k v 删除集合中的元素
-   srandmember key count 随机取出几个元素 ，超过长度全部取出，负数可能会有重复值。
-   spop key 随机出栈，元素会消失
-   smove k1 k2 element 将k1中的某个值赋给k2
-   数学集合类
    -   差集：sdiff k1 k2 在k1中不在k2中的元素
    -   交集：sinter k1 k2 在k1k2中都有的元素
    -   并集：sunion k1 k2 在k1k2中所有的元素

### 3.6、Redis哈希（Hash KV模式不变，但V是一个键值对）

-   hset/hmset k k1 v1 k2 v2...设置值
-   hget k k1 获取获取哈希表中k1的值
-   hmget k k1 k2...获取哈希表中多个k的值
-   hgetall k 获取哈希表中所有的键值对
-   hdel k k1 删除哈希表中k1这个键值对
-   hkeys k 获取哈希表中所有的键
-   hvals k 获取哈希表中所有的值
-   hincrby/hincrbyfloat k k1 哈希表中k1对应的数字加指定的数值
-   hsetnx k k1 v1 当哈希表中没有对于键值对时才添加
-   hlen 获取哈希表中的键值对的个数
-   hexists key 判断哈希表中是否存在指定的key

### 3.7、Redis有序集合Zset(sorted set)

-   zadd key score value 向有序集合中添加元素
-   zrange key start end withscores  查看指定范围内的元素带有分数
-   zrangebyscores key startscore endscore withscores limit startindex offset 查看指定分数范围内的元素 ( 表示不等于
-   zcard 查看元素个数
-   zcount key startscore endscore 查看指定分数范围内的元素个数
-   zscore key value 查看元素对应的分数
-   zrevrank key values 获取逆序下标
-   zrevrange 逆序显示元素
-   zrevrangebyscore key startscore endscore withscore limit startindex offset  根据分数逆序获取值，注意分数要逆序写。
-   zrem k v 删除指定元素

## 4、配置文件：redis.conf

-   参数说明

redis.conf 配置项说明如下：

1.  Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程

daemonize no
2\. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
pidfile /var/run/redis.pid
3\. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字
port 6379
4\. 绑定的主机地址
bind 127.0.0.1
5.当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
timeout 300
6\. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
loglevel verbose
7\. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null
logfile stdout
8\. 设置数据库的数量，默认数据库为0，可以使用SELECT 命令在连接上指定数据库id
databases 16
9\. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
save 
Redis默认配置文件中提供了三个条件：
save 900 1
save 300 10
save 60 10000
分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。

10\. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
rdbcompression yes
11\. 指定本地数据库文件名，默认值为dump.rdb
dbfilename dump.rdb
12\. 指定本地数据库存放目录
dir ./
13\. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
slaveof 
14\. 当master服务设置了密码保护时，slav服务连接master的密码
masterauth
15\. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH 命令提供密码，默认关闭
requirepass foobared
16\. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息
maxclients 128
17\. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
maxmemory
18\. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认   为no
appendonly no
19\. 指定更新日志文件名，默认为appendonly.aof
 appendfilename appendonly.aof
20\. 指定更新日志条件，共有3个可选值： 
no：表示等操作系统进行数据缓存同步到磁盘（快） 
always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
everysec：表示每秒同步一次（折衷，默认值）
appendfsync everysec

21\. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）
 vm-enabled no
22\. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
 vm-swap-file /tmp/redis.swap
23\. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
 vm-max-memory 0
24\. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值
 vm-page-size 32
25\. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
 vm-pages 134217728
26\. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
 vm-max-threads 4
27\. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
glueoutputbuf yes
28\. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
hash-max-zipmap-entries 64
hash-max-zipmap-value 512
29\. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
activerehashing yes
30\. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
include /path/to/local.conf

## 5、Redis的持久化

### 5.1、RDB(Redis Data Base)

-   在指定的时间间隔内将内存中的数据集快照写入磁盘。
-   Redis会单独创建（fork）一个子进程来进行持久化，先将内存中的数据写入一个临时文件中，待持久化结束后再将这个临时文件替换上一次持久化的文件。整个过程主线程不进行任何IO操作，这就确保了极高的性能，如果需要进行大规模的数据恢复，且对数据的完整性不是非常敏感，那么RDB方式要比AOF方式更高效。
-   RDB的缺点是最后一个持久化的数据可能丢失。
-   Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程。
-   RDB保存的是dump.rdb文件
-   配置位置：见配置第9项
-   如何触发RDB快照：
    -   拷贝快照配置文件
    -   save命令：save命令只进行保存，在保存过程中会阻塞。
    -   bgsave命令：Redis会在后台异步进行快照操作，快照同时还可以响应客户端请求，可以通过lastsave命令获取最后一次执行快照的时间。
    -   执行flushall命令，也会产生dump.rdb文件，但是里面是空的，无意义。
-   如何恢复：
    -   将备份文件（dump.rdb）移动到工作路径下启动redis即可
    -   config get dir  :获取备份文件的目录
-   RDB的优势：
    -   适合大规模数据的恢复
    -   对数据完整性和一致性要求不高
-   RDB的劣势：
    -   在一定的时间间隔内做一次备份，如果redis以外宕机的话，最后一次备份之后的修改的数据就会丢失。
    -   Fork的时候，内存中的数据被克隆一份，大致2倍的膨胀性要考虑。
-   如何停止：
    -   动态停止所有RDB保存规则的方法：redis-cli config set save ""
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623828106686-768bb663-0132-4e03-9fcf-29c25645c689.png#align=left\&display=inline\&height=322\&margin=\[object Object]\&name=image.png\&originHeight=643\&originWidth=884\&size=81851\&status=done\&style=none\&width=442>)

### 5.2、AOF(Append Only File)

-   AOF以日志的形式来记录每个写操作，将Redis执行过程的所有写指令记录下来(读操作不记录)，只允许追加文件不可以改写文件，redis启动之初会读取该文件重新构建数据，
-   换言之：redis重启的或就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。
-   AOF保存的是appendonly.aof文件
-   配置位置：
    -   appendonly yes
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623829541046-be94affc-92ea-4c35-a33d-dee0381b4890.png#align=left\&display=inline\&height=240\&margin=\[object Object]\&name=image.png\&originHeight=480\&originWidth=1993\&size=114621\&status=done\&style=none\&width=996.5>)

-   Rewrite
    -   AOF采用文件追加的方式，文件会越来越大为避免出现此种情况，新增重写机制，当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集，可以使用命令bgrewriteaof
    -   重写的原理：
        -   AOF文件持续增长过大时，会fork出一条新进程来讲文件重写（会先写一个临时文件最后再重命名为appendonly.aof）
        -   遍历新进程的内存数据，每条记录有一条set语句，重写aof文件的操作并没有读取旧的aof文件，而是将整个内存中的数据内容用命令的方式重写了一个新的aof文件。
    -   触发机制：
        -   Redis会记录上一次重写时AOF文件大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发。
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623830914147-7d954982-dde0-4b5f-98b0-c51f16a75d5f.png#align=left\&display=inline\&height=208\&margin=\[object Object]\&name=image.png\&originHeight=416\&originWidth=2469\&size=122314\&status=done\&style=none\&width=1234.5>)

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623830982212-45baac9a-820a-4121-b385-251352bc17fb.png#align=left\&display=inline\&height=317\&margin=\[object Object]\&name=image.png\&originHeight=634\&originWidth=833\&size=72026\&status=done\&style=none\&width=416.5>)

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623831234541-deb2166c-e3b4-41e7-8290-dd2fa06dcb35.png#align=left\&display=inline\&height=397\&margin=\[object Object]\&name=image.png\&originHeight=794\&originWidth=1372\&size=137773\&status=done\&style=none\&width=686>)

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623831422453-d3d4c3fb-9ee6-429d-9907-73a390c7de8e.png#align=left\&display=inline\&height=430\&margin=\[object Object]\&name=image.png\&originHeight=860\&originWidth=1847\&size=234427\&status=done\&style=none\&width=923.5>)

## 6、Redis事务

-   可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，按照顺序串行化的执行而不会被其他命令插入。
-   Redis事务命令：
    -   discard 取消事务，放弃执行事务块中的所有命令
    -   exec 执行所有事务块中的命令
    -   multi 标记事务块的开始
    -   unwatch 取消watch命令对所有key的监视
    -   watch key ... 监视一个或多个key如果在事务执行之前这些key被其他命令修改，那么事务被打断。
-   ## case1：正常执行：
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623832500145-0854500c-8cf9-4944-b385-b85bca0e350e.png#align=left\&display=inline\&height=454\&margin=\[object Object]\&name=image.png\&originHeight=908\&originWidth=641\&size=562898\&status=done\&style=none\&width=320.5>)
-   ## case2：放弃事务
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623832602197-869cbe2c-509f-4af5-a80e-e0487c8d0062.png#align=left\&display=inline\&height=222\&margin=\[object Object]\&name=image.png\&originHeight=443\&originWidth=644\&size=275886\&status=done\&style=none\&width=322>)
-   ## case3：全体连坐(发生一个错误全部不执行)
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623832746199-01d2edd0-1b28-45af-b8c5-f39e4731d126.png#align=left\&display=inline\&height=195\&margin=\[object Object]\&name=image.png\&originHeight=390\&originWidth=1343\&size=514281\&status=done\&style=none\&width=671.5>)
-   ## case4：冤头债主(执行错误，，对的执行，错的不执行)
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623832923517-33baf2db-bf37-4f3b-be32-8996e1eee1c6.png#align=left\&display=inline\&height=244\&margin=\[object Object]\&name=image.png\&originHeight=487\&originWidth=1146\&size=565420\&status=done\&style=none\&width=573>)
-   case5：watch监控
    -   乐观锁：每次获取数据不会加锁，更新数据时会判断数据是否被其他线程更改，通过版本号控制。只有提交的版本大于当前版本才能提交。
    -   悲观锁：每次获取数据时都要加锁，其他线程想要操作当前数据都会阻塞，直到释放锁。
    -   CAS：比较并交换，自旋锁
    -
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623833791247-53debe72-279b-4773-afe4-1d6b8af4f3dd.png#align=left\&display=inline\&height=426\&margin=\[object Object]\&name=image.png\&originHeight=852\&originWidth=2465\&size=232555\&status=done\&style=none\&width=1232.5>)
-   事务的三个阶段
    -   开启：multi开启一个事务块
    -   入队：将多个命令加入到事务队列中，接受到这些命令并不会立即执行，而是放入等待执行的事务队列里面
    -   执行：exec命令触发事务
-   事务三个特性
    -   单独的隔离操作：事务中的所有命令都会被序列化，按照顺序串行的执行，不会被其他客户端发送的命令请求所打断。
    -   没有隔离级别的概念：事务队列中的命令在没有提交之前都不会实际的执行
    -   不保证原子性：redis同一个事务中如果有一条命令执行失败，其他命令会继续执行没有回滚。

## 7、Redis的发布订阅

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623911153218-2b22580b-edeb-4e1b-873d-6469836a2f77.png#align=left\&display=inline\&height=305\&margin=\[object Object]\&name=image.png\&originHeight=609\&originWidth=673\&size=117228\&status=done\&style=none\&width=336.5>)

-   先订阅后发布才能收到消息
-   一次性可以订阅多个消息：subscribe c1 c2 c3....
-   消息发布：publish c1 msg

## 8、Redis的复制(Master/Slave)

-   主从复制：主机负责写，从机负责读
-   配从库不配主库
-   从库配置：slaveof 主库ip 主库端口
-   每次从库与主库断开连接都要重新连接，除非修改了配置文件
-   info replication 查看连接情况
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623918154120-b82dc402-a348-4861-8524-0352945090a8.png#align=left\&display=inline\&height=211\&margin=\[object Object]\&name=image.png\&originHeight=421\&originWidth=1221\&size=52229\&status=done\&style=none\&width=610.5>)

-   一主二仆（一台主机两台从机）
    -   1 切入点问题？slave1、slave2是从头开始复制还是从切入点开始复制?比如从k4进来，那之前的123是否也可以复制     全部复制
    -   2 从机是否可以写？set可否？  从机不能进行写操作
    -   3 主机shutdown后情况如何？从机是上位还是原地待命    从机原地待命
    -   4 主机又回来了后，主机新增记录，从机还能否顺利复制？ 可以
    -   5 其中一台从机down后情况如何？依照原有它能跟上大部队吗？ 可以
-   薪火相传
    -   上一个slave作为下一个slave的主机，salve同样可以接受到其他slave的连接和同步请求，该slave作为链条中下一个的master。可以有效减轻master的写压力。
    -   中途变更转向会清除之前的数据，重新建立连接拷贝数据。
-   反客为主
    -   主机挂掉，使用从机充当主机
    -   命令：slaveof no one
-   复制原理：
    -   salve启动成功连接到master后会发送sync命令
    -   master接收到命令启动后台存盘进程，同时收集所有接受到用于修改数据集命令，在后台进程执行完毕之后，master将传送这个数据文件到slave，以完成一次完全同步。
    -   全量复制：slave在接受到数据库文件后，将其存盘并加载到内存中。
    -   增量复制：master继续将新的收集到的修改命令传送给从机，完成同步。
    -   只要是重新连接主机都要进行一次全量复制。
-   哨兵模式（sentinel）
    -   反客为主的自动模式，能够监控后台主机是否故障，如果主机故障根据投票数自动将从机变为主机
    -   步骤：
        -   编写 sentinel.conf 文件，内容为：sentinel monitor  主机名(自定义)   主机ip 主机端口 1
        -   最后的1 表示当主机宕机后slave通过投票重新选择新的主机。
        -   启动哨兵模式：redis-sentinel   sentinel.conf文件地址
        -   问题：如果主机重启之后会不会出现双master冲突：
            -   不会，原来的主机重启之后会作为从机连接到主机上。
    -   一组sentinel可以监视多个master
-   复制的缺点：复制延时，主机的数据库数据复制到从机上会有一定的延时，特别是从机较多的时候。
