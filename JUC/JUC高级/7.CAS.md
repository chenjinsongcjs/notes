# CAS

-   是什么
    -   CAS是compare and swap 的缩写，即比较并交换，是并发算法中常用的一种技术，主要涉及到内存偏移量，期望值和更新值
    -   在进行CAS操作的时候，将内存位置的值和期望的值进行比较，如果相等，就将原来的值更新为最新的值，如果不相等，就什么都不做或者重来
    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_Pf3EJUdv-_.png)
    -   CAS有硬件级别保证
        -   CAS是JDK提供的**非阻塞原子性操作**，它通过硬件保证了比较-更新的原子性。它是非阻塞的且自身原子性，也就是说这玩意效率更高且通过硬件保证，说明这玩意更可靠。
        -   CAS是一条CPU的原子指令（cmpxchg指令），不会造成所谓的数据不一致问题，Unsafe提供的CAS方法（如compareAndSwapXXX）底层实现即为CPU指令cmpxchg。
        -   执行cmpxchg指令的时候，会判断当前系统是否为多核系统，如果是就给总线加锁，只有一个线程会对总线加锁成功，加锁成功之后会执行cas操作，也就是说CAS的原子性实际上是CPU实现的， 其实在这一点上还是有排他锁的，只是比起用synchronized， 这里的排他时间要短的多， 所以在多线程情况下性能会比较好
-   compareAndSet源码

    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_W-eOlKeUpp.png)

> 上面三个方法都是类似的，主要对4个参数做一下说明。
> var1：表示要操作的对象
> var2：表示要操作对象中属性地址的偏移量
> var4：表示需要修改数据的期望的值
> var5/var6：表示需要修改为的新值

## CAS底层原理？如果知道，谈谈你对UnSafe的理解

-   CAS底层：Unsafe + Spinlock
-   Unsafe类是CAS的核心类，由于java不能直接操作内存，所以就需要调用native方法来访问，Unsafe类就相当于一个后们，Unsafe类中的所有方法都是native的，基于该类可以直接访问指定的内存空间，Unsafe类在sun.misc包中，里面的方法可以想c一样使用指针操作内存空间。CAS的操作都依赖于Unsafe类的方法
-   注意Unsafe类中的所有方法都是native修饰的，也就是说Unsafe类中的方法都直接调用操作系统底层资源执行相应任务&#x20;

## 自旋锁（Spinlock）

-   线程在获取锁的过程中不会阻塞，而是采用循环的方式尝试获取锁，当发现锁被占用时，不断的去判断锁的状态，直到获取。
-   这种锁的好处时减少类线程上下文的切换，缺点是：循环会消耗CPU
-   自己实现自旋锁

```java
package com.xyz;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;

public class SpinLockDemo {
    private static volatile AtomicReference<Thread> spinLock = new AtomicReference<>();
    public void lock(){
        while (!spinLock.compareAndSet(null,Thread.currentThread())){
            System.out.println(Thread.currentThread().getName()+" 自旋");
        }
        System.out.println(Thread.currentThread().getName()+" get lock");
    }
    public void unlock(){
        System.out.println(Thread.currentThread().getName()+" release lock");//io耗时
        spinLock.compareAndSet(Thread.currentThread(),null);
    }
    //自旋锁测试
    public static void main(String[] args) {
        SpinLockDemo lockDemo = new SpinLockDemo();
        new Thread(()->{
            try{
                lockDemo.lock();
                System.out.println(Thread.currentThread().getName()+" come in");
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lockDemo.unlock();
            }
        },"t1").start();
        new Thread(()->{
            try{
                lockDemo.lock();
                System.out.println(Thread.currentThread().getName()+" come in");
            }finally {
                lockDemo.unlock();
            }
        },"t2").start();
    }
}

```

### CAS缺点

-   循环测试锁状态，消耗CPU
-   ABA问题—>使用带有时间戳的原子引用解决
-   只能保证一个变量的原子性
