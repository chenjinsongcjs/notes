# MySQL进阶

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_opDKrRDeg7.png)

-   MySQL客户端与服务器端的连接方式
    -   TCP/IP连接，最常用
    -   命名管道&#x20;
    -   共享内存
    -   UNIX域套接字

## 一个SQL语句的执行过程

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_o6bOXJTQRP.png)

-   查询缓存

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_s0nZoaVtfL.png)

-   分析器
    -   分析器的作用主要知道“做什么”
    -   对SQL语句进行词法分析，检查SQL的关键字
    -   对SQL语句进行句法分析，检查SQL语句是否合法
-   优化器
    -   优化器的作用主要是知道“怎么做”
    -   优化器的作用主要是解决如何使用索引

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_jIMObeoH4g.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image__goqPz6aiK.png)

## MySQL的存储引擎

-   InnoDB
-   MyiSM
-   Memory
-   Archive

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_pd5xwV7l8R.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_38BYiYqFzc.png)

-   Memory
    -   数据存储在内存中，速度快
    -   数据安全性差
-   Archive
    -   数据压缩，空间利用率高
    -   插入数据块
    -   不支持索引，查询慢

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_RveGaae8uR.png)

### InnoDB是索引组织表

-   InnoDB的表是索引组织起来的
-   有主键就用主键，没有主键就找第一个非空唯一的列作为主键，
-   以上两个条件都没有，会创建6字节的指针，作为主键

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_TkKEMngean.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_U3LQZmG93x.png)

### 为什么说索引即数据

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_-6jqMF8gcH.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_eWDACz2_Dy.png)

-   主索引

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image__7DAZ5BC_u.png)

-   辅助索引
    -   通过某个字段建立辅助索引，辅助索引也是 一颗B+树，只是不存储数据
    -   通过辅助索引找到，对应的行号，再去查找主索引，才能获取数据

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_B_82-17XTg.png)

### InnoDB数据表是如何存储的

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_m3NCjKYS67.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_4PX2yfL_U3.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_JSXLljGz2v.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_PloEUefWXq.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_WEHPTocaAp.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_ZfYaz--9R9.png)

### InnoDB的数据行

-   变长列，指的是存储空间是变长的

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_d36BiqgNLu.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_wscO12lBU6.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_J3kKlvc3xZ.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_S_zLYcgRsf.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_qkfAiAZPK3.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_EceMZ8QznK.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_Byy1U65sW2.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_CPdr0iBqDD.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_9vWkQultQS.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_qBddgvqqh0.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_z0KMkJTXmY.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_waVPllHByp.png)

### 约束

-   唯一约束，插入时开销较大，需要检查数据是否存在

### 如何使用不存在的数据表

-   视图：只存储SQL逻辑，不存储真正的数据

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_rch_wSLgCR.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_ouWx0299Wl.png)

```sql
create 
algorithm merge
view 
as select ...


```

-   怎样提高MySQL大的性能

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_cPlwoRzIcQ.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_HMmclScCxO.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_qxj_573gaT.png)

# SQL分析

[慢查询](obsidian://open?vault=notes&file=MySQL%2F%E6%85%A2%E6%9F%A5%E8%AF%A2)

[如何处理数据更新](obsidian://open?vault=notes&file=MySQL%2F%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%E6%95%B0%E6%8D%AE%E6%9B%B4%E6%96%B0)

[ORM框架](obsidian://open?vault=notes&file=MySQL%2FORM%E6%A1%86%E6%9E%B6)

[数据库备份](obsidian://open?vault=notes&file=MySQL%2F%E6%95%B0%E6%8D%AE%E5%BA%93%E5%A4%87%E4%BB%BD)

[MySQL复制](obsidian://open?vault=notes&file=MySQL%2FMySQL%E5%A4%8D%E5%88%B6)

[数据库扩容](obsidian://open?vault=notes&file=MySQL%2F%E6%95%B0%E6%8D%AE%E5%BA%93%E6%89%A9%E5%AE%B9)

[数据库切换](obsidian://open?vault=notes&file=MySQL%2F%E6%95%B0%E6%8D%AE%E5%BA%93%E5%88%87%E6%8D%A2)

[未来数据库](obsidian://open?vault=notes&file=MySQL%2F%E6%9C%AA%E6%9D%A5%E6%95%B0%E6%8D%AE%E5%BA%93)
