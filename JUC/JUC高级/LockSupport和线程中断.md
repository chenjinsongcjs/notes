# LockSupport和线程中断

# 线程中断机制

## 什么是中断

-   首先，一个线程不应该是由其他线程来强制中断或停止，而是由线程自己自行停止，所以Thread.stop, Thread.suspend, Thread.resume 都已经被废弃了。
-   其次，在Java中没有办法立即停止一个线程，然而停止一个线程显得尤其重要，比如停止一个耗时的操作，因此java提供了一个停止线程的方法—中断
-   中断只是一种协作机制，Java没有增加任何语法，中断的过程完全由程序员自己决定。
-   如果要中断一个线程，你可以调用线程的interrrupt方法，该方法仅仅是将线程的中断标志位设置为true，你还需要不断的检测这个中断标志位，如果为true表示其他线程想要中断这个线程，此时究竟要做什么由自己决定。

### 中断的相关API

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_hp0UvU0Edv.png)

| `public void interrupt()`             | 实例方法，&#xA;实例方法interrupt()仅仅是设置线程的中断状态为true，不会停止线程                                                      |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `public static boolean interrupted()` | 静态方法，Thread.interrupted();判断线程是否被中断，并清除当前中断状态&#xA;这个方法做了两件事：&#xA;1 返回当前线程的中断状态&#xA;2 将当前线程的中断状态设为false |
| `public boolean isInterrupted()`      | 实例方法，&#xA;判断当前线程是否被中断（通过检查中断标志位）                                                                       |

## 如何使用中断标识停止线程？

-   在需要中断的线程中不断监听中断标志位，一旦发生中断执行中断处理逻辑。
-   实现中断的三个方法
    -   通过一个volatile变量实现
    ```java
    public class InterruptDemo {
        //volatile变量模拟中断标志位，中断线程
        private static volatile boolean isStop = false;
        public static void main(String[] args) throws InterruptedException {
            new Thread(()->{
                while (true){
                    System.out.println(Thread.currentThread().getName()+" 检测中断标志位");
                    if (isStop){
                        System.out.println(Thread.currentThread().getName()+" 发生中断");
                        break;
                    }
                }
            }).start();
            TimeUnit.SECONDS.sleep(1);
            isStop = true;
            TimeUnit.SECONDS.sleep(1);
        }
    }

    ```
    -   通过AtomicBoolean实现中断
    ```java
    public class InterruptDemo {
        //AtomicBoolean变量模拟中断标志位，中断线程
        private static final AtomicBoolean isStop = new AtomicBoolean();
        public static void main(String[] args) throws InterruptedException {
            new Thread(()->{
                while (true){
                    System.out.println(Thread.currentThread().getName()+" 检测中断标志位");
                    if (isStop.get()){
                        System.out.println(Thread.currentThread().getName()+" 发生中断");
                        break;
                    }
                }
            }).start();
            TimeUnit.SECONDS.sleep(1);
            isStop.set(true);
            TimeUnit.SECONDS.sleep(1);
        }
    }
    ```
    -   通过线程自带的中断API方法实现
    ```java
    public class InterruptDemo {
        public static void main(String[] args) throws InterruptedException {
            Thread t = new Thread(() -> {
                while (true) {
                    System.out.println(Thread.currentThread().getName() + " 检查中断");
                    if (Thread.currentThread().isInterrupted()) {
                        System.out.println(Thread.currentThread().getName() + " 发生中断");
                        break;
                    }
                }
            }, "t");
            t.start();

            TimeUnit.SECONDS.sleep(1);
            t.interrupt();
            TimeUnit.SECONDS.sleep(1);
        }
    }
    ```

第1列

实例方法`interrupt()`，没有返回值

底层调用的是native本地方法interrupt0



注：如果当前线程状态为：`wait，join，sleep`，调用`interrupt`方法会抛出一个`InterruptException`异常





第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_R2yE8iXMeI.png)

第1列

实例方法isInterrupted，返回布尔值

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_9jsU1nX3X4.png)

### 当前线程的中断标识为true，是不是就立刻停止？

具体来说，当对一个线程，调用 interrupt() 时：

①  如果线程处于正常活动状态，那么会将该线程的中断标志设置为 true，仅此而已。

被设置中断标志的线程将继续正常运行，不受影响。所以， interrupt() 并不能真正的中断线程，需要被调用的线程自己进行配合才行。

②  如果线程处于被阻塞状态（例如处于sleep, wait, join 等状态），在别的线程中调用当前线程对象的interrupt方法，

那么线程将立即退出被阻塞状态，并抛出一个`InterruptedException`异常。

### 中断异常案例

-   如果线程发生中断异常，中断标志位会被重置为false，必须在catch中重新设置中断标志位为true

```java
public class InterruptDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
            while (true) {
                System.out.println(Thread.currentThread().getName() + " 检查中断");
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println(Thread.currentThread().getName() + " 发生中断");
                    break;
                }
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    //如果发生中断异常，会将当前线程的中断标志位清除为false，
                    // 必须重新设置中断标志位为true否则会死循环
                    //Thread.currentThread().interrupt();
                    e.printStackTrace();
                }
            }
        }, "t");
        t.start();

        TimeUnit.SECONDS.sleep(1);
        t.interrupt();
        TimeUnit.SECONDS.sleep(1);
    }
}
```

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_ZiNtl0AR-t.png)

### 静态方法`Thread.interrupted()`

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_kGkzD2grCp.png)

```java
public class InterruptDemo {
    public static void main(String[] args) throws InterruptedException {
        System.out.println(Thread.interrupted());
        Thread.currentThread().interrupt();
        System.out.println(Thread.interrupted());
        System.out.println(Thread.interrupted());

    }
}
//静态方法返回中断标志位，并重置为false
false  
true
false

```

# LockSupport

### LockSupport是什么？

-   LockSupport是用来创建锁和其他同步类的基本线程阻塞原语
-   LockSupport的`park()`和`unpark()`分别用于阻塞线程和解除阻塞线程

## 线程的等待唤醒机制

### 三种让线程等待唤醒的机制

1.  方式1：使用Object中的wait()方法让线程等待，使用Object中的notify()方法唤醒线程
2.  方式2：使用JUC包中Condition的await()方法让线程等待，使用signal()方法唤醒线程
3.  方式3：LockSupport类可以阻塞当前线程以及唤醒指定被阻塞的线程

### Object类中的wait和notify方法实现线程等待和唤醒

```java
package com.atguigu.juc.prepare;

import java.util.concurrent.TimeUnit;

/**
 *
 * 要求：t1线程等待3秒钟，3秒钟后t2线程唤醒t1线程继续工作
 *
 * 1 正常程序演示
 *
 * 以下异常情况：
 * 2 wait方法和notify方法，两个都去掉同步代码块后看运行效果
 *   2.1 异常情况
 *   Exception in thread "t1" java.lang.IllegalMonitorStateException at java.lang.Object.wait(Native Method)
 *   Exception in thread "t2" java.lang.IllegalMonitorStateException at java.lang.Object.notify(Native Method)
 *   2.2 结论
 *   Object类中的wait、notify、notifyAll用于线程等待和唤醒的方法，都必须在synchronized内部执行（必须用到关键字synchronized）。
 *
 * 3 将notify放在wait方法前面
 *   3.1 程序一直无法结束
 *   3.2 结论
 *   先wait后notify、notifyall方法，等待中的线程才会被唤醒，否则无法唤醒
 */
public class LockSupportDemo
{
    public static void main(String[] args)//main方法，主线程一切程序入口
    {
        Object objectLock = new Object(); //同一把锁，类似资源类
        new Thread(() -> {
            synchronized (objectLock) {
                try {
                    objectLock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName()+"\t"+"被唤醒了");
        },"t1").start();
        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3L); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            synchronized (objectLock) {
                objectLock.notify();
            }
            //objectLock.notify();
            /*synchronized (objectLock) {
                try {
                    objectLock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }*/
        },"t2").start();
    }
}

```

> wait和notify、notifyall必须在同步代码块或同步方法中，且先wait后notify程序才能正常执行

### Condition接口中的await后signal方法实现线程的等待和唤醒

```java
package com.atguigu.juc.prepare;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
public class LockSupportDemo2
{
    public static void main(String[] args)
    {
        Lock lock = new ReentrantLock();
        Condition condition = lock.newCondition();

        new Thread(() -> {
            lock.lock();
            try
            {
                System.out.println(Thread.currentThread().getName()+"\t"+"start");
                condition.await();
                System.out.println(Thread.currentThread().getName()+"\t"+"被唤醒");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        },"t1").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3L); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            lock.lock();
            try
            {
                condition.signal();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
            System.out.println(Thread.currentThread().getName()+"\t"+"通知了");
        },"t2").start();

    }
}

```

> Condtion中的线程等待和唤醒方法之前，需要先获取锁

> 一定要先await后signal，不要反了

### Object和Condition使用等待和唤醒的限制条件

-   线程必须先持有锁，必须在锁块中（synchronized或lock）
-   必须先等待再唤醒

### LockSupport类中的park等待和unpark唤醒

-   原理
    -   LockSupport是用来创建锁和其他同步类的基本线程阻塞原语。
    -   LockSupport类使用了一种名为`Permit`（许可）的概念来做到阻塞和唤醒线程的功能， 每个线程都有一个许可(permit)，
    -   permit只有两个值1和零，默认是零。

    -   可以把许可看成是一种(0,1)信号量（Semaphore），但与 Semaphore 不同的是，许可的累加上限是1

第1列

-   阻塞
    -   park()  阻塞当前线程
    -   park(thread) 阻塞指定线程


第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_aY6WUiNCq_.png)

permit默认是零，所以一开始调用park()方法，当前线程就会阻塞，直到别的线程将当前线程的permit设置为1时，park方法会被唤醒，
然后会将permit再次设置为零并返回。

第1列

-   唤醒
    -   unpark(Thread thread)&#x20;



第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_w8VrzOKSEv.png)

调用unpark(thread)方法后，就会将thread线程的许可permit设置成1(注意多次调用unpark方法，不会累加，permit值还是1)会自动唤醒thread线程，即之前阻塞中的LockSupport.park()方法会立即返回。

-   正常使用LockSupport不需要加锁

```java
public class InterruptDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " 被阻塞");
            LockSupport.park();//一开始permit为0 阻塞当前线程
            System.out.println(Thread.currentThread().getName() + " 被唤醒");
        });
        t.start();
        TimeUnit.SECONDS.sleep(2);
        LockSupport.unpark(t);

    }
}

```

-   LockSupport的park和unpark没有顺序之分

```java
public class InterruptDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " 被阻塞");
            try {
                TimeUnit.SECONDS.sleep(2);//加延迟，让unpark先运行，唤醒线程
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            LockSupport.park();
            System.out.println(Thread.currentThread().getName() + " 被唤醒");
        });
        t.start();
        LockSupport.unpark(t);

    }
}
```
