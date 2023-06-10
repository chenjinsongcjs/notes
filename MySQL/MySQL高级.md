# MySQL高级

## 1.概述

-   MySQL是一个关系型数据库管理系统，由瑞典MySQL AB公司开发，目前属于Oracle公司。

## 2.MySQL配置文件

## 3.MySQL逻辑架构介绍

-   和其他数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用，主要体现在存储引擎的架构上。
-   插件式的存储引擎将查询处理和其他的系统任务以及数据的存储提取相分离。这种架构可以根据业务需求和实际需要选择合适的存储引擎。

    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626422972138-d01b3807-bde4-42d1-92f6-81c3ae660edc.png#height=486\&id=gQBj3\&margin=\[object Object]\&name=image.png\&originHeight=972\&originWidth=1494\&originalType=binary\&ratio=1\&size=1096522\&status=done\&style=none\&width=747>)

-   1.连接层
    -   为客户端提供连接服务
-   2.服务层
    -   完成大多数的核心服务功能，如：SQL接口，解析，优化，缓存。。。
-   3.引擎层
    -   真正负责mysql中数据的存储和提取。
-   4.存储层
    -   主要将数据存储在运行于裸设备的文件系统之上。

```纯文本
1.Connectors
指的是不同语言中与SQL的交互
2 Management Serveices & Utilities： 
系统管理和控制工具
3 Connection Pool: 连接池
管理缓冲用户连接，线程处理等需要缓存的需求。
负责监听对 MySQL Server 的各种请求，接收连接请求，转发所有连接请求到线程管理模块。每一个连接上 MySQL Server 的客户端请求都会被分配（或创建）一个连接线程为其单独服务。而连接线程的主要工作就是负责 MySQL Server 与客户端的通信，
接受客户端的命令请求，传递 Server 端的结果信息等。线程管理模块则负责管理维护这些连接线程。包括线程的创建，线程的 cache 等。
4 SQL Interface: SQL接口。
接受用户的SQL命令，并且返回用户需要查询的结果。比如select from就是调用SQL Interface
5 Parser: 解析器。
SQL命令传递到解析器的时候会被解析器验证和解析。解析器是由Lex和YACC实现的，是一个很长的脚本。
在 MySQL中我们习惯将所有 Client 端发送给 Server 端的命令都称为 query ，在 MySQL Server 里面，连接线程接收到客户端的一个 Query 后，会直接将该 query 传递给专门负责将各种 Query 进行分类然后转发给各个对应的处理模块。
主要功能：
a . 将SQL语句进行语义和语法的分析，分解成数据结构，然后按照不同的操作类型进行分类，然后做出针对性的转发到后续步骤，以后SQL语句的传递和处理就是基于这个结构的。
b.  如果在分解构成中遇到错误，那么就说明这个sql语句是不合理的
6 Optimizer: 查询优化器。
SQL语句在查询之前会使用查询优化器对查询进行优化。就是优化客户端请求的 query（sql语句） ，根据客户端请求的 query 语句，和数据库中的一些统计信息，在一系列算法的基础上进行分析，得出一个最优的策略，告诉后面的程序如何取得这个 query 语句的结果
他使用的是“选取-投影-联接”策略进行查询。
       用一个例子就可以理解： select uid,name from user where gender = 1;
       这个select 查询先根据where 语句进行选取，而不是先将表全部查询出来以后再进行gender过滤
       这个select查询先根据uid和name进行属性投影，而不是将属性全部取出以后再进行过滤
       将这两个查询条件联接起来生成最终查询结果
7 Cache和Buffer： 查询缓存。
他的主要功能是将客户端提交 给MySQL 的 Select 类 query 请求的返回结果集 cache 到内存中，与该 query 的一个 hash 值 做一个对应。该 Query 所取数据的基表发生任何数据的变化之后， MySQL 会自动使该 query 的Cache 失效。在读写比例非常高的应用系统中， Query Cache 对性能的提高是非常显著的。当然它对内存的消耗也是非常大的。
如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等
8 、存储引擎接口
存储引擎接口模块可以说是 MySQL 数据库中最有特色的一点了。目前各种数据库产品中，基本上只有 MySQL 可以实现其底层数据存储引擎的插件式管理。这个模块实际上只是 一个抽象类，但正是因为它成功地将各种数据处理高度抽象化，才成就了今天 MySQL 可插拔存储引擎的特色。
     从图2还可以看出，MySQL区别于其他数据库的最重要的特点就是其插件式的表存储引擎。MySQL插件式的存储引擎架构提供了一系列标准的管理和服务支持，这些标准与存储引擎本身无关，可能是每个数据库系统本身都必需的，如SQL分析器和优化器等，而存储引擎是底层物理结构的实现，每个存储引擎开发者都可以按照自己的意愿来进行开发。
    注意：存储引擎是基于表的，而不是数据库。

```

## 4.mysql存储引擎

-   查看命令：
    -   show engines; 查看mysql现在提供的存储引擎
    -   show variables like '%storage\_engine%';查看当前默认引擎
-   ## myisam和innoDB的区别
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626424312174-eccea742-9aa7-47de-8004-93ff38756f5a.png#height=294\&id=AGdU0\&margin=\[object Object]\&name=image.png\&originHeight=588\&originWidth=1353\&originalType=binary\&ratio=1\&size=267337\&status=done\&style=none\&width=676.5>)
    -   InnoDB还支持回滚点的设置

## 5.索引优化分析

### 5.1、性能下降SQL慢，执行时间长、等待时间长

-   查询语句写的不好
-   索引失效
-   关联太多的join
-   服务器调优及各个参数设置(缓冲、线程数等待)

### 5.2、常见通用的join查询

-   ## SQL执行顺序
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626425424717-510168de-9561-4e21-9c28-47d93fb86d1c.png#height=163\&id=avYsh\&margin=\[object Object]\&name=image.png\&originHeight=326\&originWidth=373\&originalType=binary\&ratio=1\&size=40150\&status=done\&style=none\&width=186.5>)

        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626425438191-e527a309-4fa0-47c3-9175-0ef8a90b04e1.png#height=119\&id=oRT9H\&margin=\[object Object]\&name=image.png\&originHeight=238\&originWidth=278\&originalType=binary\&ratio=1\&size=39685\&status=done\&style=none\&width=139>)
    \-
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626425446377-4b225cc9-aeb5-449e-b69e-05b7c1da089b.png#height=114\&id=pkIr2\&margin=\[object Object]\&name=image.png\&originHeight=228\&originWidth=799\&originalType=binary\&ratio=1\&size=55974\&status=done\&style=none\&width=399.5>)
-   ## join图
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626425590590-a056073d-c6e0-48c1-b901-f5bf76b7102a.png#height=298\&id=EEs2t\&margin=\[object Object]\&name=image.png\&originHeight=595\&originWidth=783\&originalType=binary\&ratio=1\&size=360399\&status=done\&style=none\&width=391.5>)

### 5.3、索引简介

-   索引是什么
    -   MySQL官方定义：索引是帮助MySQL高效获取数据的数据结构。
    -   你可以理解为：排好序的快速查找的数据结构。
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626426210899-4935c780-5ad2-4b7e-a459-f84b90b0f7dc.png#height=155\&id=MRwZ7\&margin=\[object Object]\&name=image.png\&originHeight=309\&originWidth=751\&originalType=binary\&ratio=1\&size=135575\&status=done\&style=none\&width=375.5>)
    -   一般来说索引本身很大，不可能全部存储在内存中，因此往往以文件形式存储在硬盘上
    -   我们平时所说的索引，如果没有特别指明，都是指B树(多路搜索树，并不一定是二叉树)结构组织的索引。其中聚集索引，次要索引，覆盖索引，复合索引，前缀索引，唯一索引默认都是使用B+树索引，统称索引。当然,除了B+树这种类型的索引之外，还有哈希索引(hash index)等。
-   索引优势
    -   类似于大学图书馆书目索引，提高数据检索效率，降低数据的IO成本
    -   通过索引列对数据进行排序，降低数据排序成本，降低了CPU的消耗
-   索引劣势
    -   实际上索引也是一张表，该表保存了主键和索引字段，并指向实体表的记录，所以索引列也是要占用空间的。
    -   虽然索引大大提高了查询速度，同时却降低了更新表的速度。如果对表进行增删改，不仅要更新数据，同时还需要更新索引信息。
    -   索引只是提高效率的一个因素，如果你的mysql中有大数据量的表，就需要花时间建立优秀的索引。
-   索引分类
    -   单值索引：一个索引只包含单个列，一个表可以有多个单值索引。建议单值索引不超过5个，优先考虑复合索引。
    -   唯一索引：索引列的值必须唯一，但允许空值
    -   复合索引：一个索引包含多个列
-   基本语法
    -   创建
        -   CREATE \[UNIQUE] INDEX indexname ON tablename(columename(length));
        -   ALTER TABLE tablename ADD \[UNIQUE] INDEX \[indexname]  (columename(length));
        -   如果是char、varchar，length可以小于字段实际长度；如果是blob和text类型必须指定length。
    -   删除
        -   DROP INDEX indexname ON tablename;
    -   查看
        -   SHOW INDEX FROM tablename;
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626428756513-43d48137-022d-4abf-bdc6-45a3fda7b783.png#height=153\&id=pvsmF\&margin=\[object Object]\&name=image.png\&originHeight=306\&originWidth=1225\&originalType=binary\&ratio=1\&size=135095\&status=done\&style=none\&width=612.5>)
-   MySQL索引结构
    -   ## BTree索引
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626429031988-da805a92-f028-48bc-b743-d3e638e8f8d0.png#height=240\&id=pa5Td\&margin=\[object Object]\&name=image.png\&originHeight=240\&originWidth=500\&originalType=binary\&ratio=1\&size=152906\&status=done\&style=none\&width=500>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626429496365-bf85777b-f931-462d-9975-235c101ebfd4.png#height=268\&id=dkW8Y\&margin=\[object Object]\&name=image.png\&originHeight=536\&originWidth=1695\&originalType=binary\&ratio=1\&size=206947\&status=done\&style=none\&width=847.5>)
        -   [http://blog.codinglabs.org/articles/theory-of-mysql-index.html](http://blog.codinglabs.org/articles/theory-of-mysql-index.html "http://blog.codinglabs.org/articles/theory-of-mysql-index.html")
    -   Hash索引
    -   full-text全文索引
    -   R-Tree索引
-   那些情况需要建索引
    -   1.主键自动创建唯一索引
    -   2.频繁作为查询条件的字段应该创建索引
    -   3.查询中与其他表关联的字段，外键关系建立索引。
    -   4.频繁更新的字段不适合创建索引，因为每次更新不仅仅是更新记录还会更新索引，增加IO负担
    -   5.where条件里用不到的字段不创建索引
    -   6.单值、组合索引的选择问题，（在高并发下倾向创建组合索引）
    -   7.查询中排序的字段，排序字段若通过索引去访问将大大提高排序的速度。
    -   8.排序中统计或者分组字段。
-   那些情况不要创建索引
    -   表记录太少
    -   经常增删改的表
    -   数据重复且分布平均的表字段；如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。

### 5.4、性能分析

-   ## MySQL 查询优化
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626431524421-a44470ce-1d25-4ef4-ad85-64f2047ebebd.png#height=97\&id=FmX5Y\&margin=\[object Object]\&name=image.png\&originHeight=194\&originWidth=752\&originalType=binary\&ratio=1\&size=170944\&status=done\&style=none\&width=376>)
-   MySQL常见瓶颈
    -   CPU：CPU饱和的时候一般发生在数据装入内存的时候
    -   IO：磁盘IO瓶颈发生在装入数据远大于内存容量时
    -   服务器硬件的性能瓶颈：top/free/iostat/vmstat来查看系统的性能状态。
-   Explain
    -   是什么：查看执行计划
        -   使用EXPLAIN关键字可以模拟优化器执行SQL语句，从而知道mysql是如何处理你的sql语句，分析你的查询语句或者结构的性能瓶颈。
    -   能干嘛
        -   表的读取顺序
        -   数据读取操作的操作类型
        -   那些索引可以使用
        -   那些索引被实际使用
        -   表之间的引用
        -   每张表有多少行被优化器查询
    -   怎么用
        -   Explain+SQL语句
        -   ## 执行计划包含的信息
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626433096842-97147233-26ee-4774-8ec9-f391b8d1a6f9.png#height=20\&id=i2QCg\&margin=\[object Object]\&name=image.png\&originHeight=40\&originWidth=1919\&originalType=binary\&ratio=1\&size=7697\&status=done\&style=none\&width=959.5>)
    -   各个字段解释
        -   id
            -   select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序。
            -   id相同，执行顺序由上至下
            -   id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行。
            -   id相同和不同，同时存在。id如果相同可以认为是同一组，从上之下执行，在所有组中id值越大优先级越高，越先执行。
        -   ## select\_type
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626435720171-9330e7df-3ac1-4719-bcb1-7f9dd4aa6863.png#height=133\&id=ZUmEz\&margin=\[object Object]\&name=image.png\&originHeight=265\&originWidth=305\&originalType=binary\&ratio=1\&size=91438\&status=done\&style=none\&width=152.5>)
            -   SIMPLE：简单的select查询，查询中不包含子查询或者UNION
            -   PRIMARY：查询中若包含任何复杂的子查询，最外层查询则被标记为primary
            -   SUBQUERY：在select或者where中包含了子查询
            -   DERIVED：在from列表中的子查询被标记为derived(衍生)，mysql会递归执行这些子查询，把结果放在临时表中。
            -   UNION：若第二个SELECT出现在UNION之后，则被标记为UNION;若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED
            -   UNION RESULT：从union表中获取结果的select
        -   table
            -   显示这一行数据时关于那张表的。
        -   ## type
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626437121766-c758cacf-3c99-408f-a18b-b8ef1d138f5e.png#height=37\&id=RIiSm\&margin=\[object Object]\&name=image.png\&originHeight=74\&originWidth=793\&originalType=binary\&ratio=1\&size=43322\&status=done\&style=none\&width=396.5>)
            -   访问类型排列
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626437216902-ff166bb8-b7ab-4109-a801-7bcbdb85adac.png#height=122\&id=a5D1W\&margin=\[object Object]\&name=image.png\&originHeight=244\&originWidth=735\&originalType=binary\&ratio=1\&size=52267\&status=done\&style=none\&width=367.5>)
            -   显示查询使用了何种类型：最好到最差依次为：system>const>eq\_ref>ref>range>index>all
                -   system：表只有一行记录（等于系统表），这是const的特例，平时不会出现，这个也可以忽略。
                -   const：表示通过一次索引就找到了，const用于比较primary key或者unique索引，因为只匹配一行数据，所以很快，如果将主键置于where中，就能将查询转换为常量。
                -   eq\_ref：唯一性索引，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一索引扫描。
                -   ref：非唯一索引扫描，返回匹配某个单独值的所有行。
                -   range：只检索指定范围的行，使用一个索引来选择行，key列显示使用了那个索引，一般就是在你的where语句中出现了between、<、>、in等的查询这种范围扫描索引扫描比全表扫描要好，因为他只需要开始索引的某一点，而结束于另一点，不用扫描全部索引
                -   index：Full index Scan,index与all的区别为index值遍历索引树。因为索引文件通常比数据文件小，，也就是说虽然all和index都是全表扫描，但是index是从索引中读，而all是从硬盘中读。
                -   all：Fulltable Scan 遍历全表找到匹配的行。
                -   备注：一般来说，得保证查询只是达到range级别，最好达到ref。
        -   possible\_keys
            -   显示可能应用到这张表中的索引，一个或多个。查询涉及到的字段若存在索引，则该索引被列出，但不一定被查询实际使用。
        -   key
            -   实际使用的索引，如果为null则没有使用索引。
            -   查询中若使用了覆盖索引，则索引和查询的select字段重叠
        -   key\_len
            -   表示索引中使用的字节数，可以通过该列计算查询中使用的索引的长度。在不损失精度的情况下，越短越好。
            -   key\_len显示的值为索引最大可能长度，并非实际使用长度，即key\_len是根据表定义计算而得，不是通过表内检索出的
        -   ref
            -   显示索引那一列被使用了，如果可能的话，是一个常数，那些列或常量被用于查找索引列上的值。
        -   rows
            -   根据表统计信息及索引 选用情况，大致估算出找到所需的记录所需要读取的行数。越少越好。
        -   Extra：包含不适合在其他列中显示但十分重要的额外信息
            -   Using filesort：
                -   说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引排序进行读取。mysql中无法利用索引完成排序操作称为文件排序。
            -   Using temporary：
                -   使用临时表保存中间结果，mysql在对查询结果排序时使用临时表。常用于order by和group by
            -   Using index：
                -   覆盖索引
                    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626441126156-2bfef365-b70c-4745-800b-fbc907394c6b.png#height=166\&id=YwxbT\&margin=\[object Object]\&name=image.png\&originHeight=332\&originWidth=910\&originalType=binary\&ratio=1\&size=179086\&status=done\&style=none\&width=455>)
                -   表示相应的select操作中使用了覆盖索引（Coveing Index）,避免访问了表的数据行，效率不错！如果同时出现using where，表明索引被用来执行索引键值的查找；如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。
                    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626441482228-2459c41e-ddea-43a9-92b6-853d599817f0.png#height=172\&id=Dc68w\&margin=\[object Object]\&name=image.png\&originHeight=344\&originWidth=682\&originalType=binary\&ratio=1\&size=83524\&status=done\&style=none\&width=341>)
            -   Using where：表明使用where过滤
            -   Using join buffer：使用连接缓存
            -   impossible where：where子句的值总是false，不能获取任何数据
            -   .select tables optimized away：
                -   在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT( \*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。
            -   distinct：优化distinct ，在找到第一匹配的元素后就停止找同样值的操作。
    -   ## 热身case
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626442006302-ff760918-ec6b-4616-b99c-79fcb17dcf20.png#height=115\&id=AKlUq\&margin=\[object Object]\&name=image.png\&originHeight=230\&originWidth=694\&originalType=binary\&ratio=1\&size=44378\&status=done\&style=none\&width=347>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626442012616-9569edc1-a2b2-4694-b2e5-673eb5427043.png#height=118\&id=l1IFS\&margin=\[object Object]\&name=image.png\&originHeight=236\&originWidth=905\&originalType=binary\&ratio=1\&size=184068\&status=done\&style=none\&width=452.5>)
-   索引优化
    -   ## 索引分析
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626443189859-a36187db-42e9-4ee7-aca2-cfd941de7e2c.png#height=185\&id=veQw9\&margin=\[object Object]\&name=image.png\&originHeight=370\&originWidth=598\&originalType=binary\&ratio=1\&size=114035\&status=done\&style=none\&width=299>)
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626443196038-3327d741-cebe-4998-86ac-565571c3020b.png#height=131\&id=wphte\&margin=\[object Object]\&name=image.png\&originHeight=261\&originWidth=845\&originalType=binary\&ratio=1\&size=106131\&status=done\&style=none\&width=422.5>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626443210619-fb918842-0cb2-4787-936d-5a85f1383d46.png#height=147\&id=fsiZn\&margin=\[object Object]\&name=image.png\&originHeight=293\&originWidth=745\&originalType=binary\&ratio=1\&size=81073\&status=done\&style=none\&width=372.5>)
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626443214712-4d0eeaf9-24b9-4e8c-b689-4c1923c1106b.png#height=44\&id=fejIJ\&margin=\[object Object]\&name=image.png\&originHeight=88\&originWidth=616\&originalType=binary\&ratio=1\&size=33259\&status=done\&style=none\&width=308>)
    -   避免索引失效
        -   ## 全值匹配
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626443400248-d58658a5-4f64-4c1d-9e4a-556a2da4ce6e.png#height=284\&id=DHHB0\&margin=\[object Object]\&name=image.png\&originHeight=567\&originWidth=1136\&originalType=binary\&ratio=1\&size=195424\&status=done\&style=none\&width=568>)
        -   最佳左前缀法则
            -   如果索引了多列，要遵守最佳左前缀法则，指的是查询从索引的最左前列开始并且不跳过索引中的列。
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626443721813-90edde69-0dda-4288-acb3-bb4199fd3f85.png#height=160\&id=Rwv0p\&margin=\[object Object]\&name=image.png\&originHeight=319\&originWidth=590\&originalType=binary\&ratio=1\&size=59050\&status=done\&style=none\&width=295>)
        -   ## 不要在索引列上做任何操作(计算、函数、自动或手动的类型转换)，会导致索引失效而转为全表扫描。
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626443858732-a985b3ca-653f-451c-9eda-b3d5a4ec5622.png#height=160\&id=PJQRX\&margin=\[object Object]\&name=image.png\&originHeight=319\&originWidth=590\&originalType=binary\&ratio=1\&size=59050\&status=done\&style=none\&width=295>)
        -   ## 存储引擎不能使用索引中范围条件右边的列
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626444126287-99aa9711-d6e6-4dd7-8495-bcad00dafa55.png#height=258\&id=zz3hY\&margin=\[object Object]\&name=image.png\&originHeight=516\&originWidth=1280\&originalType=binary\&ratio=1\&size=338150\&status=done\&style=none\&width=640>)
        -   ## 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少select\*
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626444452523-d706b2b5-9233-4fdd-acf7-a32a0a50da4e.png#height=262\&id=cVhUt\&margin=\[object Object]\&name=image.png\&originHeight=524\&originWidth=1368\&originalType=binary\&ratio=1\&size=356462\&status=done\&style=none\&width=684>)
        -   ## mysql在使用不等于（！=或者<>）的时候无法使用索引会导致全表扫描
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626444541601-e92b0742-58ec-4a39-b387-fda174dd20d5.png#height=196\&id=mV4k9\&margin=\[object Object]\&name=image.png\&originHeight=391\&originWidth=1167\&originalType=binary\&ratio=1\&size=223453\&status=done\&style=none\&width=583.5>)
        -   ## is null,is not null 也无法使用索引
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626444655547-689a0661-7b46-41ab-9ddb-648e992f4efa.png#height=67\&id=QPH1E\&margin=\[object Object]\&name=image.png\&originHeight=133\&originWidth=539\&originalType=binary\&ratio=1\&size=43421\&status=done\&style=none\&width=269.5>)
        -   ## like以通配符开头（'\$abc...'）mysql索引失效会变成全表扫描操作
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626444742754-8483ca12-65cd-4eca-8705-44c82cb2ddfd.png#height=260\&id=UKSwp\&margin=\[object Object]\&name=image.png\&originHeight=519\&originWidth=1169\&originalType=binary\&ratio=1\&size=283354\&status=done\&style=none\&width=584.5>)
        -   ## 字符串不加单引号索引失效
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626444854802-b34d715a-62c9-4bf9-a22c-e391eceb2263.png#height=179\&id=msrm3\&margin=\[object Object]\&name=image.png\&originHeight=357\&originWidth=806\&originalType=binary\&ratio=1\&size=65336\&status=done\&style=none\&width=403>)
        -   ## 少用or,用它连接时会索引失效
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626444897550-116a5c49-e652-49e6-9b1c-e7524022febb.png#height=86\&id=hdjhw\&margin=\[object Object]\&name=image.png\&originHeight=171\&originWidth=618\&originalType=binary\&ratio=1\&size=54223\&status=done\&style=none\&width=309>)
        -   总结
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626445011866-e343fa3a-b4cb-407b-886f-fb6b18e7e8f4.png#height=164\&id=XJriI\&margin=\[object Object]\&name=image.png\&originHeight=327\&originWidth=805\&originalType=binary\&ratio=1\&size=100058\&status=done\&style=none\&width=402.5>)
        -
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626445017791-b326c757-b09c-4f18-b98f-e9ac8e70a7f7.png#height=86\&id=RGIdf\&margin=\[object Object]\&name=image.png\&originHeight=172\&originWidth=239\&originalType=binary\&ratio=1\&size=43766\&status=done\&style=none\&width=119.5>)
        -   like KK%相当于=常量     %KK和%KK% 相当于范围
    -   ## 一般性建议
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626445344718-c82f97d7-deed-4873-a435-e08496c59c72.png#height=108\&id=IXLAh\&margin=\[object Object]\&name=image.png\&originHeight=216\&originWidth=1110\&originalType=binary\&ratio=1\&size=47064\&status=done\&style=none\&width=555>)

## 6.查询截取分析

### 6.1、查询优化

![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626510263107-026536e4-db9a-455c-bbb6-ca07e9788930.png#height=162\&id=Ylmhy\&margin=\[object Object]\&name=image.png\&originHeight=324\&originWidth=606\&originalType=binary\&ratio=1\&size=100299\&status=done\&style=none\&width=303>)

-   ## 永远小表驱动大表，类似嵌套循环Nested Loop
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626510587952-396378ee-66f4-4a26-8575-bb1745de3022.png#height=213\&id=DPsfJ\&margin=\[object Object]\&name=image.png\&originHeight=425\&originWidth=683\&originalType=binary\&ratio=1\&size=124733\&status=done\&style=none\&width=341.5>)
    \-
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626510827546-aa15b8aa-06ab-4311-abe1-356972e30a9c.png#height=35\&id=omY8L\&margin=\[object Object]\&name=image.png\&originHeight=70\&originWidth=736\&originalType=binary\&ratio=1\&size=26152\&status=done\&style=none\&width=368>)
    \-
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626510832651-578b58c8-9ce4-4ab1-b245-237b38406e9a.png#height=51\&id=xSJqZ\&margin=\[object Object]\&name=image.png\&originHeight=101\&originWidth=842\&originalType=binary\&ratio=1\&size=37714\&status=done\&style=none\&width=421>)
-   order by 关键字优化
    -   ## order by子句，尽量使用index方式排序，避免使用filesort方式排序
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626511394487-9a3a7299-cfc9-4515-b2d7-0b527f766053.png#height=263\&id=g1CIN\&margin=\[object Object]\&name=image.png\&originHeight=525\&originWidth=1297\&originalType=binary\&ratio=1\&size=329315\&status=done\&style=none\&width=648.5>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626511406726-4fc3f6f7-ab16-40d9-9f94-948d734aeb5f.png#height=258\&id=DVSMi\&margin=\[object Object]\&name=image.png\&originHeight=516\&originWidth=1282\&originalType=binary\&ratio=1\&size=320999\&status=done\&style=none\&width=641>)
        -   MySQL支持二种方式的排序，FileSort和Index,Index效率高。它指MySQL扫描索引本身完成排序。FileSort方式效率较低。
        -   order by满足两种情况，会使用index方式排序
            -   order by语句使用索引最左前列
            -   使用where字句与order by字句条件组合满足索引最左前列。
    -   尽可能在索引列上完成排序操作，遵照索引键的最佳左前缀
    -   如果不在索引列上，filesort有两种算法：mysql就要启动双路排序和单路排序。
        -   双路排序：mysql在4.1之前使用双路排序，字面意思就是两次扫描磁盘，最终得到数据，读取行指针和orderby列，对他们进行排序，然后扫描已经排序好的列表，按照列表总的值从列表中读取相应的数据传输。（从磁盘中取排序字段，在buffer中进行排序，在从磁盘取得其他字段）
        -   取一批数据，要对磁盘进行两次扫描，众所周知，I\O是很耗时的，所以在mysql4.1之后，出现了第二张改进的算法，就是单路排序。
        -   单路排序算法：从磁盘读取查询需要的所有列，按照orderby列在buffer对它们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据，并且把随机IO变成顺序IO，但是它会使用更多的空间，因为它把每一行都保存在内存中了。
        -   结论及问题
            -   由于单路排序是后面出来的，总体而言好于双路排序
            -   ## 但是用单路有问题
                    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626512683920-087f3d63-79be-47a2-817b-998764aef738.png#height=57\&id=wBO2M\&margin=\[object Object]\&name=image.png\&originHeight=113\&originWidth=830\&originalType=binary\&ratio=1\&size=75147\&status=done\&style=none\&width=415>)
            -   优化策略：
                -   增大sort\_buffer\_size参数设置
                -   增大max\_length\_for\_sort\_data参数设置
                    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626513018501-7a28b66d-1b81-4221-8aad-1d5853fa395c.png#height=181\&id=LQrrH\&margin=\[object Object]\&name=image.png\&originHeight=362\&originWidth=934\&originalType=binary\&ratio=1\&size=211299\&status=done\&style=none\&width=467>)
            -   总结
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626513222968-c5cfc38a-dffb-445d-907e-5ff692d2a074.png#height=194\&id=tMepA\&margin=\[object Object]\&name=image.png\&originHeight=388\&originWidth=568\&originalType=binary\&ratio=1\&size=155982\&status=done\&style=none\&width=284>)
-   group by 关键字优化
    -   group by实质是先排序后分组，遵照索引键的最佳左前缀
    -   当无法使用索引列，增大max\_length\_for\_sort\_data参数的设置+增大sort\_buffer\_size参数的设置
    -   where高于having,能写在where限定的条件就不要去having限定了。

### 6.2、慢查询日志

<https://www.cnblogs.com/kerrycode/p/5593204.html>

-   是什么

![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626513955559-afaa8b02-db45-45cc-886d-1b0a97953701.png#height=161\&id=L18gZ\&margin=\[object Object]\&name=image.png\&originHeight=321\&originWidth=955\&originalType=binary\&ratio=1\&size=154358\&status=done\&style=none\&width=477.5>)

-   ## 怎么玩
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626514164322-5b8e593e-5e09-4004-bd27-8c2a13d75909.png#height=68\&id=qKcd8\&margin=\[object Object]\&name=image.png\&originHeight=135\&originWidth=950\&originalType=binary\&ratio=1\&size=74107\&status=done\&style=none\&width=475>)
    -   查看是否开启及如何开启
        -   show variables like '%slow\_query\_log%';
        -   set global slow\_query\_log = 1;
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626514431617-ef8d65d0-249d-4b21-be32-bc8eaebee890.png#height=186\&id=PJrmD\&margin=\[object Object]\&name=image.png\&originHeight=371\&originWidth=725\&originalType=binary\&ratio=1\&size=78578\&status=done\&style=none\&width=362.5>)
        -
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626514437083-09e001a4-f2b3-414b-8207-440e2fe69920.png#height=127\&id=XHUXf\&margin=\[object Object]\&name=image.png\&originHeight=254\&originWidth=956\&originalType=binary\&ratio=1\&size=124455\&status=done\&style=none\&width=478>)
    -   ## 那么开启慢查询日志后，什么样的SQL参会记录到慢查询里面？
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626514754424-25868ec2-39c8-47ab-9719-6983118fecce.png#height=163\&id=qG7gi\&margin=\[object Object]\&name=image.png\&originHeight=325\&originWidth=731\&originalType=binary\&ratio=1\&size=113035\&status=done\&style=none\&width=365.5>)
    -   查看当前多少秒算慢：SHOW VARIABLES LIKE '%long\_query\_time%';
    -   设置慢的阙值时间：SET GLOBAL long\_query\_time = 5;
    -   为什么设置后看不出变化？：需要重新连接或者新开一个回话才能看到修改值。SHOW VARIABLES LIKE 'long\_query\_time%';
    -   ## 记录慢SQL并后续分析：
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626515266448-8428f374-8641-4982-91a6-61abc3520265.png#height=203\&id=y6piz\&margin=\[object Object]\&name=image.png\&originHeight=405\&originWidth=884\&originalType=binary\&ratio=1\&size=105849\&status=done\&style=none\&width=442>)
    -   ## 查询当前系统中有多少条慢查询记录
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626515402728-81985c75-1488-4623-83de-8841375814d3.png#height=114\&id=VPyt6\&margin=\[object Object]\&name=image.png\&originHeight=227\&originWidth=407\&originalType=binary\&ratio=1\&size=28522\&status=done\&style=none\&width=203.5>)
    -   ## 配置
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626515573272-a99a2a93-3c33-43c7-9c04-03ed6368fa92.png#height=83\&id=wTcGL\&margin=\[object Object]\&name=image.png\&originHeight=165\&originWidth=450\&originalType=binary\&ratio=1\&size=35601\&status=done\&style=none\&width=225>)
-   日志分析工具mysqldumpshow
    -   查看mysqldumpshow的帮助信息

        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626515814862-123296bc-7fe7-44b0-b4bf-d0ac6d72cd97.png#height=196\&id=nIbP7\&margin=\[object Object]\&name=image.png\&originHeight=392\&originWidth=581\&originalType=binary\&ratio=1\&size=52113\&status=done\&style=none\&width=290.5>)
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626515830072-66f4bdaa-775b-4ca1-a596-c2bea795e19a.png#height=333\&id=C6uPQ\&margin=\[object Object]\&name=image.png\&originHeight=665\&originWidth=1118\&originalType=binary\&ratio=1\&size=58639\&status=done\&style=none\&width=559>)
-   ## 工作常用参考
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626515989943-c4b47d85-1598-4e4a-b32e-a8c11bb483eb.png#height=153\&id=Ccz7J\&margin=\[object Object]\&name=image.png\&originHeight=306\&originWidth=639\&originalType=binary\&ratio=1\&size=132618\&status=done\&style=none\&width=319.5>)

### 6.3、show profiles

<https://www.cnblogs.com/sxdcgaq8080/p/11844079.html>

-   是什么：是mysql提供可以用来分析当前会话中语句执行的资源消耗情况，可以用于SQL调优测量。
-   官网：[http://dev.mysql.com/doc/refman/5.5/en/show-profile.html](http://dev.mysql.com/doc/refman/5.5/en/show-profile.html "http://dev.mysql.com/doc/refman/5.5/en/show-profile.html")
-   默认情况下，参数处于关闭状态，并保存最近15次运行结果。
-   分析步骤：
    -   ## 1.是否支持，查看当前的SQL版本是否支持
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626516685206-6144be19-b158-4d2a-b38e-af60be31a09c.png#height=163\&id=hXWkf\&margin=\[object Object]\&name=image.png\&originHeight=325\&originWidth=361\&originalType=binary\&ratio=1\&size=44555\&status=done\&style=none\&width=180.5>)
    -   2.开启功能，默认是关闭，使用前需要开启
        -   SET profiling = 1；
    -   3.运行SQL
    -   4.查看结果，show profiles;
    -   ## 5.诊断SQL，show profile cpu,block io for query 上一步前面的问题SQL 数字号码；
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626517400555-86f61352-933b-4f85-9da3-b54f869ad106.png#height=161\&id=UJUQz\&margin=\[object Object]\&name=image.png\&originHeight=322\&originWidth=756\&originalType=binary\&ratio=1\&size=114255\&status=done\&style=none\&width=378>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626517416545-64a69f7f-78cf-40eb-ad1a-11ce85359cc4.png#height=328\&id=vcrqo\&margin=\[object Object]\&name=image.png\&originHeight=655\&originWidth=1481\&originalType=binary\&ratio=1\&size=78631\&status=done\&style=none\&width=740.5>)
    -   6.日常开发需要注意的结论
        -   converting HEAP to MyISAM 查询结果太大，内存都不够用了往磁盘上搬了。
        -   Creating tmp table 创建临时表
            -   拷贝数据到临时表
            -   用完删除
                ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626517772613-d542a50c-8c97-407f-9995-0a6203c69ba3.png#height=192\&id=YQuQQ\&margin=\[object Object]\&name=image.png\&originHeight=383\&originWidth=253\&originalType=binary\&ratio=1\&size=51882\&status=done\&style=none\&width=126.5>)
        -   Copying to tmp table on disk 把内存中临时表复制到磁盘，危险！！！
        -   locked

### 6.4、全局查询日志

-   ## 配置启用
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626517954293-592ff997-8fa8-4dca-a315-b33dd587116b.png#height=107\&id=LnZV8\&margin=\[object Object]\&name=image.png\&originHeight=213\&originWidth=277\&originalType=binary\&ratio=1\&size=38861\&status=done\&style=none\&width=138.5>)
-   ## 编码启用
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626518007801-c97715d8-177b-400e-9908-825af8ab16d3.png#height=181\&id=G7QWQ\&margin=\[object Object]\&name=image.png\&originHeight=361\&originWidth=803\&originalType=binary\&ratio=1\&size=73056\&status=done\&style=none\&width=401.5>)
-   永远不要在生产环境开启这个功能。

## 7.MySQL锁机制

### 7.1、概述

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626518160648-7061739a-ae65-4014-979b-15cd2b7e2981.png#height=68\&id=sbm4l\&margin=\[object Object]\&name=image.png\&originHeight=135\&originWidth=938\&originalType=binary\&ratio=1\&size=97657\&status=done\&style=none\&width=469>)

-   锁分类
    -   从数据操作的类型(读、写)分
        -   读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响
        -   写锁（排它锁）：当前写操作没有完成前，它会阻断其他写锁和读锁。
    -   从数据操作的颗粒度
        -   表锁
        -   行锁

### 7.2、三锁

-   表锁(偏读)
    -   特点：偏向myisam存储引擎，开销小，加锁块，无死锁，锁定粒度大，发生锁冲突的概率高，并发最低。
    -
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626519592250-606ecf3f-42d2-403c-9712-3880efb9dd82.png#height=83\&id=FtJDa\&margin=\[object Object]\&name=image.png\&originHeight=166\&originWidth=438\&originalType=binary\&ratio=1\&size=38173\&status=done\&style=none\&width=219>)
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626519596323-75c9c4b5-bf0f-4ed6-87c5-2701eea9c142.png#height=29\&id=SLUUB\&margin=\[object Object]\&name=image.png\&originHeight=57\&originWidth=129\&originalType=binary\&ratio=1\&size=7123\&status=done\&style=none\&width=64.5>)
    -   ## 加读锁
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626519689355-8e507957-0d8c-40bf-a578-74ec174b9ba6.png#height=187\&id=rF5vy\&margin=\[object Object]\&name=image.png\&originHeight=373\&originWidth=936\&originalType=binary\&ratio=1\&size=83226\&status=done\&style=none\&width=468>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626519695332-9106558a-b4dd-4e92-be2c-226ec2094141.png#height=149\&id=XNoAr\&margin=\[object Object]\&name=image.png\&originHeight=298\&originWidth=944\&originalType=binary\&ratio=1\&size=81512\&status=done\&style=none\&width=472>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626519703662-7806416b-8eff-44aa-9438-bacb49ed7bef.png#height=93\&id=FqNKH\&margin=\[object Object]\&name=image.png\&originHeight=186\&originWidth=931\&originalType=binary\&ratio=1\&size=59677\&status=done\&style=none\&width=465.5>)
    -   ## 加写锁
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626520187124-1319c673-299b-4ec5-aab5-761a612904eb.png#height=199\&id=B24zx\&margin=\[object Object]\&name=image.png\&originHeight=398\&originWidth=832\&originalType=binary\&ratio=1\&size=125178\&status=done\&style=none\&width=416>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626520193738-e4d3c64a-8785-4246-a028-a2aba15989ad.png#height=171\&id=ZNngu\&margin=\[object Object]\&name=image.png\&originHeight=342\&originWidth=711\&originalType=binary\&ratio=1\&size=47725\&status=done\&style=none\&width=355.5>)
    -   ## 结论
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626520459086-a180b92e-1959-41ec-b73a-a7133cc28131.png#height=182\&id=dU7TD\&margin=\[object Object]\&name=image.png\&originHeight=363\&originWidth=932\&originalType=binary\&ratio=1\&size=166494\&status=done\&style=none\&width=466>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626520480215-36c86127-89d8-4512-b349-be46d4e2b2f5.png#height=24\&id=ED61F\&margin=\[object Object]\&name=image.png\&originHeight=48\&originWidth=941\&originalType=binary\&ratio=1\&size=20441\&status=done\&style=none\&width=470.5>)
    -   ## 表锁分析
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626520647537-e242cfff-25ca-417b-aaf2-b7a896cd1ac8.png#height=172\&id=me8LK\&margin=\[object Object]\&name=image.png\&originHeight=343\&originWidth=660\&originalType=binary\&ratio=1\&size=73152\&status=done\&style=none\&width=330>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626520652220-af5df0b6-8b7d-49e7-9fca-8f0171e96e04.png#height=51\&id=w3joI\&margin=\[object Object]\&name=image.png\&originHeight=101\&originWidth=933\&originalType=binary\&ratio=1\&size=82519\&status=done\&style=none\&width=466.5>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626520658439-6737eb59-a977-4da7-ac4c-c19647c63881.png#height=22\&id=hy7HQ\&margin=\[object Object]\&name=image.png\&originHeight=44\&originWidth=932\&originalType=binary\&ratio=1\&size=42920\&status=done\&style=none\&width=466>)
-   行锁(偏写)
    -   特点：偏向innoDB存储引擎，开销大，加锁满，锁颗粒度小，发生锁冲突的概率最低，并发度最高。
    -   InnoDB与MyISAM的最大不同有两点：一是支持事务（TRANSACTION）;二是采用了行级锁
    -   ## 行锁支持事务
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626521496295-241fdb15-b10e-43e5-bf65-174194a9d94d.png#height=100\&id=nnwi7\&margin=\[object Object]\&name=image.png\&originHeight=199\&originWidth=941\&originalType=binary\&ratio=1\&size=186683\&status=done\&style=none\&width=470.5>)
        -   并发事务带来的问题
            -   ## 更新丢失
                    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626521599392-d614bae9-8b76-4e1c-8ccf-7c301475e2bb.png#height=103\&id=KYpGd\&margin=\[object Object]\&name=image.png\&originHeight=206\&originWidth=940\&originalType=binary\&ratio=1\&size=119975\&status=done\&style=none\&width=470>)
            -   ## 脏读
                    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626521691722-a1eb39f0-cdd4-4d1b-a106-82bdb8f11734.png#height=88\&id=m6FyW\&margin=\[object Object]\&name=image.png\&originHeight=175\&originWidth=943\&originalType=binary\&ratio=1\&size=112806\&status=done\&style=none\&width=471.5>)
            -   ## 不可重复读
                    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626521708959-834f1eae-ecad-4a9e-b39b-e2adf7e37141.png#height=68\&id=tt1fS\&margin=\[object Object]\&name=image.png\&originHeight=135\&originWidth=948\&originalType=binary\&ratio=1\&size=64018\&status=done\&style=none\&width=474>)
            -   ## 幻读
                    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626521836468-f7f69787-e944-423d-a9af-6e3f1b79a6f5.png#height=126\&id=Jsh3J\&margin=\[object Object]\&name=image.png\&originHeight=252\&originWidth=937\&originalType=binary\&ratio=1\&size=84568\&status=done\&style=none\&width=468.5>)
            -   ## 事务隔离级别
                    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626521918808-4fc11734-6143-4f67-a001-85736e671161.png#height=184\&id=m1fF0\&margin=\[object Object]\&name=image.png\&originHeight=367\&originWidth=942\&originalType=binary\&ratio=1\&size=193995\&status=done\&style=none\&width=471>)
    -   ## 行锁基本演示
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626522159916-d47a55c4-467e-4a64-8942-278f000d8ea4.png#height=185\&id=s3Pxg\&margin=\[object Object]\&name=image.png\&originHeight=370\&originWidth=896\&originalType=binary\&ratio=1\&size=100394\&status=done\&style=none\&width=448>)
    -   无索引行锁升级为表锁
        -   varchar  不用 ' '  导致系统自动转换类型, 行锁变表锁
    -   ## 间隙锁的危害
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626522344444-a8587501-ce9c-49dc-88df-9ebbfd8b90ca.png#height=304\&id=ADIlM\&margin=\[object Object]\&name=image.png\&originHeight=608\&originWidth=1421\&originalType=binary\&ratio=1\&size=376172\&status=done\&style=none\&width=710.5>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626522349503-ca9bf68a-337b-40bb-ad37-bd413e48aafd.png#height=25\&id=fsod3\&margin=\[object Object]\&name=image.png\&originHeight=49\&originWidth=939\&originalType=binary\&ratio=1\&size=57308\&status=done\&style=none\&width=469.5>)
    -   ## 如何锁定一行
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626522621222-65b2fb48-e9c4-4621-856f-58100b0c3d8d.png#height=146\&id=aMkVF\&margin=\[object Object]\&name=image.png\&originHeight=291\&originWidth=927\&originalType=binary\&ratio=1\&size=55847\&status=done\&style=none\&width=463.5>)
    -   ## 结论
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626523163239-996b526e-9b78-46f7-b3cd-678997311eea.png#height=71\&id=cc4jM\&margin=\[object Object]\&name=image.png\&originHeight=142\&originWidth=949\&originalType=binary\&ratio=1\&size=112587\&status=done\&style=none\&width=474.5>)
    -   ## 行锁分析
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626523246236-25275ea8-6f3a-4a6d-8d0f-e1807b2c1bda.png#height=176\&id=udI99\&margin=\[object Object]\&name=image.png\&originHeight=352\&originWidth=633\&originalType=binary\&ratio=1\&size=102940\&status=done\&style=none\&width=316.5>)
        \-
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626523250305-b78bff40-cc24-4a73-8de2-a4b00a1e494a.png#height=124\&id=iLfz9\&margin=\[object Object]\&name=image.png\&originHeight=248\&originWidth=936\&originalType=binary\&ratio=1\&size=129091\&status=done\&style=none\&width=468>)
    -   ## 优化建议
            ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626523605379-97ba53a8-362f-4ed6-8a4d-f036dce5bc07.png#height=155\&id=qyfnD\&margin=\[object Object]\&name=image.png\&originHeight=309\&originWidth=879\&originalType=binary\&ratio=1\&size=38748\&status=done\&style=none\&width=439.5>)
-   页锁
    -   开销和加锁时间界于表锁和行锁之间：会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。

## 8.主从复制

### 8.1、复制的基本原理

-   slave会从master中读取bin-log来进行数据同步
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626524167934-4940fd96-98d2-4835-baa5-dd27ece3f510.png#height=185\&id=FjkdX\&margin=\[object Object]\&name=image.png\&originHeight=369\&originWidth=760\&originalType=binary\&ratio=1\&size=150389\&status=done\&style=none\&width=380>)

### 8.2、复制的基本原则

-   每个slave只有一个master
-   每个slave只能有一个唯一的服务器id
-   每个master可以有多个slave
-   复制最大的问题就是延时。

### 8.3、一主一从常见配置

-   mysql版本一致且后台以服务运行
-   主从都配置在【mysqld】节点下，都是小写
-   主机修改my.ini配置文件
    -   主服务器唯一id：server-id = 1
    -   启用二进制日志：log-bin=自己本地的路径/mysqlbin
    -   启动错误日志：log-err=自己本地路径/mysqlerr
    -   根目录：basedir="自己本地路径"  如：basedir="D：/devSoft/MySQLService5.5/"
    -   临时目录：tmpdir="自己的本地路径" 如：tmpdir="D：/devSoft/MySQLService5.5/"
    -   数据目录：datadir="自己本地路径/Data/"
    -   read-only=0:主机可以进行读写
    -   设置不要复制的数据库：binlog-ignore-db=mysql;
    -   设置需要复制的数据：binlog-do-db=需要复制的数据库名称。
-   修改从机my.cnf配置文件
    -   从服务器唯一id
    -   启用二进制文件
-   因修改过配置文件，请主机+从机都启动后台mysql服务
-   主机从机都关闭防火墙
    -   windows手动关闭
    -   关闭虚拟机linux防火墙service iptables stop
-   在windows主机上建立用户并授权slave
    -   GRANT REPLICATION SLAVE  ON *.* TO 'zhangsan'@'从机器数据库IP‘ IDENTIFIED BY '123456';
    -   flush privileges; 刷新权限
    -   查询master的状态
        -   show master status；
        -   记录下File和Position的值
    -   执行完此步骤后不再执行主服务器MySQL，防止主服务器状态值变化
-   在linux从机上配置需要复制的主机
    -   CHANGE MASTER TO MASTER\_HOST='主机IP',MASTER\_USER='zhangsan'，MASTER\_PASSWORD='123456',MASTER\_LOG\_FILE='File名字'，MASTER\_LOG\_POS=Position数字；
    -
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1626535402927-3a737586-5287-4213-b10b-dc59ad250330.png#height=240\&id=fEMFl\&margin=\[object Object]\&name=image.png\&originHeight=479\&originWidth=968\&originalType=binary\&ratio=1\&size=151719\&status=done\&style=none\&width=484>)
    -   启动从服务器复制功能：start salve；
    -   show slave status\G
        -   下面两个参数都是YES，则说明主从配置成功！
            -   Slave\_IO\_Running:Yes
            -   Slave\_SQL\_Running:Yes
-   主机新建库、新建表、insert记录，从机复制
-   如何停止从服务复制功能
    -   stop slave；
