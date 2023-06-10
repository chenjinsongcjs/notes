# JVM - 上

## 目录

-   [虚拟机](#虚拟机)
    -   [java虚拟机](#java虚拟机)
    -   [JVM的位置](#JVM的位置)
    -   [JVM的整体结构(HotSpot VM)](#JVM的整体结构HotSpot-VM)
    -   [JVM的模型架构](#JVM的模型架构)
    -   [JVM的生命周期](#JVM的生命周期)
    -   [JVM的发展历程](#JVM的发展历程)
    -   [类加载子系统](#类加载子系统)
        -   [1、类加载器与类的加载过程](#1类加载器与类的加载过程)
            -   [1.1、类加载子系统作用](#11类加载子系统作用)
            -   [1.2、类加载器的角色](#12类加载器的角色)
            -   [1.3、类加载过程](#13类加载过程)
        -   [2、类加载器的分类](#2类加载器的分类)
        -   [3、双亲委派机制](#3双亲委派机制)
        -   [4、其他内容](#4其他内容)
    -   [运行时数据区](#运行时数据区)
        -   [1.1 、线程](#11-线程)
        -   [1.2、程序计数器(PC寄存器)](#12程序计数器PC寄存器)
        -   [1.3、虚拟机栈](#13虚拟机栈)
            -   [1.3.1 栈的存储单位](#131-栈的存储单位)
            -   [1.3.2、栈帧的内部结构](#132栈帧的内部结构)
            -   [1.3.3、操作数栈](#133操作数栈)
            -   [1.3.4、栈顶缓存技术](#134栈顶缓存技术)
            -   [1.3.5、动态连接(指向运行时常量池的方法引用)](#135动态连接指向运行时常量池的方法引用)
            -   [1.3.6、方法的调用](#136方法的调用)
            -   [1.3.7、方法返回地址](#137方法返回地址)
            -   [1.3.8 本地方法接口](#138-本地方法接口)
            -   [1.3.9 本地方法栈](#139-本地方法栈)
        -   [2、堆](#2堆)
            -   [2.1、堆的核心概述](#21堆的核心概述)
            -   [2.2、设置堆内存大小和OOM](#22设置堆内存大小和OOM)
            -   [2.3、年轻代与老年代](#23年轻代与老年代)
            -   [2.3、图解对象分配过程](#23图解对象分配过程)
            -   [2.5、Minor GC、Major GC、Full GC](#25Minor-GCMajor-GCFull-GC)
            -   [2.6、堆空间分代思想](#26堆空间分代思想)
            -   [2.7、内存分配策略](#27内存分配策略)
            -   [2.8、为对象分配内存：TLAB](#28为对象分配内存TLAB)
            -   [2.9、堆空间参数设置](#29堆空间参数设置)
            -   [2.10、堆是分配对象存储的唯一选择吗？NO](#210堆是分配对象存储的唯一选择吗NO)
        -   [3、方法区](#3方法区)
            -   [3.1、堆、栈、方法区的交互关系](#31堆栈方法区的交互关系)
            -   [3.2、方法区的理解](#32方法区的理解)
            -   [3.3、设置方法区大小与OOM](#33设置方法区大小与OOM)
            -   [3.4、方法区的内部结构](#34方法区的内部结构)
            -   [2.5、方法区使用举例](#25方法区使用举例)
            -   [2.6、方法区的演进细节](#26方法区的演进细节)
            -   [3.7、方法区的垃圾回收](#37方法区的垃圾回收)
        -   [4、对象的实例化、内存布局与访问定位](#4对象的实例化内存布局与访问定位)
            -   [4.1、对象的实例化](#41对象的实例化)
            -   [4.2、对象的内存布局](#42对象的内存布局)
            -   [4.3、对象的访问定位](#43对象的访问定位)
        -   [5、直接内存](#5直接内存)
        -   [6、执行引擎](#6执行引擎)
            -   [6.1、执行引擎概述](#61执行引擎概述)
            -   [java代码编译和执行的过程](#java代码编译和执行的过程)
            -   [6.2、解释器](#62解释器)
            -   [6.3、JIT编译器](#63JIT编译器)

# 虚拟机

-   所谓虚拟机，就是一台虚拟计算机，它是一款软件，用来执行一系列虚拟计算机指令，大体上虚拟机可以分为系统虚拟机和程序虚拟机。
-   系统虚拟机是物理计算机的完全仿真，比如Visual Box等
-   程序虚拟机是专门为执行单个计算机程序而设计，比如Java虚拟机

## java虚拟机

-   java虚拟机是一台执行java字节码的虚拟计算机，他有独立的运行机制。
-   java技术的核心就是java虚拟机。。
-   Java虚拟机就是二进制字节码的运行环境，负责装载字节码到其内部，解释编译为对应平台的机器指令执行。
-   Java虚拟机的特点：
    -   一次编译，到处运行
    -   自动内存管理
    -   自动垃圾回收功能

## JVM的位置

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621913180179-7e69b49a-12b2-4459-9c0c-a581c70d624d.png#align=left\&display=inline\&height=255\&margin=\[object Object]\&name=image.png\&originHeight=1019\&originWidth=1639\&size=1554386\&status=done\&style=none\&width=410> "image.png")

## JVM的整体结构(HotSpot VM)

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621913267582-e783cf49-4c07-49af-b2e2-27dc2c2608ff.png#align=left\&display=inline\&height=246\&margin=\[object Object]\&name=image.png\&originHeight=985\&originWidth=1083\&size=648327\&status=done\&style=none\&width=271> "image.png")

## JVM的模型架构

-   java编译器输入的指令流基本上是一种基于栈的指令集架构，另一种指令集架构则是基于寄存器的指令集架构。
-   基于栈的指令集架构的特点：
    -   设计和实现更简单，适用于资源受限的系统
    -   避开了寄存器的分配难题，使用零地址指令方式分配
    -   零地址指令，执行过程依赖于操作栈，指令集更小，编译器容易实现
    -   不需要硬件支持，可移植性好，更好实现跨平台。
-   基于寄存器架构的特点：
    -   指令集完全依赖于硬件，可移植性差
    -   性能优秀和执行高效
    -   完成一项操作花费更少的指令
    -   指令集往往都以一地址指令，二地址指令，三地址指令为主
    -   传统PC和android的Davlik虚拟机使用这种架构。

## JVM的生命周期

-   虚拟机的启动
    -   Java虚拟机的启动是通过引导类加载器创建一个初始化类来完成的
-   虚拟机的执行
    -   执行java程序。
    -   程序开始虚拟机开始执行，程序结束虚拟机停止执行
    -   执行一个所谓的java程序的时候真真正正执行的是一个叫做java虚拟机的进程
-   虚拟机的退出
    -   程序正常结束
    -   程序在执行过程中遇到异常或者错误终止
    -   操作系统出现错误
    -   某线程调用Runtime类或System类的exit方法等等

## JVM的发展历程

-   世界上第一款商用Java虚拟机：Sun Classic VM，JDK1.4淘汰，内部只提供解释器，hotspot内置了此虚拟机
-   Exact VM ：具备现代高性能虚拟机的雏形，热点探测，编译器与解释器混合工作，但是被hotspot替代。
-   HotSpot VM：Oracle JDK和OpenJDK的默认虚拟机
-   BEA 的JRockit ：专注于服务端应用，JRockit内部不包含解释器，是世界上最快的JVM
-   IBM的J9：广泛用于IBM的各种Java产品
-   其他：KVM和CDC/CLDC HotSpot、Azul VM、Liquid VM、Apache Harmony、Microsoft JVM、TaobaoJVM。

## 类加载子系统

### 1、类加载器与类的加载过程

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621915462959-eda585d2-186d-4778-b7f6-918ae040546c.png#align=left\&display=inline\&height=286\&margin=\[object Object]\&name=image.png\&originHeight=572\&originWidth=680\&size=104849\&status=done\&style=none\&width=340> "image.png")

#### 1.1、类加载子系统作用

-   类加载子系统负责从文件系统或者网络中加载Class文件，class文件在文件开头有特定的文件标识：CAFE BABE
-   ClassLoader只负责class文件的加载，至于是否可以运行，由执行引擎决定
-   加载的类信息存放在方法区中

#### 1.2、类加载器的角色

-   class file 加载到JVM中，被称为DNA元数据模板，放在方法区
-   在class文件->JVM->最终成为元数据模板，此过程就要一个运输工具：类加载器，扮演一个快递员的角色。

#### 1.3、类加载过程

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621916298177-ce900f66-24dd-49a1-a413-68007ef46d5b.png#align=left\&display=inline\&height=121\&margin=\[object Object]\&name=image.png\&originHeight=485\&originWidth=1483\&size=956471\&status=done\&style=none\&width=371> "image.png")

-   加载
    -   通过一个类的全限定类名获取定义此类的二进制字节流
    -   将这个字节流所代表的静态存储结构转换为方法区的运行时数据结构
    -   在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。
-   链接
    -   验证(Verify)
        -   目的：确保class文件的字节流中包含信息符合当前虚拟机的要求保证被加载类的正确性，不会危害虚拟机自身的安全。
        -   主要包括四种验证：文件格式验证、元数据验证、字节码验证、符号引用验证。
    -   准备(Prepare)
        -   为类变量(静态变量)分配内存并且设置类变量的默认初始值，即零值
        -   这里不包含final修饰的static，因为final在编译的时候就会分配，准备阶段会显示初始化。
        -   这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量会随着对象一起分配到java堆中。
    -   解析(Resolve)
        -   将常量池中的符号引用转换为直接引用的过程
        -   解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等等。
-   初始化
    -   初始化阶段就是执行类构造器方法()的过程
    -   此方法不需定义，是javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来。
    -   构造器方法中指令按语句在源文件中出现的顺序执行
    -   ()不同于类的构造器(构造器是虚拟机视角下的（）)
    -   若该类具有父类，JVM会保证子类的()执行前，父类的()已经执行完毕
    -   虚拟机必须保证一个类的()方法在多线程下被同步加锁。

### 2、类加载器的分类

-   JVM支持两种类型的类加载器，分别为引导类加载器和自定义类加载器(包含扩展类加载器和系统类加载器)
-   将所有派生与抽象类ClassLoader的类加载器划分为自定义类加载器
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621918541559-26113a5e-1e9c-47cb-abba-a7dab54e5d45.png#align=left\&display=inline\&height=231\&margin=\[object Object]\&name=image.png\&originHeight=924\&originWidth=1661\&size=558012\&status=done\&style=none\&width=415> "image.png")

-   引导类加载器
    -   使用c/c++语言实现，嵌套在JVM内部
    -   用来加载java的核心类库：JAVA\_HOME/jre/lib/rt.jar、resources.jar、sun.boot.class.path路径下的内容。
    -   不继承自java.lang.ClassLoader，没有父加载器
    -   加载扩展类加载器和系统类加载器，并指定为他们的父类加载器
    -   只加载java、javax、sun等开头的类
-   扩展类加载器
    -   java语言编写，派生于ClassLoader类
    -   父类加载器为引导类加载器
    -   加载java.ext..dirs系统属性所指定的目录中的类库，或者jre/lib/ext子目录下的类库。
-   系统类加载器
    -   java语言编写，派生于ClassLoader类
    -   父类加载器为引导类加载器
    -   负责加载classPath合作java.class.path指定路径下的类库
    -   是程序中的默认加载器
    -   通过：ClassLoader.getSystemClassLoader()方法可以获得该类加载器
-   用户自定义加载器
    -   继承于ClassLoader或者URLClassLoader即可
-   ## 关于CLassLoader
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621919649866-86970bdd-f3b2-454d-98b1-6656f352505d.png#align=left\&display=inline\&height=195\&margin=\[object Object]\&name=image.png\&originHeight=778\&originWidth=1740\&size=911179\&status=done\&style=none\&width=435> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621919762500-8bbaffd3-3948-4d4f-a5af-53ff17951ecc.png#align=left\&display=inline\&height=222\&margin=\[object Object]\&name=image.png\&originHeight=889\&originWidth=1108\&size=1123072\&status=done\&style=none\&width=277> "image.png")

### 3、双亲委派机制

-   java虚拟机对class文件采用的是按需加载的方式。
-   加载某个类的class文件时，java虚拟机采用的是双亲委派机制，即把请求交由父类处理，它是一种任务委派模式。
-   双亲委派机制的工作原理
    -   如果一个类加载器收到了类加载请求，他并不会自己去加载，而是把这个请求委托给父类加载器去执行。
    -   如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的引导类加载器。
    -   如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子类加载器才会尝试自己去加载，这就是双亲委派模式。
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621920537163-7da383a4-153f-4ad5-a7a4-a670c60784d9.png#align=left\&display=inline\&height=194\&margin=\[object Object]\&name=image.png\&originHeight=774\&originWidth=952\&size=844847\&status=done\&style=none\&width=238> "image.png")
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621920923141-14a2ad3f-2bf0-48a3-9630-f64a4fb9d471.png#align=left\&display=inline\&height=253\&margin=\[object Object]\&name=image.png\&originHeight=1012\&originWidth=1445\&size=2117220\&status=done\&style=none\&width=361> "image.png")
-   双亲委派机制的优势
    -   避免类的重复加载
    -   保护程序安全，放置核心API被篡改
-   沙箱安全机制
    -   核心类库交由引导类加载器加载，自定义核心类库不被允许，保证了核心类源码的安全性。

### 4、其他内容

-   在JVM中表示两个对象是否为同一个类存在两个条件
    -   类的完整类名必须一致，包括包名
    -   加载这个类的ClassLoader(指的是ClassLoader的实例对象)必须相同
-   JVM必须知道一类型是由引导类加载器加载的还是由用户类加载器加载的，如果一个类型是由用户类加载器加载的，那么JVM会将这个类加载器的引用作为类型信息的一部分保存在方法区中，当解析一个类型到另一个类型的引用的时候，JVM需要保证这两个类型的类加载器是相同的。
-   类的主动使用和被动使用
    -   主动使用
        -   创建类的实例
        -   访问某个类的或接口的静态变量，或者多该静态变量赋值
        -   调用类的静态方法
        -   反射
        -   初始化一个类的子类
        -   jva虚拟机启动时被标明为启动类的类
        -   JDK7 开始提供的动态语言支持
    -   除了以上七种情况，其他使用java类的方式被看做类的被动使用，都不会导致类的初始化。

## 运行时数据区

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621922174212-2126484a-3fe9-4e89-aff0-8095e06332c5.png#align=left\&display=inline\&height=453\&margin=\[object Object]\&name=image.png\&originHeight=906\&originWidth=1637\&size=550739\&status=done\&style=none\&width=819> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621922302598-d1178ffb-b1e2-417d-826e-74116b879992.png#align=left\&display=inline\&height=215\&margin=\[object Object]\&name=image.png\&originHeight=859\&originWidth=1901\&size=2575155\&status=done\&style=none\&width=475> "image.png")

-   每个线程独有：程序计数器、虚拟机栈、本地方法栈
-   线程间共享的：堆、堆外内存(永久代或元空间、代码缓存)

### 1.1 、线程

-   线程是一个程序里的运行单元。JVM允许一个应用有多个线程并行的执行。
-   在HotSpot JVM中，每个线程都与操作系统的本地线程直接映射
-   操作系统负责所有线程的安排调度到任何一个可用的CPU上。一旦本地线程初始化成功，它就会代用Java线程中的run方法。
-   JVM后台系统线程
    -   虚拟机线程
    -   周期任务线程
    -   GC线程
    -   编译线程
    -   信号调度线程

### 1.2、程序计数器(PC寄存器)

-   JVM中的PC寄存器是对物理PC寄存器的一种抽象模拟
-   PC寄存器的作用
    -   PC寄存器用来存储指向下一条指令的地址，也即将要执行的指令得代码。由执行引擎读取下一条指令。
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621923612830-a8bea54a-0582-49df-9ec4-5ec79e3f3392.png#align=left\&display=inline\&height=227\&margin=\[object Object]\&name=image.png\&originHeight=908\&originWidth=1088\&size=497906\&status=done\&style=none\&width=272> "image.png")
-   PC寄存器是一块很小的内存空间，也是运行速度最快的存储区域
-   PC寄存器是线程独有的，生命周期与线程的生命周期一致
-   任何时间一个线程都只有一个方法在执行，即当前方法，PC寄存器会存储当前线程正在执行的Java方法的JVM指令地址，或者如果是执行native方法，则是未指定值undefined
-   PC寄存器是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621924387455-2691953f-3e7c-4efe-8a1e-6ea8f1d0533e.png#align=left\&display=inline\&height=247\&margin=\[object Object]\&name=image.png\&originHeight=987\&originWidth=1933\&size=796481\&status=done\&style=none\&width=483> "image.png")

-   使用PC寄存器存储字节码指令地址有什么用？
    -   因为CPU需要不停的切换各个线程，这个时候切换回来以后，就得知道接着从哪开始继续执行
    -   JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令。
-   ## PC寄存器为什么会被设定为线程私有？
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1621924899155-c7093a14-3874-468c-8f71-2bd337ee1b8c.png#align=left\&display=inline\&height=185\&margin=\[object Object]\&name=image.png\&originHeight=738\&originWidth=1730\&size=1001915\&status=done\&style=none\&width=433> "image.png")

### 1.3、虚拟机栈

-   栈是运行时的单位，堆是存储单位
    -   栈解决程序的运行问题，程序如何执行，或者如何处理数据。
    -   堆解决的是数据存储问题，数据怎么放，放在那儿。
-   Java虚拟机栈是线程私有的，与线程的生命周期一致，其内部保存一个个的栈帧，对应一次次的java方法调用。
-   JVM直接对java栈的操作只有两个：
    -   每个方法执行，伴随着压栈
    -   方法执行结束后出栈
-   对于栈来说不存在垃圾回收的问题
-   栈中可能出现的异常：java栈的大小是动态的或者是固定不变的。
    -   如果采用固定大小的Java虚拟机栈，可能会抛出一个StackOverflowError异常
    -   如果java虚拟机栈可以动态扩展，无法申请到足够的内存，java虚拟机将会抛出一个OutOfMemoryError异常。
-   设置栈内存大小
    -   我们可以采用-Xss 选项来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度。

#### 1.3.1 栈的存储单位

-   每个线程都有自己的栈，栈中的数据都是以栈帧的格式存在，线程上每个正在执行的方法都对应这一个栈帧，栈帧是一个内存区块，是一个数据集。
-   在一条活动的线程中，一个时间点上只有一个活动的栈帧，即只有当前正在执行的方法的栈帧（栈顶栈帧）是有效的，这个栈帧被称为当前栈帧，与当前栈帧相对应的方法就是当前方法，定义这个方法的类就是当前类。
-   执行引擎运行的所有字节码指令只针对当前栈帧进行操作。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622003869604-904629f2-9357-4f4c-a088-0ee538a2d0cb.png#align=left\&display=inline\&height=243\&margin=\[object Object]\&name=image.png\&originHeight=485\&originWidth=923\&size=118968\&status=done\&style=none\&width=461.5> "image.png")

-   不同线程中所包含的栈帧是不能相互引用的。
-   如果前一个栈帧调用了当前栈帧，那么会在当前栈帧结束的时候将返回值传递给前一个栈帧。
-   Java方法中有两种返回函数的方式：
    -   一种是正常的函数返回，使用哪个return指令；
    -   另一种是抛出异常，不管是哪一种方式，都会让当前栈帧出栈

#### 1.3.2、栈帧的内部结构

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622004382055-4fe4e538-b75b-477b-84c1-5412ea6b2906.png#align=left\&display=inline\&height=372\&margin=\[object Object]\&name=image.png\&originHeight=744\&originWidth=2004\&size=472531\&status=done\&style=none\&width=1002> "image.png")

-   局部变量表
    -   局部变量表也被称为局部变量数组或本地变量表
    -   定义为一个数字数组，主要用于存储方法参数和定义在方法体内的局部变量，这些数据类型主要包括各类基本数据类型、对象应用以及returnAddress类型。
    -   局部变量表是建立在线程栈上的，是线程私有的，不存在线程安全问题。
    -   局部变量表所需的容量大小在编译期就确定下来的，并保存在方法的Code属性的maximum local variable数据项中，在方法运行期间是不会改变局部变量表的大小的。
    -   方法嵌套调用的次数由栈的大小决定，一般来说，栈越大，方法嵌套调用的次数越多，对一个函数而言，它的参数和局部变量越多，使得局部变量表膨胀，它的栈帧就越大，以满足方法调用所需传递的信息增大的需求，进而函数调用就会占用更多的栈空间，导致其嵌套调用次数就会减少。
    -   局部变量表中的变量只在当前方法调用中有效，在方法执行时，虚拟机通过使用局部变量表完成参数值到参数变量列表的传递过程，当方法调用结束后，随着方法栈帧销毁，局部变量表也会随之销毁。
    -   局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收。
-   关于Slot的理解
    -   局部变量表最基本的存储单位是Slot变量槽
    -   在局部变量表中，32位以内的类型只占用一个Slot（包括returnAddress），64位的类型（long和double）占用两个Slot
        -   byte、short、char、boolean在存储之前会被转换为int
    -   JVM会为局部变量表中的Slot分配一个访问索引，通过这个索引即可成功访问到局部变量表中指定的局部变量值。
    -   当一个实例方法被调用的时候后，它的方法参数和方法体内定义的局部变量会按照顺序赋值到局部变量表的每一个slot上。
    -   如果需要访问局部变量表中的一个64位的局部变量，只需要使用第一个索引即可。（64位数据类型占用的两个slot中的第一个slot索引）。
    -   如果当前帧是由构造方法或者实例方法创建的那么该对象引用this将会存放在index为0的slot处，其余的参数按照参数顺序继续排列。
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622006389719-0365ca37-23ee-40e3-bb79-cac4f624bd60.png#align=left\&display=inline\&height=231\&margin=\[object Object]\&name=image.png\&originHeight=462\&originWidth=396\&size=76867\&status=done\&style=none\&width=198> "image.png")
    -   栈帧中局部变量表中的槽位是可以重用的，如果一个局部变量过了其作用域，那么在其作用域之后声明的新的局部变量就有可能会复用过期局部变量的槽位，从而达到节省资源的目的。
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622006697615-eac3ea44-cca3-4841-b380-38e20ea2f3bb.png#align=left\&display=inline\&height=125\&margin=\[object Object]\&name=image.png\&originHeight=250\&originWidth=614\&size=86488\&status=done\&style=none\&width=307> "image.png")
-   静态变量和局部变量的对比
    -   类变量有两次初始化的机会
        -   第一次在链接的准备阶段进行分配内存空间默认初始化
        -   第二次在初始化阶段将程序员定义的值赋予类变量
    -   和类变量不同的是局部变量没有系统初始化的过程，所以局部变量必须进行人为初始化否则不能使用。

#### 1.3.3、操作数栈

-   操作数栈也称为表达式栈
-   操作数栈，在方法执行过程中，根据字节码指令，往栈中写入数据或提取数据，即入栈/出栈
    -   某些字节码指令将值压入操作数栈，其余的字节码指令将操作数取出栈，使用他们之后再把结果压入栈。

        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622007703800-06ba0de3-b6cc-4697-8b45-095717ddf7fb.png#align=left\&display=inline\&height=87\&margin=\[object Object]\&name=image.png\&originHeight=173\&originWidth=608\&size=45577\&status=done\&style=none\&width=304> "image.png")

        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622007863863-dc665770-ea6d-40ff-aa6a-93bfbc77213f.png#align=left\&display=inline\&height=222\&margin=\[object Object]\&name=image.png\&originHeight=443\&originWidth=1197\&size=268673\&status=done\&style=none\&width=599> "image.png")
    -   操作数栈，主要用来保存计算过程的中间结果，同时作为计算过程中变量临时的存储空间。
    -   操作数栈就是JVM执行引擎的一个工作区，当一个方法公开是执行的时候，一个新的栈帧也会随之被创建出来，之歌方法的操作数栈是空的。
    -   每一个操作数栈都会有一个明确的栈深度用于存储数值，其所需的最大深度在编译期就定义好了，保存在Code属性中，为max\_stack的值。
    -   栈中的任何一个元素都可以是任意的java数据类型
        -   32位类型占用一个栈单位深度
        -   64位类型占用两个栈单位深度
    -   操作数栈并非采用访问索引的方式进行数据访问的，而是只能通过标准的入栈和出栈操作来完成一次数据访问。
    -   如果被调用的方法带有返回值的话，其返回值会被压入当前栈帧的操作数栈中，并更新PC寄存器的下一条需要执行的字节码指令。
    -   Java虚拟机的解释引擎是基于栈的执行引擎，其中的栈指的是操作数栈。

#### 1.3.4、栈顶缓存技术

-   将栈顶元素全部缓存在物理CPU的寄存器中，以此降低对内存的读/写次数，提升执行引擎的执行效率。

#### 1.3.5、动态连接(指向运行时常量池的方法引用)

-   动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用。

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622009771746-7c5bdc2b-6a1a-4788-8cd3-d6c2d1448eb8.png#align=left\&display=inline\&height=264\&margin=\[object Object]\&name=image.png\&originHeight=528\&originWidth=1149\&size=237834\&status=done\&style=none\&width=574.5> "image.png")

-   常量池的作用，就是为了提供一些符号和常量，便于指令的识别。

#### 1.3.6、方法的调用

-   静态连接：当一个字节码文件被装载进JVM内部时，如果被调用的目标方法在编译期可知，且运行期间保持不变时，这种情况下将调用方法的符号引用转换为直接引用的过程称之为静态连接。对应的绑定机制为：早期绑定
-   动态连接：如果被调用的方法在编译期无法被确定下来，也就是说，只能够在程序运行期间调用方法的符号引用转换为直接引用，由于这种引用转换过程具备动态性，因此也就被称为动态链接。对应的绑定机制为：晚期绑定
-   非虚方法
    -   如果方法在编译期就确定了具体的调用版本，这个版本在运行时是不可变的，这样的方法称为非虚方法。
    -   静态方法、私有方法、final方法、实例构造器、父类方法都是非虚方法。其他方法都是虚方法
-   普通调用指令：
    -   invokestatic：调用静态方法，解析阶段确定唯一方法版本
    -   invokespecial：调用方法、私有方法、父类方法、解析阶段确定唯一方法版本  ，，，以上两种为非虚方法调用
    -   invokevirtual：调用所有虚方法
    -   invokeinterface：调用接口方法
-   动态调用指令
    -   invokedynamic：动态解析出需要调用的方法 （JDK7中出现，JDK8中出现Lambda表达式，能直接生成invokedynamic）
-   方法重写的本质

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622011354985-633be324-d12e-4411-a54e-952631954d71.png#align=left\&display=inline\&height=267\&margin=\[object Object]\&name=image.png\&originHeight=534\&originWidth=1112\&size=424104\&status=done\&style=none\&width=556> "image.png")

-   虚方法表
    -   每一个类中都有虚方法表，表中存放着各个方法的实际入口。
    -   虚方法表会在类加载的链接阶段被创建并开始初始化，类的变量初始值准备完成之后，JVM会把该类的方法表也初始化完毕

#### 1.3.7、方法返回地址

-   方法正常退出，调用者的pc寄存器的值作为返回地址，异常退出时，返回地址要通过异常表来确定，栈帧中一般不会保存这一部分信息。
-   正常完成出口和异常完成出口的区别在于：通过异常完成出口退出的不会给它的上层调用者产生任何的返回值。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622012120749-9c6ea039-d77b-4606-b5a1-36e7621261ef.png#align=left\&display=inline\&height=216\&margin=\[object Object]\&name=image.png\&originHeight=431\&originWidth=1058\&size=329281\&status=done\&style=none\&width=529> "image.png")
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622012170768-a0031245-fac2-47d8-90c3-cf9ac73d9f55.png#align=left\&display=inline\&height=107\&margin=\[object Object]\&name=image.png\&originHeight=214\&originWidth=1023\&size=218157\&status=done\&style=none\&width=511.5> "image.png")

#### 1.3.8 本地方法接口

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622737940115-8c9b7f05-5a37-416d-a07f-562f7db732a1.png#align=left\&display=inline\&height=290\&margin=\[object Object]\&name=image.png\&originHeight=579\&originWidth=921\&size=337766\&status=done\&style=none\&width=460.5> "image.png")

-   本地方法就是一个 Native Method，就是java调用非java代码的接口
-   标识符 native可以和其他的java标识符连用，除了abstract
-   为什么要使用 native method
    -   与java环境外交互
    -   与操作系统交互

#### 1.3.9 本地方法栈

-   java的虚拟机栈用于管理java方法的调用，而本地方法栈用于管理本地方法的调用。
-   本地方法栈是线程私有的，这个栈可以是固定大小的也可以是动态扩展的。
-   本地方法栈不存在GC，但是可能存在StackOverflowError和OutOfMemoryError（动态扩展是无法分配内存）
-   当某个线程调用一个本地方法时，它就进入了一个全新的并且不受虚拟机限制的世界，它和虚拟机拥有相同的权限
    -   可以通过本地方法调用虚拟机的运行时数据区
    -   可以直接使用本地处理器的寄存器
    -   可以直接从本地内存的堆中分配任意数量的内存
-   并不是所有的java虚拟机都有本地方法，因为java虚拟机规范中没有明确的要求
-   在HotSpot中将本地方法接口与本地方法栈合二为一

### 2、堆

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622739328688-468e20b0-99f3-4ad0-aa6d-f084a1499bf7.png#align=left\&display=inline\&height=299\&margin=\[object Object]\&name=image.png\&originHeight=598\&originWidth=1237\&size=265008\&status=done\&style=none\&width=618.5> "image.png")

#### 2.1、堆的核心概述

-   一个JVM实例中只存在一个堆内存。
-   java堆在JVM启动的时候被创建，其空间大小也就确定了，是JVM管理的最大的一块内存空间。堆的大小是可以修改的
-   堆可以处于物理上不连续的内存空间中，但是逻辑上他应该被视为连续的
-   所有的java线程共享java堆，堆上还可以分配线程私有的缓冲区(TLAB)
-   几乎所有的对象和数组都应当在运行时分配在堆上，逃逸分析例外。
-   堆是GC执行垃圾回收的重点区域
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622739812107-b753e369-804d-49ec-80b2-c9597f404bd4.png#align=left\&display=inline\&height=175\&margin=\[object Object]\&name=image.png\&originHeight=350\&originWidth=729\&size=118211\&status=done\&style=none\&width=364.5> "image.png")

-   ## 堆内存细分
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622739882194-eb966114-6eb8-4fc7-925a-1f8274bd7f3e.png#align=left\&display=inline\&height=221\&margin=\[object Object]\&name=image.png\&originHeight=442\&originWidth=882\&size=363075\&status=done\&style=none\&width=441> "image.png")
    -   JDK的堆内存分配
    
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622740050968-e2bda282-fcb6-4eba-be7b-c676e1e049b9.png#align=left\&display=inline\&height=480\&margin=\[object Object]\&name=image.png\&originHeight=959\&originWidth=1367\&size=710458\&status=done\&style=none\&width=683.5> "image.png")
    -   JDK8的堆内存分配
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622740113874-7f949efe-fcef-4400-9f2f-fa87fb1a2cba.png#align=left\&display=inline\&height=445\&margin=\[object Object]\&name=image.png\&originHeight=889\&originWidth=1870\&size=601573\&status=done\&style=none\&width=935> "image.png")

#### 2.2、设置堆内存大小和OOM

-   \-Xms ：用于设置堆区的起始内存，等价于：-XX:InitialHeapSize
-   \-Xmx ：用于设置堆区的最大内存，等价于：-XX:MaxHeapSize
-   一旦堆区中的内存大小超过堆的最大内存，将会抛出OutOfMemoryError。
-   通常将 -Xms 和-Xmx两个参数配置相同的值，目的是为了能够在java垃圾回收机制起立完堆区后不需要重新分隔计算堆区的大小，从而提高性能。
-   默认情况下：初始内存大小：物理内存/64，，，最大内存大小：物理内存  /4。

#### 2.3、年轻代与老年代

-   存储在JVM中的jjava对象可以被划分为两类
    -   一类是声明周期较短的瞬时对象，此类对象的创建和销毁非常迅速
    -   另一类对象的生命周期很长，极端情况下甚至和JVM的生命周期一致。

    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622740927952-bc9853c2-b638-4435-8187-7737eafc03ec.png#align=left\&display=inline\&height=244\&margin=\[object Object]\&name=image.png\&originHeight=488\&originWidth=910\&size=266302\&status=done\&style=none\&width=455> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622741008929-7859427e-4b83-4238-bb3a-f6b280c98d50.png#align=left\&display=inline\&height=160\&margin=\[object Object]\&name=image.png\&originHeight=320\&originWidth=1648\&size=272388\&status=done\&style=none\&width=824> "image.png")



    -   配置新生代和老年代在堆中的占比
        -   默认-XX:NewRatio=2,表示新生代占1，老年代占2，新生代占整个堆的1/3.
    -   新生区中Eden区和Survivor区的默认占比为8:1:1,可以通过参数 -XX:SurvivorRatio=?进行设置。
    -   几乎所有的对象都是在Eden区中被new出来的，除了不能存放的大对象，放入老年区外。
    -   可以使用参数 -Xmn 设置新生代最大内存大小，但是这个参数一般不会修改。
    -
![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622741457905-f5b47885-a251-4f26-b4e5-2716259b2f0a.png#align=left\&display=inline\&height=254\&margin=\[object Object]\&name=image.png\&originHeight=508\&originWidth=1693\&size=291957\&status=done\&style=none\&width=846.5> "image.png")
        -   当Eden区的内存空间被占满时会触发一次Minor GC对新生区的内存空间进行一个垃圾回收，存活的对象会被移入Survivor区(to区)，经过多次垃圾回收之后，Survivor区的对象的年龄计数器达到一定的阈值此对象就会进入老年区。

#### 2.3、图解对象分配过程

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622741837203-3b03d41d-58a5-45ee-9634-3d27c352cbff.png#align=left\&display=inline\&height=294\&margin=\[object Object]\&name=image.png\&originHeight=587\&originWidth=1723\&size=846800\&status=done\&style=none\&width=861.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622741939645-717531e0-bbf2-4eda-886f-b203e2383ba7.png#align=left\&display=inline\&height=169\&margin=\[object Object]\&name=image.png\&originHeight=338\&originWidth=1756\&size=329406\&status=done\&style=none\&width=878> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622742036399-c0894bb4-98f5-4915-abc4-641d3ae3d89e.png#align=left\&display=inline\&height=355\&margin=\[object Object]\&name=image.png\&originHeight=709\&originWidth=870\&size=211575\&status=done\&style=none\&width=435> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622742183749-6867a3c4-0066-4c15-af4a-3afd8c4d5aa9.png#align=left\&display=inline\&height=346\&margin=\[object Object]\&name=image.png\&originHeight=692\&originWidth=873\&size=363817\&status=done\&style=none\&width=436.5> "image.png")

-   总结
    -   针对幸存者s0,s1区的总结：复制之后有交换，谁空谁是to
    -   关于垃圾回收：频繁在新生区收集，很少在养老去收集，几乎不在方法区收集。

#### 2.5、Minor GC、Major GC、Full GC

-   GC按照回收区域分为两大类：一类是部分收集，一类是整堆收集
    -   部分收集：不是完整收集整个java堆的垃圾收集，其中又分为：
        -   新生代收集(Young GC / Minor GC):只是针对新生代的垃圾收集
        -   老年代收集(Old GC / Major GC):只是老年代的垃圾收集。
            -   注意，很多时候Major GC和Full GC混合使用，需要具体分辨是老年代回收还是整堆回收。
        -   混合收集：收集整个新生代以及部分老年代的垃圾收集，木器只有G1 GC会有这种行为。
    -   整堆收集(Full GC) ：收集整个java堆和方法区的垃圾收集
-   年轻代GC触发机制
    -   当年轻代空间不足时，就会触发Minor GC，这里的年轻代满指的是Eden区满，Survivor区满不会触发GC，Survivor区是被动收集。
    -   Minor GC会引发STW，暂停其他用户线程，等待垃圾回收结束，用户线程才恢复运行。
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622743198795-6141df85-2bc7-4c96-b271-4ab4be7f4060.png#align=left\&display=inline\&height=519\&margin=\[object Object]\&name=image.png\&originHeight=1037\&originWidth=1805\&size=807843\&status=done\&style=none\&width=902.5> "image.png")
-   老年代GC触发机制
    -   老年区满，会触发Major GC或Full GC
    -   出现了Major GC 经常会伴随着至少一次的Minor GC，，单着并不是绝对的，在Parallel scavenge中，直接进行Major GC
    -   Major GC的速度一般会比Minor GC慢10倍以上，STW的时间更长。
    -   如果Major GC后，内存还不足，就报OOM。
-   Full GC的触发情况(要尽量避免)
    -   调用System.gc(),建议系统执行Full GC 但是不是必然执行
    -   老年代空间不足
    -   方法区空间不足
    -   通过 Minor GC 进入老年代的平均大小大于老年代的可用内存。
    -   幸存者区的对象晋升到老年区的大小大于老年区的可用空间。

#### 2.6、堆空间分代思想

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622743810963-4690e4cf-56f8-44b6-a616-39a1c12f3fef.png#align=left\&display=inline\&height=446\&margin=\[object Object]\&name=image.png\&originHeight=891\&originWidth=1926\&size=857820\&status=done\&style=none\&width=963> "image.png")
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622743864091-e53ef323-ffb2-4ba0-a280-63a3b581ac25.png#align=left\&display=inline\&height=432\&margin=\[object Object]\&name=image.png\&originHeight=863\&originWidth=1818\&size=787219\&status=done\&style=none\&width=909> "image.png")

#### 2.7、内存分配策略

-   优先分配到Eden区
-   大对象直接分配到老年代
-   长期存活的对象分配到老年代
-   动态年龄判断
    -   如果Survivor 区中相同年龄额所有对象的大小总和大于Survivor空间的一般，年龄大于或等于该年龄的对象可以直接进入老年代，无须等到 MaxTenuringThreshold 中要求的年龄。
-   空间分配担保
    -   \-XX:HandlePromotionFailure

#### 2.8、为对象分配内存：TLAB

-   为什么有TLAB(Thread Local Allocation Buffer)
    -   堆空间是线程共享区域，任何线程都可以访问到堆区的共享数据
    -   由于对象实例的创建在JVM中非常频繁，因此在并发环境下从对重划分内存空间是线程不安全的。
    -   为了避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度。
-   什么是TLAB
    -   在Eden区中为每个线程分配一个私有的缓冲区域。
    -   多线程中分配内存空间使用TLAB，可以避免一系列的非线程安全问题，同时还能够提升内存分配的吞吐量，这种分配方式，称为快速分配策略。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622745365156-0fe678b3-a8c2-4c2f-837b-c18329bc7ce2.png#align=left\&display=inline\&height=422\&margin=\[object Object]\&name=image.png\&originHeight=844\&originWidth=1596\&size=251294\&status=done\&style=none\&width=798> "image.png")

-   尽管不是所有的对象实例都能在TLAB中成功分配内存，但是JVM确实将TLAB作为内存分配的首选。。
-   在程序开发非过程中可以通过 -XX:UseTLAB 参数设置是否开启TLAB空间。
-   默认情况下，TLAB空间的内存非常小，仅占整个Eden空间的1%， 我们可以通过选项： -XX:TLABWasteTargetPercent 设置TLAB空间所占Eden空间的百分比。
-   一旦对象在TLAB空间分配内存失败时，JVM就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在Eden区空间分配内存
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622745925821-4a8c4fec-0850-469b-9d76-d7c472793d04.png#align=left\&display=inline\&height=428\&margin=\[object Object]\&name=image.png\&originHeight=855\&originWidth=1826\&size=524498\&status=done\&style=none\&width=913> "image.png")

#### 2.9、堆空间参数设置

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622745973323-afe15c21-4f64-4c98-a6b2-430a856f58e7.png#align=left\&display=inline\&height=395\&margin=\[object Object]\&name=image.png\&originHeight=790\&originWidth=1820\&size=805857\&status=done\&style=none\&width=910> "image.png")
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622746066200-c9a88ac4-b152-4558-abb3-6d464da927ca.png#align=left\&display=inline\&height=249\&margin=\[object Object]\&name=image.png\&originHeight=497\&originWidth=1794\&size=493046\&status=done\&style=none\&width=897> "image.png")
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622746235397-479df146-292b-4c63-9a03-e99940480922.png#align=left\&display=inline\&height=467\&margin=\[object Object]\&name=image.png\&originHeight=935\&originWidth=1775\&size=1510719\&status=done\&style=none\&width=887.5> "image.png")

#### 2.10、堆是分配对象存储的唯一选择吗？NO

-   如果经过逃逸分析之后发现，一个对象没有逃逸出方法的话，那么久可能被优化成栈上分配。无须在堆上分配，也无须进行垃圾回收。
-   逃逸分析的基本行为就是分析对象动态作用域   
    -   当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸。
    -   当一个对象在方法中被定义，它被外部方法所引用，则认为发生逃逸。
-   JDK7之后HotSpot默认开启逃逸分析。
-   可以使用选项：-XX:+DoEscapeAnalysis 显示开启逃逸分析
-   使用：-XX:+PrintEscapeAnalysis 查看逃逸分析的筛选结果。
-   开发中能使用局部变量就使用局部变量。
-   逃逸分析中的代码优化
    -   栈上分配
        -   将堆分配转换为栈分配
    -   同步省略(锁消除)
        -   如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步。
    -   分离对象或标量替换
        -   有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分或全部可以不存储在内存，而是存储在CPU寄存器中。
        -
            ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622747855495-224936e1-fe9d-44bf-a974-566f796177eb.png#align=left\&display=inline\&height=479\&margin=\[object Object]\&name=image.png\&originHeight=958\&originWidth=1956\&size=1015543\&status=done\&style=none\&width=978> "image.png")
        -   标量替换参数设置：-XX:EliminateAllocations 开启标量替换，默认打开。
        -
            ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622748061792-56abb3b6-dde3-4311-9c47-f9ccacf10b07.png#align=left\&display=inline\&height=471\&margin=\[object Object]\&name=image.png\&originHeight=941\&originWidth=1922\&size=1192260\&status=done\&style=none\&width=961> "image.png")
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622748209218-97fd7d3d-93fc-433b-b56b-48363539d0ce.png#align=left\&display=inline\&height=481\&margin=\[object Object]\&name=image.png\&originHeight=961\&originWidth=1841\&size=1752048\&status=done\&style=none\&width=920.5> "image.png")

### 3、方法区

#### 3.1、堆、栈、方法区的交互关系

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622808752241-c9365204-7bdf-431a-9ad9-d84309d4df72.png#align=left\&display=inline\&height=571\&margin=\[object Object]\&name=image.png\&originHeight=1141\&originWidth=2341\&size=1424424\&status=done\&style=none\&width=1170.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622808928164-0a13807e-791a-4617-9438-019928fde690.png#align=left\&display=inline\&height=517\&margin=\[object Object]\&name=image.png\&originHeight=1033\&originWidth=2011\&size=583600\&status=done\&style=none\&width=1005.5> "image.png")

#### 3.2、方法区的理解

-   《java虚拟机规范》中明确说明：尽管所有的方法区在逻辑上属于堆的一部分，但是一些简单的实现可能不会选择去进行垃圾收集或者进行压缩，但是对于HotSopt虚拟机而言，方法区还有一个别名叫做Non-Heap，目的就是要和堆分开。所以方法区看做是一块独立于java堆的内存空间。
-   方法区与java堆一样是一块线程共享的内存区域
-   方法区在JVM启动时被创建，物理内存空间可以是不连续的。
-   方法区的大小，跟堆空间一样可以固定大小也可以扩展。
-   方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，虚拟机同样会报：OOM
-   HotSopt中方法区的演进
    -   JDK7：永久代，JDK8：元空间
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622809736703-1b0dae57-ed25-48c6-994f-de74610a072b.png#align=left\&display=inline\&height=510\&margin=\[object Object]\&name=image.png\&originHeight=1019\&originWidth=2006\&size=567225\&status=done\&style=none\&width=1003> "image.png")
    -   元空间和永久代的最大区别是：元空间不在虚拟机设置的内存中，而是使用本地内存。OOM同样存在，受到外部内存大小的限制。

#### 3.3、设置方法区大小与OOM

-   JDK7及其以前
    -   通过 -XX:PermSize 来设置永久代初始分配空间，默认值是20.75M
    -   \-XX:MaxPermSize 来设定永久代最大可分配空间。32位及其默认值是64M，64位及其默认值是82M。超过最大值会报OOM
-   JDK8及其之后
    -   元空间的大小可以使用参数 -XX:MetaspaceSize和-XX:MaxMetaspaceSize 来设置
    -   默认值依赖于平台，在windows平台下，-XX:MetaspaceSize是21M,-XX:MaxMetaspaceSize 是-1 ，表示没有限制。
    -   \-XX:MetaspaceSize 的值是一个初始的高水位线，一旦触及这个水位线，Full GC将会被触发并卸载没用的类，然后这个高水位线将会被重置，新的高水位线的值取决于GC后释放了多少元空间，如果元空间释放不足，那么在不超过MaxMetaspaceSize时，适当提高该值，如果释放空间过多，则适当降低该值。
    -   如果初始的高水位线设置过低，上述高水位线调整情况会发生很多次，通过垃圾回收的日志可以观察到Full GC多次调用，为了避免频繁GC，建议将高水位线设置为一个相对较高的值。
    -   如何解决OOM？
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622811219249-d760487e-725a-40af-87bd-41ea0e4c1e59.png#align=left\&display=inline\&height=457\&margin=\[object Object]\&name=image.png\&originHeight=913\&originWidth=1790\&size=1187851\&status=done\&style=none\&width=895> "image.png")

#### 3.4、方法区的内部结构

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622811324563-fdb7121d-2bc1-49b3-9a22-7eb6fe9a8b6c.png#align=left\&display=inline\&height=565\&margin=\[object Object]\&name=image.png\&originHeight=1130\&originWidth=1969\&size=1023645\&status=done\&style=none\&width=984.5> "image.png")

-   方法区中存储什么？
    -   方法区中主要存储已经被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等等。
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622811424344-f59ec1d2-2361-4054-bef4-472f3f8de44a.png#align=left\&display=inline\&height=174\&margin=\[object Object]\&name=image.png\&originHeight=348\&originWidth=1865\&size=346921\&status=done\&style=none\&width=932.5> "image.png")
    -   类型信息：
        -   对于加载的类型：class、interface、enum、annotation必须存储以下信息
            -   类型的全类名
            -   权限修饰符
            -   直接父类的完整有效名
            -   直接接口的有序列表
    -   域信息：
        -   保存域的相关信息和域声明的顺序
        -   相关信息：域名称、域类型、域修饰符
    -   方法信息：
        -   保存方法的相关信息和声明顺序
        -   相关信息：方法名、返回值类型、参数数量和类型、方法修饰符、方法的字节码，异常表
-   non-final的类变量
    -   类数据逻辑上的一部分，被所有的类的实例共享，即使没有实例对象也能访问。
    -   被声明为final的变量在编译的时候就会被分配。
-   运行时常量池和常量池
    -   方法区内部包含了运行时常量池
    -   字节码文件内部包含了常量池。字节码文件中存储了指向常量池的符号引用。
    -   常量池中存储的数据类型主要包括：数量值、字符串值、类引用、字段引用、方法引用。
    -   常量池，可以看做一张表，虚拟机指令根据这张表找到要执行的类名，方法名，参数类型、字面量等类型。
    -   运行时常量池是方法区的一部分。
    -   常量池表时class文件的一部分，用于存放编译期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。
    -   在加载类和接口到虚拟机后，就会创建对应的运行时常量池。
    -   JVM为每一个加载的类型(类或接口)都维护一个常量池，池中的数据项就想数组项一样可以通过索引访问。
    -   运行时常量池也会OOM。
    -   运行时常量池将常量池中的符号引用转换为真实地址。
    -   运行时常量池相对于常量池的一个重要特征是：具有动态性。

#### 2.5、方法区使用举例

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622813850698-7718d868-60f6-4004-8d61-e12039dc30d6.png#align=left\&display=inline\&height=193\&margin=\[object Object]\&name=image.png\&originHeight=385\&originWidth=911\&size=152163\&status=done\&style=none\&width=455.5> "image.png")

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622813867991-620cf5c0-4cf4-46e8-8c5c-f8c23608c74b.png#align=left\&display=inline\&height=519\&margin=\[object Object]\&name=image.png\&originHeight=1037\&originWidth=1985\&size=780853\&status=done\&style=none\&width=992.5> "image.png")

#### 2.6、方法区的演进细节

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622813954426-3e665b76-006d-4f92-ab51-ecf7ca3267ba.png#align=left\&display=inline\&height=434\&margin=\[object Object]\&name=image.png\&originHeight=867\&originWidth=1821\&size=1257824\&status=done\&style=none\&width=910.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622814092481-a4b18ae9-4145-49c3-b869-a1a89c1ecc29.png#align=left\&display=inline\&height=366\&margin=\[object Object]\&name=image.png\&originHeight=732\&originWidth=1617\&size=426957\&status=done\&style=none\&width=808.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622814152727-8afd308a-7cac-486d-a521-ecf51cd73ac3.png#align=left\&display=inline\&height=367\&margin=\[object Object]\&name=image.png\&originHeight=733\&originWidth=1607\&size=441790\&status=done\&style=none\&width=803.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622814186234-5d0ea09a-84f1-4a47-92e2-7299c84943b0.png#align=left\&display=inline\&height=371\&margin=\[object Object]\&name=image.png\&originHeight=742\&originWidth=1918\&size=636883\&status=done\&style=none\&width=959> "image.png")

-   永久代为什么要被元空间替换
    -   为永久代设置空间大小是很难确定的
    -   对永久代进行调优是很困难的。
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622814438172-520aa21e-76c2-4351-82a6-886dbeeaf1b6.png#align=left\&display=inline\&height=458\&margin=\[object Object]\&name=image.png\&originHeight=916\&originWidth=1920\&size=1277194\&status=done\&style=none\&width=960> "image.png")
-   StringTable 为什么要调整？
    -   JDK7中将StringTable放到了堆空间中，因为永久代的回收效率很低，在Full GC的时候才会触发，而Full GC 是在老年代空间不足、永久代空间不足时才会触发，这就导致StringTable回收效率不高，而我们开发中会有大量的字符串被创建，回收效率低，导致永久代内存不足，放到堆中，能及时回收内存。
-   静态变量放在那里？
    -   静态引用对应的对象实体始终存放在堆空间中。

#### 3.7、方法区的垃圾回收

-   方法区的垃圾回收主要回收两部分内容：常量池中废弃的常量和不再使用的类型。
-   常量池中主要存放的两大类型：字面量和符号引用
    -   字面量：比如文本字符创、被声明为final的常量值
    -   符号引用：类和接口的全限定名、字段的名称和描述符、方法的名称和描述符。
-   HotSpot虚拟机对常量池的回收策略是很明确的，只要常量池中的常量没有被任何地方引用，就可以被回收。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622820074707-7e676a02-e835-4359-8808-ab7f8407cd91.png#align=left\&display=inline\&height=473\&margin=\[object Object]\&name=image.png\&originHeight=945\&originWidth=1877\&size=1446269\&status=done\&style=none\&width=938.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622820132846-79a47a16-86e6-497c-b607-2c244c8e4c60.png#align=left\&display=inline\&height=470\&margin=\[object Object]\&name=image.png\&originHeight=940\&originWidth=1942\&size=677333\&status=done\&style=none\&width=971> "image.png")

### 4、对象的实例化、内存布局与访问定位

#### 4.1、对象的实例化

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622820537396-f6287e38-72a8-48c9-b7ea-23225ca6eee4.png#align=left\&display=inline\&height=655\&margin=\[object Object]\&name=image.png\&originHeight=1309\&originWidth=2127\&size=610472\&status=done\&style=none\&width=1063.5> "image.png")

#### 4.2、对象的内存布局

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622821328524-8a524100-2473-4558-a74e-06deb295ee60.png#align=left\&display=inline\&height=659\&margin=\[object Object]\&name=image.png\&originHeight=1317\&originWidth=1591\&size=368573\&status=done\&style=none\&width=795.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622821818021-e52e9632-10eb-40ff-bc55-2dbd177568df.png#align=left\&display=inline\&height=536\&margin=\[object Object]\&name=image.png\&originHeight=1072\&originWidth=2016\&size=1423863\&status=done\&style=none\&width=1008> "image.png")

#### 4.3、对象的访问定位

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622822360007-d1688c76-e0af-4306-a0ae-7e471e0d8f07.png#align=left\&display=inline\&height=372\&margin=\[object Object]\&name=image.png\&originHeight=744\&originWidth=2416\&size=309780\&status=done\&style=none\&width=1208> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622822495221-ce317ffa-6eee-4b48-83e2-e4867ef5f892.png#align=left\&display=inline\&height=530\&margin=\[object Object]\&name=image.png\&originHeight=1060\&originWidth=1736\&size=668381\&status=done\&style=none\&width=868> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622822508861-c6b75a74-15fd-4409-812e-83835a118b7d.png#align=left\&display=inline\&height=503\&margin=\[object Object]\&name=image.png\&originHeight=1005\&originWidth=1747\&size=514708\&status=done\&style=none\&width=873.5> "image.png")

### 5、直接内存

-   直接内存不是虚拟机运行时数据区的一部分，是java堆外的，直接向系统申请的内存空间，
-   访问直接内存的速度会优于java堆，读写频繁的场合可以使用直接内存。java的NIO允许使用java程序使用直接内存，用于数据缓冲区。
-   直接内存也可能导致OutOfMemoryError。
-   java堆和直接内存会受到系统给出的最大内存限制。
-   直接内存大小可以通过MaxDirectoryMemorySize设置
-   如果不指定，默认与堆的最大值参数一致
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622823041217-49c370c0-64b6-47ef-b196-b8e73c1aecd4.png#align=left\&display=inline\&height=491\&margin=\[object Object]\&name=image.png\&originHeight=981\&originWidth=1651\&size=840934\&status=done\&style=none\&width=825.5> "image.png")

### 6、执行引擎

#### 6.1、执行引擎概述

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622823115200-dccdd131-b44a-4e0c-8347-1f700320f5d9.png#align=left\&display=inline\&height=470\&margin=\[object Object]\&name=image.png\&originHeight=939\&originWidth=1517\&size=857229\&status=done\&style=none\&width=758.5> "image.png")

-   执行引擎的任务：将字节码指令解释或编译为对应平台上的本地机器指令。
-   执行引擎的工作过程

    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622823448340-49f59621-5599-470c-a130-dd2e9f7bca3c.png#align=left\&display=inline\&height=482\&margin=\[object Object]\&name=image.png\&originHeight=963\&originWidth=1879\&size=1043704\&status=done\&style=none\&width=939.5> "image.png")

#### java代码编译和执行的过程

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622823595951-15bb0c29-cc7d-4d08-9793-37748822b721.png#align=left\&display=inline\&height=434\&margin=\[object Object]\&name=image.png\&originHeight=867\&originWidth=1638\&size=732545\&status=done\&style=none\&width=819> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622823654527-dea451e3-3520-4564-9cae-0b3ecb5d9423.png#align=left\&display=inline\&height=297\&margin=\[object Object]\&name=image.png\&originHeight=593\&originWidth=1571\&size=1175733\&status=done\&style=none\&width=785.5> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622823693765-17f0010b-c946-40a6-b0f0-5c4281546d28.png#align=left\&display=inline\&height=371\&margin=\[object Object]\&name=image.png\&originHeight=741\&originWidth=1611\&size=1158488\&status=done\&style=none\&width=805.5> "image.png")

-   什么是解释器，什么是JIT编译器
    -   解释器：当java虚拟机启动的时候会根据预定义的规范对字节码采用逐行解释的方式执行，将每条字节码文件中的内容翻译为对应平台的本地机器指令执行。
    -   JIT编译器：就是虚拟机将源代码直接编译成和本地机器平台相关的机器语言。
-   为什么说java是半编译，半解释语言
    -   在执行java代码的时候，通常会将解释执行与编译执行二者结合起来进行。。
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622824958521-a11121e8-7086-4b27-984b-51f643b5083d.png#align=left\&display=inline\&height=532\&margin=\[object Object]\&name=image.png\&originHeight=1064\&originWidth=1974\&size=1100564\&status=done\&style=none\&width=987> "image.png")

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622825076039-1784ec37-d98e-4409-8624-8ab29e9ffe4c.png#align=left\&display=inline\&height=430\&margin=\[object Object]\&name=image.png\&originHeight=859\&originWidth=1467\&size=302239\&status=done\&style=none\&width=733.5> "image.png")

#### 6.2、解释器

-   解释器分类：
    -   字节码解释器(古老)
    -   模板解释器(现在普遍使用)

#### 6.3、JIT编译器

-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622826109087-430ebce4-fb2c-40dd-9a3e-a14140e32e65.png#align=left\&display=inline\&height=463\&margin=\[object Object]\&name=image.png\&originHeight=926\&originWidth=1857\&size=1683593\&status=done\&style=none\&width=928.5> "image.png")

-   ## HotSpot JVM的执行方式
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622826162132-43afe40d-7d49-48eb-ae22-8969b983ac2f.png#align=left\&display=inline\&height=225\&margin=\[object Object]\&name=image.png\&originHeight=450\&originWidth=1849\&size=725549\&status=done\&style=none\&width=924.5> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622826333225-e70ec740-11d0-4299-9312-6f55f750b765.png#align=left\&display=inline\&height=417\&margin=\[object Object]\&name=image.png\&originHeight=833\&originWidth=1805\&size=1034658\&status=done\&style=none\&width=902.5> "image.png")
-   热点代码及探测方式
    -   一个被多次调用的方法，或者一个方法体内部循环次数较多的循环体都可被称之为热点代码。
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622826643069-db804962-8e54-436b-af0d-ed96f4a77084.png#align=left\&display=inline\&height=423\&margin=\[object Object]\&name=image.png\&originHeight=846\&originWidth=1703\&size=1375220\&status=done\&style=none\&width=851.5> "image.png")
    -   方法调用计数器
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622826811597-526e2dcf-1201-4ed1-964d-7d3f0a57f663.png#align=left\&display=inline\&height=552\&margin=\[object Object]\&name=image.png\&originHeight=1104\&originWidth=1371\&size=738284\&status=done\&style=none\&width=685.5> "image.png")
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622826934960-1c1fd831-97a5-49bb-9e72-4a02beb6a686.png#align=left\&display=inline\&height=433\&margin=\[object Object]\&name=image.png\&originHeight=865\&originWidth=1768\&size=1219143\&status=done\&style=none\&width=884> "image.png")
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622827022499-285a3998-2c54-46bb-a6ff-9d2e8016e8e6.png#align=left\&display=inline\&height=538\&margin=\[object Object]\&name=image.png\&originHeight=1075\&originWidth=1437\&size=466011\&status=done\&style=none\&width=718.5> "image.png")
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622827071933-84e65b67-25c8-451b-8f2b-30426ed30604.png#align=left\&display=inline\&height=332\&margin=\[object Object]\&name=image.png\&originHeight=664\&originWidth=1712\&size=671588\&status=done\&style=none\&width=856> "image.png")
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622827160431-addaac0a-1aa3-4f94-9447-0aa026d5be93.png#align=left\&display=inline\&height=323\&margin=\[object Object]\&name=image.png\&originHeight=645\&originWidth=1745\&size=781544\&status=done\&style=none\&width=872.5> "image.png")
    -
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622827232532-0d08dc1e-8751-44a1-b99c-39a5448072a6.png#align=left\&display=inline\&height=368\&margin=\[object Object]\&name=image.png\&originHeight=736\&originWidth=1671\&size=844372\&status=done\&style=none\&width=835.5> "image.png")
