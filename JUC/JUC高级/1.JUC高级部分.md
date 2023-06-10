# JUC高级部分

## 为什么多线程及其重要？

-   硬件方面：摩尔定律失效，从2003年开始CPU主频已经不再翻倍，而是采用多核而不是更快的主频。在主频不再提高且核数在不断增加的情况下，要想让程序更快就要用到并行或并发编程。
-   软件方面：企业的高并发系统，异步和回调等业务需求，都需要使用到多线程

## 线程启动的底层源码

-   java中的线程和操作系统中的线程是一一对应的，
-   java中启动线程，会调用本地方法，start0()，是c++代码，会调用底层的操作系统命令创建并启动线程

```java
public class JUC {
    public static void main(String[] args) {
        new Thread(()-> System.out.println("h"),"t1").start();
    }
}
private native void start0();

```

## java多线程的相关概念

-   进程：进程是程序的一次执行，是系统进行资源分配和调度的独立单位，进程有自己的内存空间和资源。
-   线程：线程是CPU调度的最小单位，一个进程中可以有多个线程，多个线程共享进程的系统资源
-   管程：Monitor（监视器），也就是我们平时所说的锁。

```java
Monitor其实是一种同步机制，他的义务是保证（同一时间）只有一个线程可以访问被保护的数据和代码。
 
JVM中同步是基于进入和退出监视器对象(Monitor,管程对象)来实现的，每个对象实例都会有一个Monitor对象，
 Object o = new Object();
new Thread(() -> {
    synchronized (o)
    {

    }
},"t1").start();
 
Monitor对象会和Java对象一同创建并销毁，它底层是由C++语言来实现的。
```

## 用户线程和守护线程

-   java的线程分为用户线程和守护线程，将线程的daemon属性设置为true表示守护线程，false表示用户线程
-   守护线程：是一种特殊的线程，在后台默默的执行一些系统性任务，比如GC线程回收垃圾
-   用户线程：是系统的工作线程，它会完成这个程序需要完成的业务操作

> 将用户线程设置为守护线程必须在线程还没有启动时才能设置。

> 当所有用户线程结束时，不管守护线程是否还在运行，所有的守护线程都会被kill掉

```java
public class JUC {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " 是 " +
                    (Thread.currentThread().isDaemon() ? "守护线程" : "用户线程"));
        }, "t1");
        //守护线程的设置必须在线程启动之前
        t1.setDaemon(true);
        t1.start();
        TimeUnit.SECONDS.sleep(1);
    }
}

```

# 细节

[CompletableFuture](https://www.wolai.com/91vF1v3cXWgJPfSiQgGWfN "CompletableFuture")

[LockSupport和线程中断](https://www.wolai.com/uGtoeHasDVyd86HGFHwyd3 "LockSupport和线程中断")

[JMM(Java内存模型)](https://www.wolai.com/ekEjLHo2uUVdTr6MKmJ8Hj "JMM(Java内存模型)")

[volatile解析](https://www.wolai.com/qnHHTGwmKzm6LFgQL34Dze "volatile解析")

[CAS](https://www.wolai.com/uHwKbFKxRSnBbNBghdvZ5q "CAS")

[原子操作类之18罗汉增强](https://www.wolai.com/jEPNsr6vbscbPZGXQPzbbT "原子操作类之18罗汉增强")

[ThreadLocal](https://www.wolai.com/v4pyRQ8ZFdBLqXNnpKWmbJ "ThreadLocal")

[Java对象内存布局和对象头](https://www.wolai.com/hyeXDsRpD4LGaDspWbvPsX "Java对象内存布局和对象头")

[Synchronized与锁升级](https://www.wolai.com/eVgMfebPgTJZXcKeWqA3Te "Synchronized与锁升级")

[AbstractQueuedSynchronizer](https://www.wolai.com/cwVKRrJdQJB5dx3i9LUcT3 "AbstractQueuedSynchronizer")

[ReentrantLock、ReentrantReadWriteLock、StampedLock](https://www.wolai.com/tbrB9hCjmfFZG7RLNfS4aa "ReentrantLock、ReentrantReadWriteLock、StampedLock")
