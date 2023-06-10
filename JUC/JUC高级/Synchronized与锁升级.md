# Synchronized与锁升级

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_wWkwbwsjGD.png)

-   synchronized锁：由对象头中MarkWord的锁标志为不同而被复用及锁升级策略

## synchronized的性能变化

-   在JDK5之前只有synchronized，是操作系统级别的重量级锁，线程切换导致用户态和内核态之间的切换，当锁竞争激烈时，系统性能急剧下降。
-   JDK6开始，优化synchronized，为了减少获取锁和释放锁带来的性能消耗，引入了偏向锁和轻量级锁。
-   为什么每个java对象都可以是一把锁？
    -   每一个java对象都对应这一个Monitor（监视器）叫做ObjectMonitor，底层时C++代码实现的。Monitor依赖于操作系统底层的Mutex Lock（互斥锁）
-   java对象、锁、线程时怎样联系起来的？
    -   monitor中的owner字段会记录持有当前锁的线程id
    -   如果一个java对象被某个线程锁住，则该java对象的Mark Word字段中LockWord指向monitor的起始地址

## synchronized锁种类及升级步骤

-   多线程访问的三种情况
    -   只有一个线程来访问
    -   A、B两个线程交替访问
    -   竞争激烈，多个线程同时访问
-   synchronized锁升级主要是靠锁对象中的对象头中的MarkWord的锁标志位和偏向锁标记位的状态改变来实现的。

### 无锁

-   线程没有锁的竞争

```java
 //对象头的分析
 public static void main(String[] args) {
        Object object = new Object();
        System.out.println("十进制："+object.hashCode());
        System.out.println("十六进制："+Integer.toHexString(object.hashCode()));
        System.out.println("二进制："+Integer.toBinaryString(object.hashCode()));
        System.out.println(ClassLayout.parseInstance(object).toPrintable());
    }
    
    //无锁状态对象头分析
997110508
3b6eb2ec
111011011011101011001011101100
# WARNING: Unable to get Instrumentation. Dynamic Attach failed. You may add this JAR as -javaagent manually, or supply -Djdk.attach.allowAttachSelf
java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 ec b2 6e (00000001 11101100 10110010 01101110) (1857219585)
      4     4        (object header)                           3b 00 00 00 (00111011 00000000 00000000 00000000) (59)
      8     4        (object header)                           00 10 00 00 (00000000 00010000 00000000 00000000) (4096)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

//对象头的二进制数据是倒过来的，
//00000001  最后一段的最后三位为001表示无锁
//中间31位位哈希
```

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_8Yb-2Oikgv.png)

### 偏向锁

-   **当一段同步代码被同一个线程多次访问，那么在后续的访问中就会自动获得锁。**
-   HotSpot的作者发现，在多线程的情况下，锁不仅不存在锁竞争的情况，还出现了同一把锁被同一个线程多次获得的情况，偏向锁就是在这种情况下出现的，主要是为了提高一个线程访问同步代码的性能
-   对象头的标记位图（通过CAS方式修改markword中的线程ID）

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_h3KsN5dJhj.png)

-   偏向锁的说明

```java
理论落地：
      在实际应用运行过程中发现，“锁总是同一个线程持有，很少发生竞争”，也就是说锁总是被第一个占用他的线程拥有，这个线程就是锁的偏向线程。
      那么只需要在锁第一次被拥有的时候，记录下偏向线程ID。这样偏向线程就一直持有着锁(后续这个线程进入和退出这段加了同步锁的代码块时，不需要再次加锁和释放锁。而是直接比较对象头里面是否存储了指向当前线程的偏向锁)。
如果相等表示偏向锁是偏向于当前线程的，就不需要再尝试获得锁了，直到竞争发生才释放锁。以后每次同步，检查锁的偏向线程ID与当前线程ID是否一致，如果一致直接进入同步。无需每次加锁解锁都去CAS更新对象头。如果自始至终使用锁的线程只有一个，很明显偏向锁几乎没有额外开销，性能极高。
      假如不一致意味着发生了竞争，锁已经不是总是偏向于同一个线程了，这时候可能需要升级变为轻量级锁，才能保证线程间公平竞争锁。偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程是不会主动释放偏向锁的。
 
技术实现：
一个synchronized方法被一个线程抢到了锁时，那这个方法所在的对象就会在其所在的Mark Word中将偏向锁修改状态位，同时还
会有占用前54位来存储线程指针作为标识。若该线程再次访问同一个synchronized方法时，该线程只需去对象头的Mark Word 中去判断一下是否有偏向锁指向本身的ID，无需再进入 Monitor 去竞争对象了。
```

-   对象头的修改需要CAS

#### 偏向锁的JVM命令

-   java -XX:+PrintFlagsInitial | grep BiasedLock\*

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_0IyTHWRjof.png)

```java
* 实际上偏向锁在JDK1.6之后是默认开启的，但是启动时间有延迟，
* 所以需要添加参数-XX:BiasedLockingStartupDelay=0，让其在程序启动时立刻启动。
*
* 开启偏向锁：
* -XX:+UseBiasedLocking -XX:BiasedLockingStartupDelay=0
*
* 关闭偏向锁：关闭之后程序默认会直接进入------------------------------------------>>>>>>>>   轻量级锁状态。
* -XX:-UseBiasedLocking
```

### JOL对象头分析

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_JuNW7xll1j.png)

### 偏向锁的撤销

-   偏向锁使用了一种只有出现竞争的时候才会释放锁的机制，只有当其他线程竞争锁的时候，持有偏向锁的线程才会被撤销，撤销需要等到安全点才会执行，同时检查持有偏向锁的线程是否还在执行。
-   第一种情况：持有偏向锁的线程在synchronized同步代码中时，他还在执行没有结束，那么偏向锁就会被撤销，升级为轻量级锁，此时轻量级锁由当前线程持有，继续执行同步代码，其他竞争线程，进入自选等待回去轻量级锁。
-   第二种情况：持有偏向锁的线程，已经执行到synchronized同步代码块外，那么会设置锁对象的对象头为无锁状态，重新偏向。

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_Zgf9q7PXZL.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_20dqD554fg.png)

### 轻量级锁

-   当出现线程竞争，但是竞争不激烈，锁竞争的时间极短
-   本质是自旋锁
-   轻量级锁主要是提高线程近乎交替执行同步代码时的性能，不轻易使用操作系统底层互斥锁
-   升级时机： 当关闭偏向锁功能或多线程竞争偏向锁会导致偏向锁升级为轻量级锁

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_tI_RTqUsgx.png)

#### JOL展示

-   如果关闭偏向锁，就可以直接进入轻量级锁（为了方便进入轻量级锁演示）
-   `-XX:-UseBiasedLocking`

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_o5CSHqoVSc.png)

-   当线程**自旋达到一定的程度和次数**的时候，轻量级锁就会升级为重量级锁
    -   JDK6之前可以指定自旋次数进行锁升级（不要使用）
    -   JDK6之后采用自适应策略决定锁是否升级
-   偏向锁和轻量级锁的区别
    -   偏向锁退出同步代码时不会释放锁，轻量级锁退出同步代码会释放锁
    -   偏向锁：一个线程多次持有同一个锁对象，轻量级锁：出现轻微的锁竞争情况

### 重量级锁

-   有大量线程参与锁竞争，冲突性很高

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_zuqKX8G1Mz.png)

#### JOL对象头信息展示

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_UHksudFki6.png)

## 各种锁的对比

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image__MHtTl8WNA.png)

```java
synchronized在修饰方法和代码块在字节码上实现方式有很大差异，但是内部实现还是基于对象头的MarkWord来实现的。
JDK1.6之前synchronized使用的是重量级锁，JDK1.6之后进行了优化，拥有了无锁->偏向锁->轻量级锁->重量级锁的升级过程，而不是无论什么情况都使用重量级锁。
 
偏向锁:适用于单线程适用的情况，在不存在锁竞争的时候进入同步方法/代码块则使用偏向锁。
轻量级锁：适用于竞争较不激烈的情况(这和乐观锁的使用范围类似)， 存在竞争时升级为轻量级锁，轻量级锁采用的是自旋锁，如果同步方法/代码块执行时间很短的话，采用轻量级锁虽然会占用cpu资源但是相对比使用重量级锁还是更高效。
重量级锁：适用于竞争激烈的情况，如果同步方法/代码块执行时间很长，那么使用轻量级锁自旋带来的性能消耗就比使用重量级锁更严重，这时候就需要升级为重量级锁。
```

### JIT对锁的优化

-   Just In Time Compiler，一般翻译为即时编译器
-   锁消除：经过逃逸分析之后发现一个对象无法逃逸出方法，如果这个对象作为锁对象，那么可以去掉这个锁

```java
   public void m1()
    {
        //锁消除,JIT会无视它，synchronized(对象锁)不存在了。不正常的
        Object o = new Object();

        synchronized (o)
        {
            System.out.println("-----hello LockClearUPDemo"+"\t"+o.hashCode()+"\t"+objectLock.hashCode());
        }
    }

```

-   锁粗化：多次加锁，变为一次加锁

```java

/**
 * 锁粗化
 * 假如方法中首尾相接，前后相邻的都是同一个锁对象，那JIT编译器就会把这几个synchronized块合并成一个大块，
 * 加粗加大范围，一次申请锁使用即可，避免次次的申请和释放锁，提升了性能
 */
public class LockBigDemo
{
    static Object objectLock = new Object();


    public static void main(String[] args)
    {
        new Thread(() -> {
            synchronized (objectLock) {
                System.out.println("11111");
            }
            synchronized (objectLock) {
                System.out.println("22222");
            }
            synchronized (objectLock) {
                System.out.println("33333");
            }
        },"a").start();

        new Thread(() -> {
            synchronized (objectLock) {
                System.out.println("44444");
            }
            synchronized (objectLock) {
                System.out.println("55555");
            }
            synchronized (objectLock) {
                System.out.println("66666");
            }
        },"b").start();

    }
}
 
 

```
