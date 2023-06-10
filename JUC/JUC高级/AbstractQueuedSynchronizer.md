# AbstractQueuedSynchronizer

-   抽象队列同步器
-   是用来构建锁和其他同步器的重量级基础框架和整个JUC体系的基石，通过内置FIFO队列来完成资源获取线程的排队，并通过一个int变量表示锁的持有状态。
-

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_9oE1_qi4Dt.png)

-   ReentrantLock、CountDownLatch、ReentrantReadWriteLock、Semaphore等等底层都有一个静态的内部类Sync继承于AbstractQueuedSynchronizer来实现它们的功能。
-   进一步理解锁和同步器的关系
    -   锁是面向锁的使用者，定义了程序员和锁交互的使用层API，隐藏了实现细节，你调用即可。
    -   同步器是面向锁的实现者，比如Java并发大神DougLee，提出统一规范并简化了锁的实现，屏蔽了同步状态管理、阻塞线程排队和通知、唤醒机制等。
-   AOS能干嘛？
    -   抢到资源的线程直接使用处理业务，抢不到资源的必然涉及一种排队等候机制。抢占资源失败的线程继续去等待(类似银行业务办理窗口都满了，暂时没有受理窗口的顾客只能去候客区排队等候)，但等候线程仍然保留获取锁的可能且获取锁流程仍在继续(候客区的顾客也在等着叫号，轮到了再去受理窗口办理业务)。
    -   如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配。这个机制主要用的是CLH队列的变体实现的，将暂时获取不到锁的线程加入到队列中，这个队列就是AQS的抽象表现。它将请求共享资源的线程封装成队列的结点（Node），通过CAS、自旋以及LockSupport.park()的方式，维护state变量的状态，使并发达到同步的效果。
    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_x0cf_WYAmc.png)

### AOS初始

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_RKj-dPFdQA.png)

-   AQS使用一个volatile的int类型的成员变量来表示同步状态，通过内置的FIFO队列来完成资源获取的排队工作将每条要去抢占资源的线程封装成一个Node节点来实现锁的分配，通过CAS完成对State值的修改。

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_JHiM2OjlYy.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_MuHxYpDAON.png)

### AQS内部体系

#### AQS自身

-   AQS的int变量
    -   AQS的同步状态state变量：private volatile int state
    -   0 表示没有线程占有，1表示有线程占有，大于1表示重入
-   AQS的CLH队列
    -
    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_WY2o_7CYMK.png)
-   总结
    -   有阻塞就需要排队，排队就需要队列
    -   AQS = state + CLH队列

#### 内部类Node(Node类在AQS类内部)

-   Node的等待状态waitStatus成员变量：volatile int waitStatus
-

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_uBMFQ4AldZ.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_EmJgixVx6A.png)

#### AQS的基本结构

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_8zpCxCx3d4.png)

### 从我们的ReentrantLock开始解读AQS

-   Lock接口的实现类，基本都是通过【聚合】了一个【队列同步器】的子类完成线程访问控制的
-

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_3SFr5oaq9r.png)

-   可以明显看出公平锁与非公平锁的lock()方法唯一的区别就在于公平锁在获取同步状态时多了一个限制条件：`hasQueuedPredecessors()`，`hasQueuedPredecessors`是公平锁加锁时判断等待队列中是否存在有效节点的方法
-   从Lock方法开始（记录源码重要过程）

```java
对比公平锁和非公平锁的 tryAcquire()方法的实现代码，其实差别就在于非公平锁获取锁时比公平锁中少了一个判断 !hasQueuedPredecessors()
 
hasQueuedPredecessors() 中判断了是否需要排队，导致公平锁和非公平锁的差异如下：
 
公平锁：公平锁讲究先来先到，线程在获取锁时，如果这个锁的等待队列中已经有线程在等待，那么当前线程就会进入等待队列中；
 
非公平锁：不管是否有等待队列，如果可以获取锁，则立刻占有锁对象。也就是说队列的第一个排队线程在unpark()，之后还是需要竞争锁（存在线程竞争的情况下）
```

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_OSbWpHmJtq.png)

-   acquire()
    -
    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_S2KCIvBdZz.png)
-   tryAcquire(arg)（非公平锁的）
    -   return fasle ：继续推进条件，走下一个方法
    -   return true ：结束
    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_fgMR60S8ZA.png)
-   addWaiter(Node.EXCLUSIVE)
    -
    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_Hmy8_2vYms.png)
    -   enq(node)  双向链表中，第一个节点为虚节点(也叫哨兵节点)，其实并不存储任何信息，只是占位。真正的第一个有数据的节点，是从第二个节点开始的。当队列中的第一个线程获取锁，它在队列中的节点就会成为新的虚拟节点，然后把旧的虚拟节点删除
    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_z0sqeCGQvs.png)
-   acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
    -
    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_vV9fkeMT6u.png)
    -   假如再抢抢失败就会进入：shouldParkAfterFailedAcquire 和 parkAndCheckInterrupt 方法中
        -   如果前驱节点的 waitStatus 是 SIGNAL状态，即 shouldParkAfterFailedAcquire 方法会返回 true 程序会继续向下执行 parkAndCheckInterrupt 方法，用于将当前线程挂起
        ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_12F5-VqZJ6.png)
    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_PI_X0U0fSD.png)

    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_bOAeh0HT3L.png)

### 解锁

-   unlock —>sync.release(1)—>tryRelease(arg)—>unparkSuccessor

***

