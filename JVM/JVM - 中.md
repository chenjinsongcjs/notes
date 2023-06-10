# JVM - 中

## 目录

-   [1、String Table](#1String-Table)
    -   [1.1、String的基本特性](#11String的基本特性)
    -   [1.2、String的内存分配](#12String的内存分配)
    -   [1.3、String的基本操作](#13String的基本操作)
    -   [1.4、字符串拼接操作](#14字符串拼接操作)
    -   [1.5、intern()的使用](#15intern的使用)
    -   [1.6、StringTable的垃圾回收](#16StringTable的垃圾回收)
    -   [1.7、G1中的String去重操作](#17G1中的String去重操作)
-   [2、垃圾回收概述](#2垃圾回收概述)
    -   [2.1、什么是垃圾](#21什么是垃圾)
    -   [2.2、为什么需要GC](#22为什么需要GC)
    -   [2.3、早期的垃圾回收](#23早期的垃圾回收)
    -   [2.4、Java垃圾回收机制](#24Java垃圾回收机制)
-   [3、垃圾回收的相关算法](#3垃圾回收的相关算法)
    -   [3.1、垃圾标记阶段算法之引用计数算法](#31垃圾标记阶段算法之引用计数算法)
    -   [3.2、可达性分析算法(根搜索算法、追踪性垃圾收集)](#32可达性分析算法根搜索算法追踪性垃圾收集)
    -   [3.3、对象的finalization机制](#33对象的finalization机制)
    -   [3.4、清除阶段：标记-清除算法](#34清除阶段标记-清除算法)
    -   [3.5、复制算法](#35复制算法)
    -   [3.6、标记-压缩算法(Mark-Compact)](#36标记-压缩算法Mark-Compact)
    -   [3.7、比较三种算法](#37比较三种算法)
    -   [3.8、增量收集算法、分区算法](#38增量收集算法分区算法)
-   [4、垃圾回收相关概念](#4垃圾回收相关概念)
    -   [4.1、System.gc()的理解](#41Systemgc的理解)
    -   [4.2、内存溢出和内存泄漏](#42内存溢出和内存泄漏)
    -   [4.3、Stop the World](#43Stop-the-World)
    -   [4.4、垃圾回收的并行与并发](#44垃圾回收的并行与并发)
    -   [4.5、安全点和安全区域](#45安全点和安全区域)
    -   [4.6、强引用](#46强引用)
    -   [4.7、软引用](#47软引用)
    -   [4.8、弱引用](#48弱引用)
    -   [4.9、虚引用](#49虚引用)
    -   [4.10、终结器引用](#410终结器引用)
-   [5、垃圾回收器](#5垃圾回收器)
    -   [5.1、GC分类与性能指标](#51GC分类与性能指标)
    -   [5.2、不同的垃圾回收器概述](#52不同的垃圾回收器概述)
    -   [5.3、Serial回收器：串行回收](#53Serial回收器串行回收)
    -   [5.4、ParNew回收器：并行回收](#54ParNew回收器并行回收)
    -   [5.5、Parallel回收器：吞吐量优先](#55Parallel回收器吞吐量优先)
    -   [5.6、CMS回收器：低延迟](#56CMS回收器低延迟)
    -   [5.7、G1回收器：区域化分代式](#57G1回收器区域化分代式)
    -   [5.8、垃圾回收器总结](#58垃圾回收器总结)
    -   [5.9、GC日志分析](#59GC日志分析)

## 1、String Table

### 1.1、String的基本特性

-   String：字符串，使用一对""引起来表示
-   String声明为final的，不可被继承
-   String实现Serializable 接口：表示字符串可以支持序列化，实现Comparable接口：表示S提让那个可以比较大小
-   String在JDK8及其以前内部定义了 final char\[] value用于存储字符串数组，JDK9改为byte\[]，为了节省一些空间
-   String代表不可变的字符序列，简称：不可变性。
-   通过字面量的方式给一个字符串赋值，此时的字符串值声明在字符串常量池中。
-   字符串常量池中不会存储相同的字符串。
-   String的String pool是一个大小固定的HashTable，默认大小是1009.如果String pool中的String非常多，就会导致哈希冲突严重，从而导致链表会很长，而链表长了以后直接造成的影响就是当调用String。intern时性能会大幅度下降。
-   使用-XX:StringTableSize设置StringTable的长度
-   在JDK6中StringTable是固定的，就是1009，所以如果常量池中的字符串过多就会导致效率下降很快。StrinTableSize设置没有要求。
-   在JDK7中，StringTable的长度默认为60013，StringTableSize的设置没有要求
-   在JDK8中，设置StringTableSize的长度，1009是可设置的最小值。

### 1.2、String的内存分配

-   8种基本数据类型的常量池是系统协调的。
-   String的常量池可以使用两种方法：直接使用双引号声明的对象会放入常量池中；不是使用双引号声明的String对象，可以调用String的intern()方法。
-   JDk6及其以前，字符串常量池放在永久代中，之后的字符串常量池放在java堆中。

### 1.3、String的基本操作

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622909904518-2abaa7c2-5e17-4e0b-a737-bcb7f92525a5.png#align=left\&display=inline\&height=382\&margin=\[object Object]\&name=image.png\&originHeight=764\&originWidth=1767\&size=487966\&status=done\&style=none\&width=883.5> "image.png")

### 1.4、字符串拼接操作

-   常量与常量拼接结果放在常量池中，原理是编译期优化
-   常量池中不存在相同内容的常量
-   只要其中一个是变量，结果就在堆中。变量拼接的原理是StringBuilder
-   如果拼接的结果调用了intern()方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622910221184-b43ddf6f-0ab4-4288-b661-9b139c8c9a4e.png#align=left\&display=inline\&height=190\&margin=\[object Object]\&name=image.png\&originHeight=379\&originWidth=1113\&size=68034\&status=done\&style=none\&width=556.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622910257999-3e0597c3-4445-411b-b021-9f2722478670.png#align=left\&display=inline\&height=364\&margin=\[object Object]\&name=image.png\&originHeight=727\&originWidth=1001\&size=323304\&status=done\&style=none\&width=500.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622910382531-5092ba84-ccb4-4f79-b097-cd0959e08ac0.png#align=left\&display=inline\&height=345\&margin=\[object Object]\&name=image.png\&originHeight=690\&originWidth=864\&size=244982\&status=done\&style=none\&width=432> "image.png")

-   使用“+”进行字符串拼接，底层创建一个StringBuilder和String对象。

### 1.5、intern()的使用

-   如果不是使用双引号定义的字符串可以使用String提供的intern方法：intern方法会从字符串常量池中查询当前字符串是否存在，如果不存在就会将当前字符串放入常量池中。
-   Interned String就是确保字符串在内存中只有一份拷贝，这样就可以节约内存空间，加快字符串操作任务的执行速度。注意，这个值会被存放在字符串内部池中(String Intern Pool)
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622911000690-71af3c76-1fe9-4eae-8839-ac1761a96e5a.png#align=left\&display=inline\&height=537\&margin=\[object Object]\&name=image.png\&originHeight=1074\&originWidth=1042\&size=391908\&status=done\&style=none\&width=521> "image.png")

-   JDK6及其以前没有进行字符串的优化，JDK7/8中，会对字符串进行优化，堆中存在的对象，字符串常量池中会保存该对象在堆中的地址
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622911351722-c3b13b4e-f993-4ddf-8f2e-8b3a9f986b86.png#align=left\&display=inline\&height=506\&margin=\[object Object]\&name=image.png\&originHeight=1012\&originWidth=1874\&size=659571\&status=done\&style=none\&width=937> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622911373882-c836b4f6-4f8f-4726-8ed3-c61d3ad7a494.png#align=left\&display=inline\&height=404\&margin=\[object Object]\&name=image.png\&originHeight=808\&originWidth=1785\&size=498874\&status=done\&style=none\&width=892.5> "image.png")

-   总结：Intern的使用，JDK6VSJDK7/8
    -   JDK6中，将这个字符串的对象尝试放入常量池中。
        -   如果池中有，就不会放入，返回已有的池中的对象的地址
        -   如果没有，会把此对象复制一份，放入池中，并返回池中的对象的地址。
    -   JDK7以后，将这个字符串的对象尝试放入常量池中。
        -   如果池中有，就不会放入，返回已有的池中的对象的地址
        -   如果没有，则会把对象的引用地址复制一份，放入池中，并返回池中的引用地址。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622911947183-472e87c6-1848-481c-b7fe-cf62bcb643f9.png#align=left\&display=inline\&height=546\&margin=\[object Object]\&name=image.png\&originHeight=1092\&originWidth=1642\&size=620096\&status=done\&style=none\&width=821> "image.png")

### 1.6、StringTable的垃圾回收

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622912069068-b5df0d20-44ad-4b0f-95ff-f9263ac61902.png#align=left\&display=inline\&height=442\&margin=\[object Object]\&name=image.png\&originHeight=884\&originWidth=1511\&size=443137\&status=done\&style=none\&width=755.5> "image.png")

### 1.7、G1中的String去重操作

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622912269133-6405c573-31d9-41f6-8966-d85e5233f9d2.png#align=left\&display=inline\&height=397\&margin=\[object Object]\&name=image.png\&originHeight=793\&originWidth=1809\&size=1091005\&status=done\&style=none\&width=904.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622912384844-6ae1f568-7809-415a-9753-40813013ed01.png#align=left\&display=inline\&height=447\&margin=\[object Object]\&name=image.png\&originHeight=894\&originWidth=1674\&size=1037242\&status=done\&style=none\&width=837> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622912445909-cfb9e7b8-50ff-42d5-b028-8d334bfbc6d2.png#align=left\&display=inline\&height=250\&margin=\[object Object]\&name=image.png\&originHeight=500\&originWidth=1839\&size=444520\&status=done\&style=none\&width=919.5> "image.png")

## 2、垃圾回收概述

### 2.1、什么是垃圾

-   垃圾是指：运行程序中没有任何指针指向的对象。
-   如果不及时对内存中的垃圾进行清理，那么，这些垃圾对象所占的内存空间会一直保留到应用程序结束，被保留的空间无法被其他对象使用，甚至可能导致内存溢出。

### 2.2、为什么需要GC

-   不进行垃圾回收，内存迟早都会被消耗完
-   GC进行碎片整理以便存放大对象，JVM将整理出来的内存分配给新的对象。
-   没有GC就不能保证应用程序的正常进行。

### 2.3、早期的垃圾回收

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622913379761-77196102-9a8f-46ba-b897-cb530e89684c.png#align=left\&display=inline\&height=211\&margin=\[object Object]\&name=image.png\&originHeight=421\&originWidth=897\&size=343534\&status=done\&style=none\&width=448.5> "image.png")

### 2.4、Java垃圾回收机制

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622913579370-26b36c36-c907-442d-93eb-e9096bcd34bc.png#align=left\&display=inline\&height=170\&margin=\[object Object]\&name=image.png\&originHeight=339\&originWidth=849\&size=278658\&status=done\&style=none\&width=424.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622913604596-ed6a341e-e40a-4595-8ffb-24ddb4d1c6a8.png#align=left\&display=inline\&height=367\&margin=\[object Object]\&name=image.png\&originHeight=733\&originWidth=1062\&size=469362\&status=done\&style=none\&width=531> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622913688956-b533d5ef-49b1-4b7f-b956-3cfa58cf6500.png#align=left\&display=inline\&height=313\&margin=\[object Object]\&name=image.png\&originHeight=626\&originWidth=1496\&size=420130\&status=done\&style=none\&width=748> "image.png")

## 3、垃圾回收的相关算法

### 3.1、垃圾标记阶段算法之引用计数算法

-   垃圾标记阶段：进行对象存活判断
-   在进行垃圾回收之前首先需要区分出内存中那些是存活对象，那些是已经死亡的对象。只有被标记为死亡的对象，GC才会在垃圾回收的时候，释放其所占有的内存空间。
-   判断对象存活一般有两种方式：引用计数算法和可达性分析算法
-   引用计数算法，对每个对象保存一个整型的引用计数器属性，用于记录对象被引用的情况。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622914536364-3db41b7b-2578-4816-88a4-df68ae9dbea1.png#align=left\&display=inline\&height=403\&margin=\[object Object]\&name=image.png\&originHeight=805\&originWidth=1580\&size=1029626\&status=done\&style=none\&width=790> "image.png")

### 3.2、可达性分析算法(根搜索算法、追踪性垃圾收集)

-   可达性分析算法有效的解决了引用计数算法中循环引用的问题，防止内存泄漏的发生。
-   所谓的 GC Roots 根集合就是一组必须活跃的引用。
-   基本思路：
    -   可达性分析算法是以根对象集合(GC Roots)为起始点，按照从上到下的方式搜索被根对象所连接的目标对象是否可达。
    -   使用可达性分析算法后，内存中存活的对象都会被根对象集合直接或间接连接着，搜索所走过的路径称为引用链。
    -   如果目标对象没有任何引用链相连，则是不可达的，就意味着该对象已经死亡，可以标记为垃圾对象。
    -   在可达性分析算法中只有直接或间接被根对象连接的对象才是存活对象。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622915603492-6efbe409-b593-4421-bb3d-387f8badc985.png#align=left\&display=inline\&height=493\&margin=\[object Object]\&name=image.png\&originHeight=985\&originWidth=1308\&size=439101\&status=done\&style=none\&width=654> "image.png")

-   GC Roots包括一下几类元素：
    -   虚拟机栈中引用的对象
    -   本地方法栈内JNI引用的对象
    -   方法区中类静态属性引用的对象
    -   方法区中常量引用的对象
    -   所有被同步锁synchronized持有的对象
    -   java虚拟机内部的引用
        -   基本数据类型对应的Class对象，一些常驻对的异常对象，类加载器
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622915970854-9e41355c-00de-4c4b-b9b0-73d2b248ff8e.png#align=left\&display=inline\&height=453\&margin=\[object Object]\&name=image.png\&originHeight=906\&originWidth=1314\&size=1011834\&status=done\&style=none\&width=657> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622916095937-71738dab-fe9b-4061-91ff-bc16f7df0fc5.png#align=left\&display=inline\&height=436\&margin=\[object Object]\&name=image.png\&originHeight=871\&originWidth=1849\&size=1143117\&status=done\&style=none\&width=924.5> "image.png")

-   不在垃圾回收范围内的引用可以作为GC Roots
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622916220370-ef10095f-c108-46b8-a9d6-53da0f418aca.png#align=left\&display=inline\&height=207\&margin=\[object Object]\&name=image.png\&originHeight=414\&originWidth=1701\&size=587596\&status=done\&style=none\&width=850.5> "image.png")

### 3.3、对象的finalization机制

-   java语言提供了对象终止机制来允许开发人员提供对象被销毁之前的自定义处理逻辑。
-   垃圾回收此对象之前，总会先调用这个对象的finalize方法
-   finalize方法允许在子类中被重写，用于在对象被回收时进行资源释放。
-   永远不要主动的调用某个对象的finalize方法，finalize方法可能导致对象复活、影响GC性能、finalize方法若不发生GC则用于不会被调用。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622916958778-ebd42f6a-b5b5-4810-a588-607f5b4577c1.png#align=left\&display=inline\&height=387\&margin=\[object Object]\&name=image.png\&originHeight=773\&originWidth=1712\&size=1116495\&status=done\&style=none\&width=856> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622917101575-a823ba57-6d5f-4b01-8944-e988bb04895e.png#align=left\&display=inline\&height=433\&margin=\[object Object]\&name=image.png\&originHeight=866\&originWidth=1774\&size=1058547\&status=done\&style=none\&width=887> "image.png")

### 3.4、清除阶段：标记-清除算法

-   当堆中的有效内存空间被耗尽的时候就会停止整个应用程序（STW），然后进行两项工作：标记、清除
-   算法执行过程：
    -   标记：Collector从引用根节点开始遍历，标记所有被引用的对象，一般是在对象的Header中记录为可达对象
    -   清除：Collector对堆内存从头到尾进行线性遍历，如果发现某个对象在其Header中没有标记为可达对象，则将其回收。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622918154433-036f08a1-dfb7-4c97-99da-bc03321a34a1.png#align=left\&display=inline\&height=545\&margin=\[object Object]\&name=image.png\&originHeight=1090\&originWidth=1386\&size=392260\&status=done\&style=none\&width=693> "image.png")

-   缺点：
    -   效率不算高
    -   在进行GC的时候需要停止整个应用程序，影响用户体验
    -   会产生内存碎片，需要维护一个空闲列表
-   注意：何为清除：
    -   这里的清除并不是真正的置空，而是把需要清除的对象地址保存在空闲的地址列表里，下次有新对象需要加载时，判断垃圾的位置空间是否够。

### 3.5、复制算法

-   核心思想：
    -   将活着的内存空间分为两块，每次只使用其中一块，在垃圾回收时将正在使用的内存中的存活对象复制到未被使用的内存块中，之后清除正在使用的内存块中的所有对象，交换两个内存的角色，最后完成垃圾回收。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622918696959-31718e5d-1468-4c63-abbe-a6bd40f21842.png#align=left\&display=inline\&height=465\&margin=\[object Object]\&name=image.png\&originHeight=930\&originWidth=1635\&size=382131\&status=done\&style=none\&width=817.5> "image.png")

-   优点：
    -   没有标记和清除过程，实现简单，运行高效
    -   复制过去以后保证空间的连续性，不会出现碎片问题
-   缺点：
    -   需要两倍的内存空间
    -   如果系统中的垃圾独享很少，存活对象很多，效率会变低。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622918977324-1f8191ae-16fb-4917-aa65-862bb4933974.png#align=left\&display=inline\&height=486\&margin=\[object Object]\&name=image.png\&originHeight=971\&originWidth=1851\&size=718151\&status=done\&style=none\&width=925.5> "image.png")

### 3.6、标记-压缩算法(Mark-Compact)

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622919132573-f1bd52ee-eb12-424f-99c7-481d6b31ac29.png#align=left\&display=inline\&height=466\&margin=\[object Object]\&name=image.png\&originHeight=931\&originWidth=1944\&size=671876\&status=done\&style=none\&width=972> "image.png")

-   使用指针碰撞为新对象分配内存空间
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622919284871-4eda3e93-4092-4145-9105-cfa7a998f0f1.png#align=left\&display=inline\&height=124\&margin=\[object Object]\&name=image.png\&originHeight=248\&originWidth=963\&size=193015\&status=done\&style=none\&width=481.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622919336408-280e8c0a-09d8-4e1b-ac7c-dfe54c41f855.png#align=left\&display=inline\&height=220\&margin=\[object Object]\&name=image.png\&originHeight=439\&originWidth=932\&size=245708\&status=done\&style=none\&width=466> "image.png")

### 3.7、比较三种算法

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622919424489-9eb713de-88d5-4cf5-b3ce-f1b81a0a1bc5.png#align=left\&display=inline\&height=111\&margin=\[object Object]\&name=image.png\&originHeight=222\&originWidth=1902\&size=619530\&status=done\&style=none\&width=951> "image.png")

-   ## 分代收集算法
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622919636441-3d796fb7-0790-4560-9e1c-d35c4b86f506.png#align=left\&display=inline\&height=467\&margin=\[object Object]\&name=image.png\&originHeight=933\&originWidth=1905\&size=1089018\&status=done\&style=none\&width=952.5> "image.png")

### 3.8、增量收集算法、分区算法

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622919799918-4fba14c8-ce08-4e06-9f0a-787aae5b7728.png#align=left\&display=inline\&height=467\&margin=\[object Object]\&name=image.png\&originHeight=934\&originWidth=1919\&size=1753715\&status=done\&style=none\&width=959.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622919839188-3cce43ef-202b-4eef-9c2e-a6514d3c301a.png#align=left\&display=inline\&height=174\&margin=\[object Object]\&name=image.png\&originHeight=348\&originWidth=1721\&size=406607\&status=done\&style=none\&width=860.5> "image.png")

-   ## 分区算法
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622919943252-e7253ce9-2a7a-4023-8790-15157353d429.png#align=left\&display=inline\&height=303\&margin=\[object Object]\&name=image.png\&originHeight=605\&originWidth=1769\&size=612462\&status=done\&style=none\&width=884.5> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622919957060-56e1949c-f377-46c2-af67-19253146b78b.png#align=left\&display=inline\&height=448\&margin=\[object Object]\&name=image.png\&originHeight=896\&originWidth=1558\&size=1166763\&status=done\&style=none\&width=779> "image.png")

## 4、垃圾回收相关概念

### 4.1、System.gc()的理解

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622920271442-c699623c-13ad-44d5-8dbf-cc2a39d75b98.png#align=left\&display=inline\&height=324\&margin=\[object Object]\&name=image.png\&originHeight=648\&originWidth=1569\&size=750431\&status=done\&style=none\&width=784.5> "image.png")

### 4.2、内存溢出和内存泄漏

-   javadoc中对OutOfMemoryError的解释是：没有空闲内存，并且垃圾收集器也无法提供更多内存。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622920626477-9b64cd2d-8ff4-40b2-8cf1-908637ee2f58.png#align=left\&display=inline\&height=408\&margin=\[object Object]\&name=image.png\&originHeight=816\&originWidth=1644\&size=1363745\&status=done\&style=none\&width=822> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622920850402-e65cc115-5069-4f83-99ea-9510e3405f1a.png#align=left\&display=inline\&height=294\&margin=\[object Object]\&name=image.png\&originHeight=588\&originWidth=1618\&size=737079\&status=done\&style=none\&width=809> "image.png")

-   ## 内存泄漏（Memory Leak）
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622920961519-6d64db5f-bc51-41c2-9420-231b3a9d4058.png#align=left\&display=inline\&height=374\&margin=\[object Object]\&name=image.png\&originHeight=748\&originWidth=1651\&size=923215\&status=done\&style=none\&width=825.5> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622921073267-49195cf6-961d-42f9-8c80-a4baefcb2bfd.png#align=left\&display=inline\&height=274\&margin=\[object Object]\&name=image.png\&originHeight=548\&originWidth=1499\&size=504418\&status=done\&style=none\&width=749.5> "image.png")

### 4.3、Stop the World

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622921242854-55bfe63a-3c7e-4913-9301-26a6bd248dd0.png#align=left\&display=inline\&height=330\&margin=\[object Object]\&name=image.png\&originHeight=659\&originWidth=1653\&size=935935\&status=done\&style=none\&width=826.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622921294096-738d5eec-9841-4406-84a7-e0f1375c0f69.png#align=left\&display=inline\&height=286\&margin=\[object Object]\&name=image.png\&originHeight=572\&originWidth=1634\&size=569679\&status=done\&style=none\&width=817> "image.png")

### 4.4、垃圾回收的并行与并发

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622921428840-7fe86f84-2cb8-40cb-a217-622e94106f62.png#align=left\&display=inline\&height=305\&margin=\[object Object]\&name=image.png\&originHeight=609\&originWidth=1230\&size=527445\&status=done\&style=none\&width=615> "image.png")
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622921518146-e7e99a4d-6522-4a3c-b41c-dcdad647490f.png#align=left\&display=inline\&height=466\&margin=\[object Object]\&name=image.png\&originHeight=931\&originWidth=1828\&size=792076\&status=done\&style=none\&width=914> "image.png")
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622921581278-c56b675b-af55-4f23-976d-3ddc2d361b71.png#align=left\&display=inline\&height=400\&margin=\[object Object]\&name=image.png\&originHeight=799\&originWidth=1689\&size=638894\&status=done\&style=none\&width=844.5> "image.png")

### 4.5、安全点和安全区域

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622921704047-9fd16e6b-54b3-4eef-a6ac-a811f1a77f84.png#align=left\&display=inline\&height=265\&margin=\[object Object]\&name=image.png\&originHeight=530\&originWidth=1518\&size=849741\&status=done\&style=none\&width=759> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622921918627-9ad7ef94-1e6a-4d0e-bc45-13cbc23e05af.png#align=left\&display=inline\&height=284\&margin=\[object Object]\&name=image.png\&originHeight=568\&originWidth=1698\&size=523390\&status=done\&style=none\&width=849> "image.png")

-   ## 安全区域
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622922143208-cd2e127c-24b8-4048-819c-4c6894344238.png#align=left\&display=inline\&height=321\&margin=\[object Object]\&name=image.png\&originHeight=641\&originWidth=1673\&size=853279\&status=done\&style=none\&width=836.5> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622922162896-224706db-4b4e-4573-aaf8-096de88a902b.png#align=left\&display=inline\&height=315\&margin=\[object Object]\&name=image.png\&originHeight=630\&originWidth=1737\&size=549042\&status=done\&style=none\&width=868.5> "image.png")

### 4.6、强引用

-   强引用：最传统的引用定义：是指在程序代码之中普遍存在的引用赋值，无论任何情况下只要强引用关系还存在，垃圾回收器就永远不会回收掉引用对象。
-   软引用：在系统发生内存溢出之前，将会把这些对象列入回收范围之中进行二次回收，如果这次回收后还没有足够的内存，才会抛出内存溢出异常。
-   弱引用：被弱引用关联的对象只能生存到下一次垃圾回收之前。当垃圾回收器工作时，无论内存空间是否足够，都会回收掉弱引用关联的对象。
-   虚引用：无法通过虚引用来获取一个对象的实例，为一个对象设置虚引用管理的唯一目的就是在这个对象被收集器回收时收到一个系统通知。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623005697456-c49d3a89-9367-4ea9-903a-afb5a9e768b2.png#align=left\&display=inline\&height=200\&margin=\[object Object]\&name=image.png\&originHeight=399\&originWidth=899\&size=178548\&status=done\&style=none\&width=449.5> "image.png")

### 4.7、软引用

-   软引用通常用来实现内存敏感的缓存。比如高速缓存就用到了软引用，如果还有空闲内存，就可以暂时保留缓存，当内存空间不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。

### 4.8、弱引用

-   软引用和弱引用都非常适合来保存那些可有可无的缓存数据。

### 4.9、虚引用

-   为一个对象设置虚引用关联的唯一目的在于跟踪垃圾回收过程。比如：能在这个对象被收集器回收时收到一个系统通知。
-   虚引用不能单独使用。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623006415466-b6984da8-50a6-426d-922a-54898c489e93.png#align=left\&display=inline\&height=250\&margin=\[object Object]\&name=image.png\&originHeight=499\&originWidth=1026\&size=417558\&status=done\&style=none\&width=513> "image.png")

### 4.10、终结器引用

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623006476994-982d70b2-18e4-4ae9-b738-f0257e7b662a.png#align=left\&display=inline\&height=192\&margin=\[object Object]\&name=image.png\&originHeight=383\&originWidth=965\&size=145187\&status=done\&style=none\&width=482.5> "image.png")

## 5、垃圾回收器

### 5.1、GC分类与性能指标

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623006719715-83e1d75f-6eec-48a4-b6e1-2e35682f8525.png#align=left\&display=inline\&height=220\&margin=\[object Object]\&name=image.png\&originHeight=439\&originWidth=753\&size=163049\&status=done\&style=none\&width=376.5> "image.png")

-   串行回收指的是在同一时间段内只允许一个CPU用于执行垃圾回收操作，此时工作线程被暂停，直到垃圾回收工作结束。
    -   串行回收默认被应用在客户端的Client模式下的JVM中，在单CPU、处理器的应用中串行垃圾回收去的性能表现可以超过并行回收器和并发回收器
    -   在并发能力较强的CPU上，并行回收器产生的停顿时间要短于串行回收器
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623007220864-456ca57f-ff67-4885-b593-0c1bb0a39011.png#align=left\&display=inline\&height=237\&margin=\[object Object]\&name=image.png\&originHeight=473\&originWidth=900\&size=232908\&status=done\&style=none\&width=450> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623007326439-041bfa12-b751-4500-8147-59d6caca94fa.png#align=left\&display=inline\&height=112\&margin=\[object Object]\&name=image.png\&originHeight=223\&originWidth=917\&size=147541\&status=done\&style=none\&width=458.5> "image.png")

-   评估GC的性能指标
    -   吞吐量：运行用户代码的时间占总运行时间的比例
    -   暂停时间：执行垃圾收集时，程序的工作线程被暂停的时间。
    -   内存占用：Java堆所占的内存大小。
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623007787657-2fa7dfe1-dbe5-433f-898c-5c5f686a8c8c.png#align=left\&display=inline\&height=245\&margin=\[object Object]\&name=image.png\&originHeight=490\&originWidth=914\&size=226330\&status=done\&style=none\&width=457> "image.png")
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623007828530-f0bd8f29-491d-4a5f-8b38-f9ec15f1c1f1.png#align=left\&display=inline\&height=219\&margin=\[object Object]\&name=image.png\&originHeight=438\&originWidth=931\&size=162514\&status=done\&style=none\&width=465.5> "image.png")
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623007997546-34965eec-6a5d-494e-a801-22b94cd4cc50.png#align=left\&display=inline\&height=232\&margin=\[object Object]\&name=image.png\&originHeight=463\&originWidth=933\&size=419011\&status=done\&style=none\&width=466.5> "image.png")

### 5.2、不同的垃圾回收器概述

-   7款经典的垃圾收集器
    -   串行回收器：Serial、Serial Old
    -   并行回收器：ParNew、Parallel Scavenge 、Parallel OLd
    -   并发回收器：CMS、G1
    
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623008295154-6f8fbeb5-65e1-46b5-93c5-2e7e0893f5c7.png#align=left\&display=inline\&height=416\&margin=\[object Object]\&name=image.png\&originHeight=832\&originWidth=1291\&size=930821\&status=done\&style=none\&width=645.5> "image.png")
-   ## 7款经典的垃圾收集器与垃圾分代之间的关系
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623008404488-c7811b35-743d-4766-a4be-4acb8899d144.png#align=left\&display=inline\&height=440\&margin=\[object Object]\&name=image.png\&originHeight=880\&originWidth=1836\&size=525182\&status=done\&style=none\&width=918> "image.png")
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623008477617-0a8e9a1b-85c4-4e44-ac33-4ee1d5e039ad.png#align=left\&display=inline\&height=389\&margin=\[object Object]\&name=image.png\&originHeight=778\&originWidth=1461\&size=529728\&status=done\&style=none\&width=730.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623008591694-7fc6d4e5-c6c3-411a-9328-13e048936048.png#align=left\&display=inline\&height=349\&margin=\[object Object]\&name=image.png\&originHeight=697\&originWidth=1665\&size=791103\&status=done\&style=none\&width=832.5> "image.png")

### 5.3、Serial回收器：串行回收

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623008943559-d1ff248e-0f50-4141-b03a-961838e292de.png#align=left\&display=inline\&height=428\&margin=\[object Object]\&name=image.png\&originHeight=856\&originWidth=1658\&size=1127862\&status=done\&style=none\&width=829> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623009234605-a7a0cabc-9f76-41b8-81e7-be76ca523490.png#align=left\&display=inline\&height=406\&margin=\[object Object]\&name=image.png\&originHeight=811\&originWidth=1849\&size=846039\&status=done\&style=none\&width=924.5> "image.png")

-   优势：与其他的收集器的单线程相比，简单高效
-   是用 -XX:+UserSerialGC 参数可以指定年轻代和老年代都是用串行收集器。

### 5.4、ParNew回收器：并行回收

-   ParNew回收器是Serial回收器的多线程版本，除了采用并行回收之外，Serial和ParNew基本没有区别。
-   ParNew在年轻代中同样采用复制算法和STW机制
-   ParNew 是很多JVM运行在Server模式下新生代的默认垃圾回收器
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623009874836-a67bb335-8f52-4e28-be5a-371e07147e6e.png#align=left\&display=inline\&height=432\&margin=\[object Object]\&name=image.png\&originHeight=864\&originWidth=1900\&size=722915\&status=done\&style=none\&width=950> "image.png")

-   并不是任何场景下ParNew回收器都比Serial收集器高，在单CPU、处理器的环境下Serial回收器的效率更高，因为没有线程之间的切换。
-   除了Serial回收器之外，目前只有ParNew GC能与CMS收集器配合工作。
-   在程序中，开发人员可以通过选项： -XX:+UserParNewGC手动指定使用ParNew收集器执行内存回收任务。表示年轻年代使用并行收集器，不影响老年代。
-   \-XX:ParallelGCThreads 限制线程数量，默认开启和CPU数据相同的线程数。

### 5.5、Parallel回收器：吞吐量优先

-   Parallel Scanvenge 和ParNew回收器的不同
    -   Parallel Scanvenge的目标是到达一个可控的吞吐量，被称为吞吐量优先的垃圾回收器。
    -   自适应调节策略也是Parallel scanvenge与ParNew的一个重要区别。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623010856500-f941599a-613c-4333-9196-f7908913b633.png#align=left\&display=inline\&height=333\&margin=\[object Object]\&name=image.png\&originHeight=665\&originWidth=1663\&size=954732\&status=done\&style=none\&width=831.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623010960883-fa860f0a-a7f1-4247-a327-c747180fa27b.png#align=left\&display=inline\&height=282\&margin=\[object Object]\&name=image.png\&originHeight=563\&originWidth=1868\&size=480406\&status=done\&style=none\&width=934> "image.png")

-   在JDK8中，是默认的垃圾回收器
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623011099150-c5eeec6c-d4db-432e-bfbc-012e2a6dc276.png#align=left\&display=inline\&height=387\&margin=\[object Object]\&name=image.png\&originHeight=773\&originWidth=1773\&size=981919\&status=done\&style=none\&width=886.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623011150078-82891d54-68bc-4739-acbc-3a2669f5ccb8.png#align=left\&display=inline\&height=402\&margin=\[object Object]\&name=image.png\&originHeight=803\&originWidth=1705\&size=1070142\&status=done\&style=none\&width=852.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623011285812-c2588427-8560-4e07-88f8-baf5abaefa1b.png#align=left\&display=inline\&height=265\&margin=\[object Object]\&name=image.png\&originHeight=529\&originWidth=1717\&size=767345\&status=done\&style=none\&width=858.5> "image.png")

### 5.6、CMS回收器：低延迟

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623011480675-a095417e-9975-41ee-8d00-3f9c37cb3c22.png#align=left\&display=inline\&height=387\&margin=\[object Object]\&name=image.png\&originHeight=774\&originWidth=1727\&size=1341857\&status=done\&style=none\&width=863.5> "image.png")

-   CMS回收器只能与Serial、ParNew配合工作，不能与Parallel Scavenge 配合工作。
-   ## CMS工作原理
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623011695861-6df3b78b-2d71-4c62-830f-ad3e0c93beb5.png#align=left\&display=inline\&height=278\&margin=\[object Object]\&name=image.png\&originHeight=555\&originWidth=1826\&size=477498\&status=done\&style=none\&width=913> "image.png")
-   初始标记阶段：在这个阶段，程序中的所有工作线程都和因为STW机制而出现短暂的暂停，这个阶段的主要任务是仅仅标记出GCRoots能直接关联到的对象。一旦标记完成之后就会恢复之前被暂停的所有应用线程。由于直接关联对象比较小，所以这里的速度非常快。
-   并发标记阶段：从GC Roots的直接关联对象开始遍历整个对象图的过程，这个过个耗时较长但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行。
-   重新标记阶段：由于在并发标记阶段中，程序的工作线程和垃圾收集线程同时运行或交叉运行，因此为了修正标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间通常会比初始阶段稍长一些，但是也远比并发标记阶段的时间段。
-   并发清理阶段：此阶段清理删除标记阶段判断的已经死亡的对象，释放内存空间，由于不用移动存活对象，所以这个阶段也是可以与用户线程同时并发的。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623013110202-8e8a4eae-a4c2-47b8-9d65-ae6eb3b84c3c.png#align=left\&display=inline\&height=195\&margin=\[object Object]\&name=image.png\&originHeight=390\&originWidth=1733\&size=893216\&status=done\&style=none\&width=866.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623013412734-3061de60-058d-4f62-b30d-d2ccb8303e87.png#align=left\&display=inline\&height=440\&margin=\[object Object]\&name=image.png\&originHeight=880\&originWidth=1833\&size=1251723\&status=done\&style=none\&width=916.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623013569805-4ec926c6-bd0b-48a6-abbf-ab249b664540.png#align=left\&display=inline\&height=419\&margin=\[object Object]\&name=image.png\&originHeight=837\&originWidth=1753\&size=1220195\&status=done\&style=none\&width=876.5> "image.png")

-   ## CMS收集器可以设置的参数
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623013800019-11bcb237-e27c-4766-a2fa-56e01ed77dbf.png#align=left\&display=inline\&height=443\&margin=\[object Object]\&name=image.png\&originHeight=886\&originWidth=1822\&size=1227624\&status=done\&style=none\&width=911> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623013826072-0051ddf1-0826-4fcb-b032-d30d7f5346e0.png#align=left\&display=inline\&height=385\&margin=\[object Object]\&name=image.png\&originHeight=769\&originWidth=1709\&size=1057554\&status=done\&style=none\&width=854.5> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623013887824-72ed6ed2-7f76-4fff-a3c7-c44d27fec4ef.png#align=left\&display=inline\&height=177\&margin=\[object Object]\&name=image.png\&originHeight=354\&originWidth=932\&size=165657\&status=done\&style=none\&width=466> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623013919097-d3de4204-8f9b-4562-b270-10b00cd62991.png#align=left\&display=inline\&height=228\&margin=\[object Object]\&name=image.png\&originHeight=456\&originWidth=980\&size=324910\&status=done\&style=none\&width=490> "image.png")

### 5.7、G1回收器：区域化分代式

-   官方给G1设定的目标是在延迟可控的情况下获得尽可能高的吞吐量，所以才担当起全功能收集器的重任与期望。
-   根据优先列表，每次根据允许的收集时间，优先回收价值最大的Region
-   JDK9以后的默认垃圾回收器。
-   G1回收器的特点(优势)
    -   并行与并发
        -   可以多个GC线程同时工作，也可以和用户线程同时执行。
    -   分代收集
        -   将堆空间分为若干个区域，这些区域中包含了逻辑上的年轻代和老年代。
        -
            ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623082454664-ded6c324-f6af-46d6-a6c4-352f0dd9386d.png#align=left\&display=inline\&height=170\&margin=\[object Object]\&name=image.png\&originHeight=340\&originWidth=1001\&size=211087\&status=done\&style=none\&width=500.5> "image.png")
    -   空间整合
        -   G1将内存划分为一个个的region，内存的回收是以region为基本单位的，region之间是复制算法，但整体上实际可看作是标记压缩算法。两种算法都可以避免内存碎片。
    -   可预测的停顿时间模型(软实时)
        -   能让使用者明确在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不超过N毫秒。
-   ## G1回收器的参数设置
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623083244112-d7a34d47-d34a-41cb-85ea-547ab0b8f336.png#align=left\&display=inline\&height=394\&margin=\[object Object]\&name=image.png\&originHeight=788\&originWidth=1837\&size=1145777\&status=done\&style=none\&width=918.5> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623083420142-2fdf7db0-4d13-43cc-8aa5-f5aace707b79.png#align=left\&display=inline\&height=174\&margin=\[object Object]\&name=image.png\&originHeight=348\&originWidth=873\&size=154146\&status=done\&style=none\&width=436.5> "image.png")
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623083914160-d3076714-05fd-45cc-be36-dbab5fb1fc67.png#align=left\&display=inline\&height=158\&margin=\[object Object]\&name=image.png\&originHeight=316\&originWidth=989\&size=254838\&status=done\&style=none\&width=494.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623083924354-059fce02-7640-4cb6-9d70-62b53649df55.png#align=left\&display=inline\&height=255\&margin=\[object Object]\&name=image.png\&originHeight=509\&originWidth=970\&size=330597\&status=done\&style=none\&width=485> "image.png")

-   G1回收器垃圾回收过程
    -   年轻代GC(Young GC)
    -   老年代并发标记过程 (Concurrent Marking)
    -   混合回收（Mixed GC）
    -   如果需要，单线程、独占式、高强度的Full GC还是继续存在的。它针对GC的评估失败提供一种失败保护机制，即强力回收。
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623084285043-42e4fcd9-b0ec-46cb-bcda-2ed465d9969d.png#align=left\&display=inline\&height=234\&margin=\[object Object]\&name=image.png\&originHeight=467\&originWidth=1006\&size=140483\&status=done\&style=none\&width=503> "image.png")
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623084527167-4cc82050-e965-4f4a-ab92-8dc17c364aa5.png#align=left\&display=inline\&height=414\&margin=\[object Object]\&name=image.png\&originHeight=827\&originWidth=1761\&size=1609955\&status=done\&style=none\&width=880.5> "image.png")
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623084768107-f6456900-28b7-41d5-aa8b-ccbf2862a38c.png#align=left\&display=inline\&height=247\&margin=\[object Object]\&name=image.png\&originHeight=494\&originWidth=973\&size=387294\&status=done\&style=none\&width=486.5> "image.png")
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623084907179-e8e9cbcd-f62b-461c-9a40-a568ff1d033f.png#align=left\&display=inline\&height=217\&margin=\[object Object]\&name=image.png\&originHeight=433\&originWidth=1036\&size=164604\&status=done\&style=none\&width=518> "image.png")
-   ## G1回收过程：年轻代GC
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623085047614-2288ebda-9e79-4396-8af9-74f1b286f698.png#align=left\&display=inline\&height=258\&margin=\[object Object]\&name=image.png\&originHeight=515\&originWidth=997\&size=477112\&status=done\&style=none\&width=498.5> "image.png")
-   ## 并发标记过程
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623085077496-e93ff7eb-d95d-4548-b7fa-4de2e50b6e3a.png#align=left\&display=inline\&height=243\&margin=\[object Object]\&name=image.png\&originHeight=486\&originWidth=976\&size=456459\&status=done\&style=none\&width=488> "image.png")
-   ## 混合回收
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623085133434-f00de632-52de-42d8-a2ed-fd91c8e1d3a3.png#align=left\&display=inline\&height=227\&margin=\[object Object]\&name=image.png\&originHeight=454\&originWidth=980\&size=391730\&status=done\&style=none\&width=490> "image.png")
-   ## Full GC
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623085250677-f9e6250e-7f01-4f22-a3a7-daef4b124414.png#align=left\&display=inline\&height=173\&margin=\[object Object]\&name=image.png\&originHeight=346\&originWidth=891\&size=287779\&status=done\&style=none\&width=445.5> "image.png")
-   ## G1回收器优化建议
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623085297250-0a6b13b6-3e92-45ca-82d4-53f818b0adfa.png#align=left\&display=inline\&height=177\&margin=\[object Object]\&name=image.png\&originHeight=353\&originWidth=909\&size=202995\&status=done\&style=none\&width=454.5> "image.png")

### 5.8、垃圾回收器总结

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623085360890-f9ac93a8-2968-4555-afdb-908fcbff690d.png#align=left\&display=inline\&height=248\&margin=\[object Object]\&name=image.png\&originHeight=495\&originWidth=1968\&size=1887569\&status=done\&style=none\&width=984> "image.png")

### 5.9、GC日志分析

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623085542316-ac3bf3f1-cc96-445b-9920-761ad63f13ce.png#align=left\&display=inline\&height=311\&margin=\[object Object]\&name=image.png\&originHeight=622\&originWidth=1603\&size=828137\&status=done\&style=none\&width=801.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623085675016-e740e781-9539-4e00-8eee-535324a427b2.png#align=left\&display=inline\&height=375\&margin=\[object Object]\&name=image.png\&originHeight=750\&originWidth=2020\&size=644119\&status=done\&style=none\&width=1010> "image.png")

-   Minor GC日志
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623085779646-b668a1fe-e1ec-4a75-9529-5174c056faa7.png#align=left\&display=inline\&height=457\&margin=\[object Object]\&name=image.png\&originHeight=914\&originWidth=1971\&size=1108183\&status=done\&style=none\&width=985.5> "image.png")

-   Full GC日志
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1623085843053-38bf69b9-b24b-457d-987e-8f8abca0ab07.png#align=left\&display=inline\&height=479\&margin=\[object Object]\&name=image.png\&originHeight=957\&originWidth=1977\&size=917353\&status=done\&style=none\&width=988.5> "image.png")
