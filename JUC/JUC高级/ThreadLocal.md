# ThreadLocal

### ThreadLocal是什么

-   ThreadLocal提供线程的局部变量，这个变量与正常的变量不同，因为线程在访问自己的ThreadLocal时，都有自己的，独立初始化的变量副本。ThreadLocal通常时实例中静态私有的字段。使用ThreadLocal的目的是为了将某些状态与线程关联。&#x20;

### ThreadLocal能干嘛

-   实现每一个线程都有自己专属的本地变量副本
-   主要解决了让每个线程绑定自己的值，通过使用get()和set()方法，获取默认值或将其值更改为当前线程所存的副本的值从而避免了线程安全问题。

ThreaLocalAPI

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_nPOuqdon4d.png)

-   阿里规范

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_4zkuQs12vV.png)

### ThreadLocal源码

-   Thread，ThreadLocal，ThreadLocalMap 的关系

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_-SagFhG5Jp.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_4097yxprpa.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_393Dyzponm.png)

### ThreadLocal内存泄漏

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_rrKLlmBs8H.png)

-   什么时内存泄漏
    -   就是一个永远不会被使用的对象没有被垃圾回收
    -   一个对象的存活时间远远超过了使用它的时间
-   ThreadLocal出现内存泄漏的原因是使用了弱引用作为ThreadLocalMap的key

#### 为什么要使用弱引用，不使用会怎么样？

```java
public void function01()
{
    ThreadLocal tl = new ThreadLocal<Integer>();    //line1
    tl.set(2021);                                   //line2
    tl.get();                                       //line3
}

```

> line1创建一个ThreadLocal对象，t1强引用指向他
> line2设置值，会在当前ThreadLocal中的ThreadLocalMap中创建一个Entry对象，key为当前ThreadLocal的弱引用指向当前ThreadLocal，value为设置的值

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_wFAvCAb3zS.png)

-   为什么要使用弱引用
    -   当function01结束时，强引用t1就没有了，但是ThreadLocalMap中还有引用指向当前ThreadLocal对象。
    -   如果ThreadLocalMap中的引用为强引用，那么当前ThreadLocal对象就不能回收，出现内存泄漏
    -   如果ThreadLocalMap中的引用为弱引用，会减少出现内存泄漏的概率（k=null，v还没有被回收）使用弱引用在方法执行结束时ThreadLocal会被回收，且Entry中的key指向null。
    -   **此后我们调用get,set或remove方法时，就会尝试删除key为null的entry，可以释放value对象所占用的内存。**
-   使用弱引用就万事大吉了吗？
    -   当我们为ThreadLocal赋值时，其实是将Entry（弱引用k，value）放入ThreadLocalMap中，Entry中的key为弱引用，当外部的ThreadLocal强引用被置为null时，系统GC时，根据可达性分析算法，没有一条引用链能够引用到这个ThreadLcoal，他就会被回收，这样一来ThreadLocalMAp中就会存在一个key为null的Entry，就没有办法访问这些key为null的Entry的value，如果线程迟迟不结束，那么这些key为null的Entry的value就存在一条强引用链：Thread Ref → Thread →Thread'Local'Map → Entry →value永远无法回收，造成内存泄漏。
    -   当然，如果当前thread运行结束，threadLocal，threadLocalMap,Entry没有引用链可达，在垃圾回收的时候都会被系统进行回收。
    -   但在实际使用中我们有时候会用线程池去维护我们的线程，比如在Executors.newFixedThreadPool()时创建线程的时候，为了复用线程是不会结束的，所以threadLocal内存泄漏就值得我们小心
-   set、get方法会去检查所有键为null的Entry对象

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_lc7vIEOUe7.png)
