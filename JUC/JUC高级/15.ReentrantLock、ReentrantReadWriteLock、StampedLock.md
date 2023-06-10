# ReentrantLock、ReentrantReadWriteLock、StampedLock

-   无锁→独占锁→读写锁→邮戳锁

## ReentrantReadWriteLock

-   读写锁被定义为：一个资源可以被多个读线程共享或者被一个写线程写，不能同时出现读写线程
-   只有在读多写少情境之下，读写锁才具有较高的性能体现。但是可能出现写锁饥饿的情况

#### 特点

-   可重入
-   读写分离
-   从写锁→读锁，ReentrantReadWriteLock可以降级
    -   锁的严苛程度变强叫做升级，反之叫做降级
        ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_88_IiJWBdg.png)
    -   锁降级是为了让当前线程感知到数据的变化，目的是保证数据可见性
    -   **如果一个线程占有了写锁，在不释放写锁的情况下，它还能占有读锁，即写锁降级为读锁。**（同一个线程）
    -   重入还允许通过获取写入锁定，然后读取锁然后释放写锁从写锁到读取锁, 但是，从读锁定升级到写锁是不可能的。
    -   在ReentrantReadWriteLock中，当读锁被使用时，如果有线程尝试获取写锁，该写线程会被**阻塞**。所以，需要释放所有读锁，才可获取写锁，

	```java

    public static void main(String[] args)
    {
        ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

        ReentrantReadWriteLock.ReadLock readLock = readWriteLock.readLock();
        ReentrantReadWriteLock.WriteLock writeLock = readWriteLock.writeLock();

        writeLock.lock();
        System.out.println("-------正在写入");


        readLock.lock();
        System.out.println("-------正在读取");

        writeLock.unlock();
    }
```

-   小结
    -   写锁和读锁是互斥的（**这里的互斥是指线程间的互斥**，当前线程可以获取到写锁又获取到读锁，但是获取到了读锁不能继续获取写锁），这是因为读写锁要保持写操作的可见性。
    -   读写锁之间悲观的
    -   读锁过多写锁饥饿
-   面试题：有没有比读写锁更快的锁？
    -   邮戳锁：他的读写是悲观的，允许在读的过程中写，那么会有一定的概率导致数据的不一致，如果在度的过程中检测到写操作，数据不一致了，重新读一遍即可。

### 邮戳锁StampedLock

-   StampedLock是JDK1.8新增的一个读写锁，也是对1.5的ReentrantReadWriteLock的优化
-   邮戳锁也叫票据锁
-   stamp（戳记，long类型）：代表锁的状态，0表示获取锁失败；在释放锁或者转换锁的时候，需要传入最初的stamp值。

#### 锁饥饿问题

```java
ReentrantReadWriteLock实现了读写分离，但是一旦读操作比较多的时候，想要获取写锁就变得比较困难了，
假如当前1000个线程，999个读，1个写，有可能999个读取线程长时间抢到了锁，那1个写线程就悲剧了 
因为当前有可能会一直存在读锁，而无法获得写锁，根本没机会写，o(╥﹏╥)o
```

-   如何缓解锁饥饿问题
    -   使用公平锁在一定程度上可以缓解锁饥饿的问题，但是会牺牲系统的吞吐量；
    -   使用StampedLock的乐观读锁
        ```java
        ReentrantReadWriteLock
        允许多个线程同时读，但是只允许一个线程写，在线程获取到写锁的时候，其他写操作和读操作都会处于阻塞状态，
        读锁和写锁也是互斥的，所以在读的时候是不允许写的，读写锁比传统的synchronized速度要快很多，
        原因就是在于ReentrantReadWriteLock支持读并发
         
         
        StampedLock横空出世
        ReentrantReadWriteLock的读锁被占用的时候，其他线程尝试获取写锁的时候会被阻塞。
        但是，StampedLock采取乐观获取锁后，其他线程尝试获取写锁时不会被阻塞，这其实是对读锁的优化，
        所以，在获取乐观读锁后，还需要对结果进行校验。
        ```

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_97Y7eCccEh.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_f8nHUwpZI3.png)

```java
public class StampedLockDemo
{
    static int number = 37;
    static StampedLock stampedLock = new StampedLock();

    public void write()
    {
        long stamp = stampedLock.writeLock();
        System.out.println(Thread.currentThread().getName()+"\t"+"=====写线程准备修改");
        try
        {
            number = number + 13;
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            stampedLock.unlockWrite(stamp);
        }
        System.out.println(Thread.currentThread().getName()+"\t"+"=====写线程结束修改");
    }

    //悲观读
    public void read()
    {
        long stamp = stampedLock.readLock();
        System.out.println(Thread.currentThread().getName()+"\t come in readlock block,4 seconds continue...");
        //暂停几秒钟线程
        for (int i = 0; i <4 ; i++) {
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println(Thread.currentThread().getName()+"\t 正在读取中......");
        }
        try
        {
            int result = number;
            System.out.println(Thread.currentThread().getName()+"\t"+" 获得成员变量值result：" + result);
            System.out.println("写线程没有修改值，因为 stampedLock.readLock()读的时候，不可以写，读写互斥");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            stampedLock.unlockRead(stamp);
        }
    }

    //乐观读
    public void tryOptimisticRead()
    {
        long stamp = stampedLock.tryOptimisticRead();
        int result = number;
        //间隔4秒钟，我们很乐观的认为没有其他线程修改过number值，实际靠判断。
        System.out.println("4秒前stampedLock.validate值(true无修改，false有修改)"+"\t"+stampedLock.validate(stamp));
        for (int i = 1; i <=4 ; i++) {
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println(Thread.currentThread().getName()+"\t 正在读取中......"+i+
                    "秒后stampedLock.validate值(true无修改，false有修改)"+"\t"
                    +stampedLock.validate(stamp));
        }
        if(!stampedLock.validate(stamp)) {
            System.out.println("有人动过--------存在写操作！");
            stamp = stampedLock.readLock();
            try {
                System.out.println("从乐观读 升级为 悲观读");
                result = number;
                System.out.println("重新悲观读锁通过获取到的成员变量值result：" + result);
            }catch (Exception e){
                e.printStackTrace();
            }finally {
                stampedLock.unlockRead(stamp);
            }
        }
        System.out.println(Thread.currentThread().getName()+"\t finally value: "+result);
    }

    public static void main(String[] args)
    {
        StampedLockDemo resource = new StampedLockDemo();

        new Thread(() -> {
            resource.read();
            //resource.tryOptimisticRead();
        },"readThread").start();

        // 2秒钟时乐观读失败，6秒钟乐观读取成功resource.tryOptimisticRead();，修改切换演示
        //try { TimeUnit.SECONDS.sleep(6); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            resource.write();
        },"writeThread").start();
    }
}


 
 

```
