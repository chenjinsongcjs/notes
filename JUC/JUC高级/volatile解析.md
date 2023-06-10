# volatile解析

## 被volatile修饰的变量的特点？

-   可见性
-   有序性（禁止指令重排序）

> volatile的内存语义？
> 1.当对volatile变量进行写操作时，JMM会将当前线程工作内存中的值立即刷新回主内存
> 2.当对volatile变量进行读操作时，JMM会将当前线程工作内存中值失效，并从主内存中获取最新的值
> 总结：写直接写回主内存，读直接从主内存中读

## volatile变量的读写过程

-   JMM中定义了8中工作内存和主内存中的原子操作

    **read(读取)→load(加载)→use(使用)→assign(赋值)→store(存储)→write(写入)→lock(锁定)→unlock(解锁)**
-

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_4w8qCe8PEm.png)

> read: 作用于主内存，将变量的值从主内存传输到工作内存，主内存到工作内存
> load: 作用于工作内存，将read从主内存传输的变量值放入工作内存变量副本中，即数据加载
> use: 作用于工作内存，将工作内存变量副本的值传递给执行引擎，每当JVM遇到需要该变量的字节码指令时会执行该操作
> assign: 作用于工作内存，将从执行引擎接收到的值赋值给工作内存变量，每当JVM遇到一个给变量赋值字节码指令时会执行该操作
> store: 作用于工作内存，将赋值完毕的工作变量的值写回给主内存
> write: 作用于主内存，将store传输过来的变量值赋值给主内存中的变量
> 由于上述只能保证单条指令的原子性，针对多条指令的组合性原子保证，没有大面积加锁，所以，JVM提供了另外两个原子指令：
> lock: 作用于主内存，将一个变量标记为一个线程独占的状态，只是写时候加锁，就只是锁了写变量的过程。
> unlock: 作用于主内存，把一个处于锁定状态的变量释放，然后才能被其他线程占用

## 内存屏障

### 内存屏障是什么？

-   &#x20;**内存屏障**（也称内存栅栏，内存栅障，屏障指令等，**是一类同步屏障指令，是CPU或编译器在对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作**），避免代码重排序。内存屏障其实就是一种JVM指令，Java内存模型的重排规则会要求Java编译器在生成JVM指令时插入特定的内存屏障指令，通过这些内存屏障指令，volatile实现了Java内存模型中的可见性和有序性，但volatile无法保证原子性。
-   **内存屏障之前的所有写操作都要回写到主内存**，
-   **内存屏障之后的所有读操作都能获得内存屏障之前的所有写操作的最新结果(实现了可见性)。**
-   volatile使用内存屏障保证可见性和有序性

#### JVM提供了四种内存屏障指令

-   java源码和c++源码中内存屏障的体现

java源码

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_ib8wmlYFFJ.png)

C++源码

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_zgJuHSzhP4.png)

-   四大内存屏障指令

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_M4N29EC-iC.png)

-   内存屏障指令在linux\_x86中的体现

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_KfkLmGBCgh.png)

#### happens-before之volatile变量规则

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_knAygl8Jaj.png)

> 当第一个操作为volatile读时，不论第二个操作是什么，都不能重排序。这个操作保证了volatile读之后的操作不会被重排到volatile读之前。
> 当第二个操作为volatile写时，不论第一个操作是什么，都不能重排序。这个操作保证了volatile写之前的操作不会被重排到volatile写之后。
> 当第一个操作为volatile写时，第二个操作为volatile读时，不能重排。

### JMM内存屏障插入的策略（四种）

-   写操作
    -   在每个volatile写操作之前插入`storestore`内存屏障
    -   在每个volatile写操作之后插入`storeload`内存屏障
        ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_wgv_4ftK2X.png)
-   读操作
    -   在每个volatile读操作之后插入`loadload`内存屏障
    -   在每个volatile读操作之后插入`loadstore`内存屏障
        ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_9nc-YwIhBc.png)

## volatile特性详细说明

### 可见性

-   保证不同线程对同一个共享变量操作时的可见性，即当一个线程改变共享变量时其他线程能够感知到

```java
    //没有使用volatile变量，线程t无法感知其他线程对标志位的修改
    static boolean flag = true;
//    static volatile boolean flag = true;
    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+" come in");
            while (flag){

            }
            System.out.println(Thread.currentThread().getName()+" 感知到终止标志.终止线程");
        },"t").start();
        TimeUnit.SECONDS.sleep(2);
        flag = false;
        System.out.println(Thread.currentThread().getName()+" 结束");
    }

```

> 没有加volatile没有保证可见性，线程无法停止
> 添加volatile 保证可见性，线程可以正常停止

### 不具有原子性

-   volatile变量的复合操作不具有原子性（如i++）
-   原子性指的是一个操作是不可中断的，即使是在多线程环境下，一个操作一旦开始就不会被其他线程影响。

```java
 static volatile int num = 0;

    public static void main(String[] args) throws InterruptedException {
        for (int i = 1; i <= 10; i++) {
            new Thread(()->{
                for (int j = 0; j < 1000; j++) {
                    num++;//++操作在字节码层次并不是只有一个指令，需要三个指令完成，读取，add，和写回，会受到多线程的影响
                }
            },String.valueOf(i)).start();
        }
        TimeUnit.SECONDS.sleep(2);
        System.out.println(num);
    }
```

-   读取赋值一个普通变量的情况

第1列

线程1从read到write的过程中，线程2随时都可以对主内存进行读

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_rsSiI5n5cq.png)

-   读取赋值一个volatile变量

第1列

`read-load-use `和 `assign-store-write `成为了两个不可分割的原子操作 **，但是在use和assign之间依然有极小的一段真空期，** 有可能变量会被其他线程读取，导致写丢失一次...o(╥﹏╥)o

但是无论在哪一个时间点主内存的变量和任一工作内存的变量的值都是相等的。这个特性就导致了**volatile变量不适合参与到依赖当前值的运算**，如i = i + 1; i++;之类的那么依靠可见性的特点volatile可以用在哪些地方呢？\*\* 通常volatile用做保存某个状态的boolean值or int值。\*\*

《深入理解Java虚拟机》提到：

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_pFoZx5oiKu.png)

-   volatile为什么不保证原子性？
    -   JVM的字节码，i++分成三步，间隙期不同步非原子操作(i++)
        ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_rfoerxnxmA.png)

### 禁止指令重排（底层使用内存屏障实现）

-   具体分析见上面的内存屏障

## 如何正确的使用volatile

-   volatile用于单一赋值，不用于复合赋值
-   状态标志位，判断业务是否结束
-   开销较低的读写锁策略
-   DCL双端检测锁（单例等）

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_L0uTa9qojm.png)
