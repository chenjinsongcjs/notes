# JUC基础部分

## 1、什么是JUC

-   JUC指java类库中的三个包
    -   java.util.concurrent
    -   java.util.concurrent.atomic
    -   java.util.concurrent.locks

## 2、线程和进程

-   线程：程序执行和调度的基本单位，线程隶属于进程。
-   进程：资源分配的单位，一个进程可以有一个和多个线程。
-   java程序的线程与操作系统的线程一一映射，java创建线程使用的是本地方法。private native void start0();
-   并发和并行
    -   并发：CPU在同一时间段内执行多个任务。
    -   并行：CPU在同一时间刻内执行多个任务。
-   线程的几个状态

```java
public enum State {
        NEW,
        RUNNABLE,
        BLOCKED,
        WAITING,
        TIMED_WAITING,
        TERMINATED;
    }
```

-   wait和sleep的区别
    -   来自不同的类
        -   wait-->Object
        -   sleep-->Thread
    -   关于锁的释放
        -   wait会释放锁
        -   sleep不会释放锁
    -   使用范围
        -   wait只能在同步代码块中使用
        -   sleep可以在任何地方使用

## 3、Lock锁

-   传统的synchronized

```java
public class Lock {
    public static void main(String[] args) {
       Ticket t = new Ticket(); 
        new Thread(()->{
            for (int i = 0; i < 20 ; i++){
                t.sale();
            }
        },"A").start();
        new Thread(()->{
            for (int i = 0; i < 20 ; i++){
                t.sale();
            }
        },"B").start();
        new Thread(()->{
            for (int i = 0; i < 20 ; i++){
                t.sale();
            }
        },"C").start();
    }
}

class Ticket{
    private int num = 30;

    public synchronized void sale(){
        if(num > 0)
            System.out.println(""+Thread.currentThread().getName()+"-->"+num+"-->"+(num--));

    }

}
```

-   Lock接口

```java
public interface Lock {
  //加锁
    void lock();
    //可中断锁
    void lockInterruptibly() throws InterruptedException;
    //尝试获得锁
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    //解锁
    void unlock();
    //创建Condition对象
    Condition newCondition();
}

class Ticket {
    private int num = 30;
    //lock对象
    Lock lock = new ReentrantLock();
    public  void sale() {
        //上锁
        lock.lock();
        try {
            if (num > 0)
                System.out.println("" + Thread.currentThread().getName() + "-->" + num + "-->" + (num--));
        } finally {
            //解锁
        lock.unlock();
        }

    }
 //无参构造默认锁为非公平锁
 public ReentrantLock() {
        sync = new NonfairSync();
    }
 //通过有参构造器可以选择公平锁和非公平锁  
 public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

```

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622638828892-c8d8b564-ad70-42ae-aa09-657c5ec4e7a4.png#align=left\&display=inline\&height=100\&margin=\[object Object]\&name=image.png\&originHeight=199\&originWidth=1453\&size=24487\&status=done\&style=none\&width=726.5> "image.png")

-   Synchronized和Lock的区别
    -   Synchronized是java的关键字，Lock是java类
    -   Synchronized无法判断锁的状态，Lock可以判断锁的状态
    -   Synchronized可以自动释放锁，Lock必须手动释放锁，不然会导致死锁
    -   Synchronized只能是非公平锁，Lock可以设置为公平锁和非公平锁
    -   Synchronized适合锁少量的代码，Lock适合锁大量的同步代码

## 4、生产者和消费者问题

-   Synchronized版

```java
public class Demo {
    public static void main(String[] args) {
        Product product = new Product();
        new Thread(()->{
            for (int i = 0; i < 10 ; i++) {
                Thread.currentThread().setName("生产者-"+i);
                try {
                    product.incrment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"生产者").start();
        new Thread(()->{
            for (int i = 0; i < 10 ; i++) {
                Thread.currentThread().setName("消费者-"+i);
                try {
                    product.decrment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"消费者").start();
    }
}

class Product{
    private int num = 0;

    public synchronized void incrment() throws InterruptedException {
        while(num != 0){//使用while避免虚假唤醒
            System.out.println(Thread.currentThread().getName()+"wait...");
            this.wait();//当有产品的时候等待
        }
        num++;//没有产品就生产
        System.out.println(Thread.currentThread().getName()+"生产产品数量："+num);
        this.notifyAll();//唤醒消费者
    }
    public synchronized void decrment() throws InterruptedException {
        while(num == 0){//没有产品就等待
            System.out.println(Thread.currentThread().getName()+"wait...");
            this.wait();
        }
        System.out.println(Thread.currentThread().getName()+"消费产品数量："+num);
        num--;//消费产品
        this.notifyAll();//唤醒生产者
    }
}
```

-   JUC版的生产者消费者问题

```java
public class Demo {
    public static void main(String[] args) {
        Product product = new Product();
        new Thread(()->{
            for (int i = 0; i < 10 ; i++) {
                Thread.currentThread().setName("生产者A-"+i);
                try {
                    product.incrment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"生产者").start();
        new Thread(()->{
            for (int i = 0; i < 10 ; i++) {
                Thread.currentThread().setName("消费者A-"+i);
                try {
                    product.decrment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"消费者").start();
        new Thread(()->{
            for (int i = 0; i < 10 ; i++) {
                Thread.currentThread().setName("生产者B-"+i);
                try {
                    product.incrment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"生产者").start();
        new Thread(()->{
            for (int i = 0; i < 10 ; i++) {
                Thread.currentThread().setName("消费者B-"+i);
                try {
                    product.decrment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"消费者").start();


    }
}

class Product{
    private int num = 0;
    //锁住当前对象
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();
    public void incrment() throws InterruptedException {
        lock.lock();
        try {
            while( num != 0){
                System.out.println(Thread.currentThread().getName()+" wait...");
                condition.await();
            }
            num++;
            System.out.println(Thread.currentThread().getName()+" num-->"+num);
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }
    public void decrment() throws InterruptedException {
        lock.lock();
        try {
            while(num == 0){
                System.out.println(Thread.currentThread().getName()+" wait...");
                condition.await();
            }
            num--;
            System.out.println(Thread.currentThread().getName()+" num-->"+num);
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }
}
```

-   Condition 精准唤醒

```java
public class Demo {
    public static void main(String[] args) {
        Product product = new Product();
        new Thread(()->{
            for (int i = 0; i < 10 ; i++) {
                Thread.currentThread().setName("生产者A-"+i);
                try {
                    product.printA();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"生产者").start();
        new Thread(()->{
            for (int i = 0; i < 10 ; i++) {
                Thread.currentThread().setName("消费者A-"+i);
                try {
                    product.printB();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"消费者").start();
        new Thread(()->{
            for (int i = 0; i < 10 ; i++) {
                Thread.currentThread().setName("生产者B-"+i);
                try {
                    product.printC();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"生产者").start();



    }
}

class Product{
    private int num = 1;
    Lock lock = new ReentrantLock();
    //精准唤醒指定线程
    Condition condition1 = lock.newCondition();
    Condition condition2 = lock.newCondition();
    Condition condition3 = lock.newCondition();
    public void printA() throws InterruptedException {
        lock.lock();
        try {
            while( num != 1){
                System.out.println(Thread.currentThread().getName()+" wait...");
                condition1.await();
            }
            num = 2;
            System.out.println(Thread.currentThread().getName()+" num-->"+num);
            condition2.signal();
        } finally {
            lock.unlock();
        }
    }
    public void printB() throws InterruptedException {
        lock.lock();
        try {
            while(num != 2){
                System.out.println(Thread.currentThread().getName()+" wait...");
                condition2.await();
            }
            num = 3;
            System.out.println(Thread.currentThread().getName()+" num-->"+num);
            condition3.signal();
        } finally {
            lock.unlock();
        }
    }
    public void printC() throws InterruptedException {
        lock.lock();
        try {
            while(num != 3){
                System.out.println(Thread.currentThread().getName()+" wait...");
                condition3.await();
            }
            num = 1;
            System.out.println(Thread.currentThread().getName()+" num-->"+num);
            condition1.signal();
        } finally {
            lock.unlock();
        }
    }
}

```

## 5、8锁现象

```java
//1.synchronized 锁定当前方法的调用者，谁先拿到锁谁执行
//2.普通方法不受锁的影响，就算锁被其他线程持有，当前线程还能执行普通方法
//3.非静态方法，两个对象，两个调用者，就有两把锁线程互不影响
//4.两个静态方法，只有一个对象，静态方法锁定当前Class对象(只有一个)，谁先获得锁谁执行
//5.两个静态方法，两个对象，静态方法锁定当前Class对象(只有一个)，谁先获得锁谁执行
//6.一个静态同步方法，一个普通同步方法，一个是Class锁，一个是对象锁，是两把不同的锁，互不影响

主要是看锁的是谁，谁有锁
```

## 6、集合类不安全

-   ConcurrentModificationException  线程不安全可能导致的异常
-   List不安全

```java
//解决方法：
 public static void main(String[] args) {
        //集合不安全解决方法
        //使用线程安全的Vector类
        //1.使用Collections工具类
        Collections.synchronizedList(new ArrayList<>());
        //使用CopyOnWriteArrayList
        CopyOnWriteArrayList<String> arrayList = new CopyOnWriteArrayList<>();
       //底层添加操作
//        public boolean add(E e) {
//            final ReentrantLock lock = this.lock;
//            lock.lock();
//            try {
//                Object[] elements = getArray();
//                int len = elements.length;
//                Object[] newElements = Arrays.copyOf(elements, len + 1);
//                newElements[len] = e;
//                setArray(newElements);
//                return true;
//            } finally {
//                lock.unlock();
//            }
//        }
        arrayList.add("123");
    }
```

-   set不安全

```java
public static void main(String[] args) {
        //set不安全
        //1。使用Collections工具类
        Collections.synchronizedSet(new HashSet<>());
        //2.使用CopyOnWriteArraySet
        CopyOnWriteArraySet<String> arraySet = new CopyOnWriteArraySet<>();
      //底层添加操作
       /* private boolean addIfAbsent(E e, Object[] snapshot) {
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                Object[] current = getArray();
                int len = current.length;
                if (snapshot != current) {
                    // Optimize for lost race to another addXXX operation
                    int common = Math.min(snapshot.length, len);
                    for (int i = 0; i < common; i++)
                        if (current[i] != snapshot[i] && eq(e, current[i]))
                            return false;
                    if (indexOf(e, current, common, len) >= 0)
                        return false;
                }
                Object[] newElements = Arrays.copyOf(current, len + 1);
                newElements[len] = e;
                setArray(newElements);
                return true;
            } finally {
                lock.unlock();
            }
        }*/
        arraySet.add("123");
    }

  //HashSet底层是一个HashMap
public HashSet() {
        map = new HashMap<>();
    }
//添加操作
public boolean add(E e) {
        return map.put(e, PRESENT)==null;//PRESENT是一个常量 private static final Object PRESENT = new Object();
    }
```

-   map不安全

```java
  public static void main(String[] args) {
        ConcurrentHashMap<String, Object> hashMap = new ConcurrentHashMap<>();
        hashMap.put("k","v");
    }

//put的底层实现

final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```

## 7、Callable

```java
//callable与Runnable的不同
//1.Callable的call方法有返回值和能抛出异常
//2.Runnable的run方法没有返回值，不能抛出异常

@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
public class Demo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> futureTask = new FutureTask<>(new ThreadCall());
        new Thread(futureTask).start();//FutureTask的结果会被缓存，效率较高
        String s = futureTask.get();//阻塞式 获取call的返回值，一般放在最后
    }
}

class ThreadCall implements Callable<String>{

    @Override
    public String call() throws Exception {
        return "hello";
    }
}
```

## 8、常用辅助类

### 8.1 CountDownLatch

```java
//保证线程的一组操作完成之后其他线程才能执行 

public static void main(String[] args) throws InterruptedException {
        //设置初始值，在初始值被减为0之前所有的其他线程处于等待状态，
        CountDownLatch countDownLatch = new CountDownLatch(5);
        for (int i = 0; i < 5 ; i++) {
            final int temp = i;
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"-->"+ temp);
                countDownLatch.countDown();//每调用一次初始值减一
            }).start();
        }
        countDownLatch.await();//等待初始值减为0，初始值减为0，放行后面的操作
        System.out.println("其他线程可以继续执行了。。。。");
    }
```

### 8.2 CyclicBarrier

```java
//循环直到屏障点，才执行其他线程 
public CyclicBarrier(int parties, Runnable barrierAction) {}
public CyclicBarrier(int parties) {}

public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
            System.out.println("到达屏障点");
        });
        for (int i = 0; i < 7 ; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+" 执行");
                try {
                    cyclicBarrier.await();//等待直到到达屏障点
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
```

-   Semaphore

```java
//信号量主要用于限流使用， 多个共享资源互斥使用，控制线程数量

public static void main(String[] args) {
        //信号量，表示只能有3个线程运行，其他线程要等待，直到有线程释放
        Semaphore semaphore = new Semaphore(3);
        for (int i = 0; i < 10 ; i++) {
            final int temp = i;
            new Thread(()->{
                try {
                    semaphore.acquire();//获取执行位置，如果执行位置已满，其他线程处于等待状态
                    System.out.println(Thread.currentThread().getName()+" git location");
                    TimeUnit.SECONDS.sleep(3);
                    System.out.println(Thread.currentThread().getName()+" go out");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();//释放执行位置，其他线程可以抢占执行位置
                }
            }).start();
        }
    }
```

## 9、读写锁

-   ReadWriteLock：保证读的时候可以多线程读，写的时候只有一个线程在写

```java
//读锁：共享锁，可以允许多个线程读
//写锁：独占锁，只允许一个线程写





public class Demo {
    public static void main(String[] args) {

        CacheLock cacheLock = new CacheLock();
        for (int i = 0; i < 5 ; i++) {
            final int temp = i;
            new Thread(()->{
                cacheLock.write(""+temp,""+temp);
            }).start();
        }
        for (int i = 0; i < 5 ; i++) {
            final int temp = i;
            new Thread(()->{
                System.out.println(cacheLock.read("" + temp));
            }).start();
        }

    }
}

class CacheLock{
    private volatile Map<String,String> map = new HashMap<>();
    ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public String read(String key){
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+ " 开始读.......");
            String s = map.get(key);
            System.out.println(Thread.currentThread().getName()+ " 读结束.......");
            return s;
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
    public void write(String key,String value){
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+ " 写开始.....");
            map.put(key,value);
            System.out.println(Thread.currentThread().getName()+ " 写结束.....");
        } finally {
            readWriteLock.writeLock().unlock();
        }
    }
}
```

## 10、阻塞队列

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622652435166-601da532-c9e6-4e16-bf6b-8395c4309f89.png#align=left\&display=inline\&height=563\&margin=\[object Object]\&name=image.png\&originHeight=1126\&originWidth=1527\&size=281527\&status=done\&style=none\&width=763.5> "image.png")

-   当写入的时候，如果队列满了，就会进入阻塞状态
-   当出队的时候，当队列空了，就会进入阻塞状态
-   一般在线程池或者多线程并发处理时使用
-   阻塞队列添加和删除的四组API
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622652700578-7c59c297-5060-440b-940e-17832df2103d.png#align=left\&display=inline\&height=170\&margin=\[object Object]\&name=image.png\&originHeight=340\&originWidth=1479\&size=71328\&status=done\&style=none\&width=739.5> "image.png")

```java
//有返回值，会抛出异常 
public static void main(String[] args) {
        ArrayBlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(arrayBlockingQueue.add("1"));
        System.out.println(arrayBlockingQueue.add("2"));
        System.out.println(arrayBlockingQueue.add("3"));
//        System.out.println(arrayBlockingQueue.add("4"));
        System.out.println(arrayBlockingQueue.remove());
        System.out.println(arrayBlockingQueue.remove());
        System.out.println(arrayBlockingQueue.remove());
//        System.out.println(arrayBlockingQueue.remove());

    }
//有返回值不会抛出异常
  public static void main(String[] args) {
      ArrayBlockingQueue<Integer> arrayBlockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(arrayBlockingQueue.offer(1));
        System.out.println(arrayBlockingQueue.offer(2));
        System.out.println(arrayBlockingQueue.offer(3));
        System.out.println(arrayBlockingQueue.offer(4));
        System.out.println("==========================");
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll());

    }
//超过容量或者容量为空会一直阻塞
public static void main(String[] args) throws InterruptedException {
      ArrayBlockingQueue<Integer> arrayBlockingQueue = new ArrayBlockingQueue<>(3);
       arrayBlockingQueue.put(1);
       arrayBlockingQueue.put(2);
       arrayBlockingQueue.put(3);
//       arrayBlockingQueue.put(4);
        System.out.println(arrayBlockingQueue.take());
        System.out.println(arrayBlockingQueue.take());
        System.out.println(arrayBlockingQueue.take());
        System.out.println(arrayBlockingQueue.take());
    }
//带有参数的offer和poll在超过指定时间之后会停止
 public static void main(String[] args) throws InterruptedException {
      ArrayBlockingQueue<Integer> arrayBlockingQueue = new ArrayBlockingQueue<>(3);
      arrayBlockingQueue.offer(1);
      arrayBlockingQueue.offer(1);
      arrayBlockingQueue.offer(1);
      arrayBlockingQueue.offer(1,3, TimeUnit.SECONDS);
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll(3,TimeUnit.SECONDS));
    }
```

-   SynchronousQueue 同步队列
    -   同步队列没有容量，只能存储一个元素，元素进去之后，必须取出才能存放下一个元素。

	```java
  public static void main(String[] args) throws InterruptedException {
        SynchronousQueue<String> strings = new SynchronousQueue<>();
        strings.put("1");
//        System.out.println(strings.take());
    }
```

## 11、线程池

-   线程池掌握三大方法、七大参数、四大拒绝策略
-   池化技术：提前将资源准备好放入池中，需要的时候从池中取出，用完之后放回池中。
-   线程池的好处：
    -   降低资源的消耗
    -   提高响应速度
    -   方便管理
    -   可以实现线程复用，可以控制最大并发数，管理线程。
-   线程池三大方法

```java
//线程池中只有一个线程
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
//线程池中可以有指定个数线程
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
//线程池中初始没有线程，最大线程数为Integer.MAX_VALUE
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

 public static void main(String[] args) {
//        ExecutorService pool = Executors.newSingleThreadExecutor();
//        ExecutorService pool = Executors.newFixedThreadPool(3);
        ExecutorService pool = Executors.newCachedThreadPool();
        try {
            for (int i = 0; i <10 ; i++) {
                //执行线程 void execute(Runnable command);
                pool.execute(()->{
                    System.out.println(Thread.currentThread().getName());
                });
            }
        } finally {
            //使用完线程池之后关闭线程池。
            pool.shutdown();
        }


    }
```

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622656599295-95087164-4218-4699-9896-6358e2670c10.png#align=left\&display=inline\&height=234\&margin=\[object Object]\&name=image.png\&originHeight=468\&originWidth=1454\&size=343567\&status=done\&style=none\&width=727> "image.png")

-   七大参数

```java
public ThreadPoolExecutor(int corePoolSize,//核心线程数
                              int maximumPoolSize,//最大线程数
                              long keepAliveTime,//线程存活时间
                              TimeUnit unit,//存活时间单位
                              BlockingQueue<Runnable> workQueue,//阻塞队列
                              ThreadFactory threadFactory,//线程工厂，一般不动
                              RejectedExecutionHandler handler//当线程池不能提供服务时的拒绝策略
                              ) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622657094810-4bac9ea4-82dd-4413-8f0a-8ff9152930a1.png#align=left\&display=inline\&height=421\&margin=\[object Object]\&name=image.png\&originHeight=841\&originWidth=1428\&size=129181\&status=done\&style=none\&width=714> "image.png")

-   当阻塞队列和核心线程数满的时候会触发最大线程工作
-   当阻塞队列和最大线程数满的时候会触发拒绝策略
-   四种拒绝策略

```java
//当线程池不能提供服务时，由当前线程处理  
public static class CallerRunsPolicy implements RejectedExecutionHandler {
        public CallerRunsPolicy() { }
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                r.run();
            }
        }
    }
//当线程池不能提供服务时，抛出异常
public static class AbortPolicy implements RejectedExecutionHandler {
        public AbortPolicy() { }
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("Task " + r.toString() +
                                                 " rejected from " +
                                                 e.toString());
        }
    }
//当线程池不能提供服务时，拒绝处理不抛出异常
public static class DiscardPolicy implements RejectedExecutionHandler {
        public DiscardPolicy() { }
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        }
    }
//当线程池不能提供服务时,尝试着和最先进入的线程竞争，不会抛出异常
public static class DiscardOldestPolicy implements RejectedExecutionHandler {
        public DiscardOldestPolicy() { }
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                e.getQueue().poll();
                e.execute(r);
            }
        }
    }
}
```

-   线程池的最大大小如何设置
    -   CPU密集型
    -   IO密集型

	```java
// 1、CPU 密集型，几核，就是几，可以保持CPu的效率最高！
// 2、IO 密集型 > 判断你程序中十分耗IO的线程，
```

## 12、四大函数式接口

-   只含有一个抽象方法的接口，称为函数式接口

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

@FunctionalInterface
public interface Supplier<T> {
    T get();
}

public static void main(String[] args) {
        Function<String,String > function = (str)->{return str;};
        Predicate<String> predicate = (str)->{return true;};
        Consumer<String> consumer = (str)->{
            System.out.println(str);
        };
        Supplier<String> stringSupplier = ()->{return "1024";};
        System.out.println(function.apply("123"));
        System.out.println(predicate.test("1024"));
        consumer.accept("1024");
        System.out.println(stringSupplier.get());

    }
```

-   ## Lambda表达式
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622726813167-f8e355b0-0bcf-4581-a73d-fdfae1dd6610.png#align=left\&display=inline\&height=476\&margin=\[object Object]\&name=image.png\&originHeight=952\&originWidth=1983\&size=241070\&status=done\&style=none\&width=991.5> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622726858475-eb642586-4f90-4392-af44-4d1cfc282f76.png#align=left\&display=inline\&height=497\&margin=\[object Object]\&name=image.png\&originHeight=994\&originWidth=2028\&size=204388\&status=done\&style=none\&width=1014> "image.png")
-   ## 方法引用
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622727376309-cd55ba0f-8858-4bca-98cf-7c52b40681ae.png#align=left\&display=inline\&height=107\&margin=\[object Object]\&name=image.png\&originHeight=213\&originWidth=420\&size=31284\&status=done\&style=none\&width=210> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622727800868-ef4d105b-487a-46f6-9167-346fad879bdf.png#align=left\&display=inline\&height=506\&margin=\[object Object]\&name=image.png\&originHeight=1011\&originWidth=1744\&size=124067\&status=done\&style=none\&width=872> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622727919340-872b758a-d851-492e-9e32-099c977a11f0.png#align=left\&display=inline\&height=470\&margin=\[object Object]\&name=image.png\&originHeight=940\&originWidth=2067\&size=180452\&status=done\&style=none\&width=1033.5> "image.png")
-   ## 构造器引用
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622728234167-61652309-19d0-4372-8dca-0d48076291fa.png#align=left\&display=inline\&height=434\&margin=\[object Object]\&name=image.png\&originHeight=867\&originWidth=2134\&size=263401\&status=done\&style=none\&width=1067> "image.png")

## 13、Stream API

-   Stream自己不会存储元素
-   Stream不会改变源对象，相反Steam会返回一个持有结果的新的Stream对象
-   Stream是延迟操作的，当需要结果时它才执行
-
    ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622729013893-b30caf71-8af5-4fc4-8cb3-5499a67011b6.png#align=left\&display=inline\&height=555\&margin=\[object Object]\&name=image.png\&originHeight=1110\&originWidth=2086\&size=282380\&status=done\&style=none\&width=1043> "image.png")

-   创建Stream流的方式

```java
//1.Collection接口中提供了两个创建Stream流的方法
default Stream<E> stream() {} //创建串行流
default Stream<E> parallelStream() {}//创建并行流
//2.通过数组创建Stream流,使用Arrays工具中的静态方法创建流，可以接受任意类型的数组
public static <T> Stream<T> stream(T[] array) {}
public static IntStream stream(int[] array) {}
public static LongStream stream(long[] array){}
public static DoubleStream stream(double[] array){}
//3.通过Stream的静态方法of创建流，可以接受任意参数
public static<T> Stream<T> of(T... values) {}
//4.创建无限流,Stream的静态方法
public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {}
public static<T> Stream<T> generate(Supplier<T> s) {}
public static void main(String[] args) {
        Stream<Integer> iterate = Stream.iterate(0, x -> x + 2);
        iterate.limit(10).forEach(System.out::println);
        Stream<Double> generate = Stream.generate(Math::random);
        generate.limit(10).forEach(System.out::println);

    }
```

-   ## Stream的中间操作
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622730253605-c875b9eb-9cd0-4dd1-83fa-33f646084fd3.png#align=left\&display=inline\&height=344\&margin=\[object Object]\&name=image.png\&originHeight=687\&originWidth=2090\&size=221667\&status=done\&style=none\&width=1045> "image.png")
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622730273698-b571e86b-c675-4ff5-9001-f46418898953.png#align=left\&display=inline\&height=505\&margin=\[object Object]\&name=image.png\&originHeight=1010\&originWidth=2112\&size=357264\&status=done\&style=none\&width=1056> "image.png")
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622730288223-6f78cd27-77f2-4a24-9558-bf013d41e4cb.png#align=left\&display=inline\&height=357\&margin=\[object Object]\&name=image.png\&originHeight=713\&originWidth=1915\&size=112415\&status=done\&style=none\&width=957.5> "image.png")
-   ## Stream流的终止操作
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622731160354-f8d87931-40b9-4c07-aff8-3dc3b43d466e.png#align=left\&display=inline\&height=363\&margin=\[object Object]\&name=image.png\&originHeight=726\&originWidth=2005\&size=179115\&status=done\&style=none\&width=1002.5> "image.png")
    \-
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622731407270-d582198e-0b06-463a-8720-9e3dd62d5182.png#align=left\&display=inline\&height=356\&margin=\[object Object]\&name=image.png\&originHeight=711\&originWidth=2029\&size=201205\&status=done\&style=none\&width=1014.5> "image.png")
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622731462652-635a0d40-d3a5-45df-bfe3-577e8106318a.png#align=left\&display=inline\&height=425\&margin=\[object Object]\&name=image.png\&originHeight=849\&originWidth=2012\&size=191012\&status=done\&style=none\&width=1006> "image.png")
        ![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622731505796-b0c71d43-2216-4e2b-a651-a0c9e01e8eab.png#align=left\&display=inline\&height=442\&margin=\[object Object]\&name=image.png\&originHeight=883\&originWidth=2086\&size=216900\&status=done\&style=none\&width=1043> "image.png")

## 14、ForkJoin

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622731862143-d2959205-49de-42b1-9008-8c207c9085cd.png#align=left\&display=inline\&height=580\&margin=\[object Object]\&name=image.png\&originHeight=1160\&originWidth=1296\&size=244712\&status=done\&style=none\&width=648> "image.png")

-   ForkJoin的工作特点：工作窃取，，维持的是双端队列

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622731923773-379ae761-72a3-42c1-831f-6baa79f7e128.png#align=left\&display=inline\&height=588\&margin=\[object Object]\&name=image.png\&originHeight=1176\&originWidth=792\&size=104474\&status=done\&style=none\&width=396> "image.png")

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622732273291-0c3e6de2-d24a-45b3-9b52-12562426a341.png#align=left\&display=inline\&height=223\&margin=\[object Object]\&name=image.png\&originHeight=446\&originWidth=1091\&size=121697\&status=done\&style=none\&width=545.5> "image.png")

```java
package collectionsource;
import org.junit.jupiter.api.Test;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.concurrent.RecursiveTask;
public class Demo  extends RecursiveTask<Long> {
    private long start ;
    private long end;
    public Demo(long start,long end){
        this.start = start;
        this.end = end;
    }
    private long temp = 10000L;
    @Override
    protected Long compute() {
        if ((end-start) < temp){
            Long sum = 0L;
            for (Long i = start; i <= end; i++) {
                sum += i;
            }
            return sum;
        }else {
            Long middle = (start+end)/2;
            Demo demo = new Demo(start, middle);
            demo.fork();//拆分任务
            Demo demo1 = new Demo(middle + 1, end);
            demo1.fork();//拆分任务
            return demo.join()+demo1.join();
        }
    }
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        Demo demo = new Demo(0L, 100000000L);
        ForkJoinTask<Long> submit = forkJoinPool.submit(demo);
        System.out.println(submit.get());
    }
}
```

## 15、异步回调

```java
package collectionsource;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

public class Demo{
    public static void main(String[] args) throws ExecutionException, InterruptedException {
    //没有返回值的异步回调
        //        CompletableFuture<Void> runAsync = CompletableFuture.runAsync(() -> {
//            try {
//                TimeUnit.SECONDS.sleep(3);
//            } catch (InterruptedException e) {
//                e.printStackTrace();
//            }
//            System.out.println(Thread.currentThread().getName() + "--> runAsync");
//        });
//        System.out.println("===========");
//        runAsync.get();
        //有返回值的异步回调
        CompletableFuture<Integer> supplyAsync = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "--> supplyAsync");
//            int i = 10 / 0;
            return 1024;
        });
        System.out.println(supplyAsync.whenComplete((t, u) -> {
            System.out.println("t-->" + t);//正常的返回结果
            System.out.println("u-->" + u);//返回错误信息
        }).exceptionally(e -> {
            System.out.println(e.getMessage());
            return 404;//返回错误信息
        } ).get());
    }
}






```

## 16、JMM

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622734291117-8c88091d-7670-489a-94f3-44feaf802088.png#align=left\&display=inline\&height=336\&margin=\[object Object]\&name=image.png\&originHeight=671\&originWidth=1067\&size=59115\&status=done\&style=none\&width=533.5> "image.png")

-   volatile java提供的一种轻量级同步机制
    -   保证可见性
    -   不保证原子性
    -   禁止指令重排、使用内存屏障

```纯文本
内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可在分的（对于double和long类型的变量来说，load、store、read和write操作在某些平台上允许例外）
lock （锁定）：作用于主内存的变量，把一个变量标识为线程独占状态
unlock （解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
read （读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
load （载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中
use （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令
assign （赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中
store （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，
以便后续的write使用write （写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中
JMM对这八种指令的使用，制定了如下规则：
不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write
不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存
不允许一个线程将没有assign的数据从工作内存同步回主内存
一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量
实施use、store操作之前，必须经过assign和load操作
一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，
必须重新load或assign操作初始化变量的值
如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
对一个变量进行unlock操作之前，必须把此变量同步回主内存
```

## 17、单例模式

```java
//饿汉式单例模式、可能会导致资源的浪费，类一加载就创建对象，但是对象可能不会马上用到
public class Demo{
    //构造器私有化
    private Demo(){
    }
    private static final Demo INSTANCE = new Demo();
    public static Demo getInstance(){
        return INSTANCE;
    }
}

//DCL 懒汉式 单例模式
public class Demo{
    //构造器私有化
    private Demo(){
    }
    private volatile static  Demo INSTANCE ;
    public static Demo getInstance(){
        if(INSTANCE == null){
            synchronized (Demo.class){
                if (INSTANCE == null)
                    INSTANCE = new Demo();
            }
        }
        return INSTANCE;
    }
}
//静态内部类
public class Holder {
  private Holder(){
  } 
  public static Holder getInstace(){
    return InnerClass.HOLDER;
  } 
  public static class InnerClass{
    private static final Holder HOLDER = new Holder();
  }
}
//枚举
public enum EnumSingle {
  INSTANCE;
    public EnumSingle getInstance(){
      return INSTANCE;
    }
}
```

## 18、CAS

```java
 public static void main(String[] args) {
        AtomicInteger integer = new AtomicInteger(10);
        //比较并交换，如果当前值满足期望值，就更新。
        System.out.println(integer.compareAndSet(10, 20));
        System.out.println(integer.get());
    }
CAS ： 比较当前工作内存中的值和主内存中的值，如果这个值是期望的，那么则执行操作！如果不是就
一直循环！
缺点：
1、 循环会耗时
2、一次性只能保证一个共享变量的原子性
3、ABA问题
```

-   ABA问题
    -   一个线程连续更改了主内存中的共享变量多次，又变为了原来的值，但是其他线程并不知道它的修改操作，并且这个值又是其他线程的期望值。

![image.png](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1622736464812-0b73e441-c119-49a4-bd5a-3003568aa74b.png#align=left\&display=inline\&height=320\&margin=\[object Object]\&name=image.png\&originHeight=640\&originWidth=1138\&size=91037\&status=done\&style=none\&width=569> "image.png")

## 19、原子引用解决ABA问题

```java
package collectionsource;


import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicStampedReference;

public class Demo{
    public static void main(String[] args) {
        AtomicStampedReference<Integer> stampedReference = new AtomicStampedReference<>(1,1);
        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"-->版本号："+stampedReference.getStamp());
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            stampedReference.compareAndSet(1,2,stampedReference.getStamp(),stampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"-->版本号："+stampedReference.getStamp());
            stampedReference.compareAndSet(2,1,stampedReference.getStamp(),stampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"-->版本号："+stampedReference.getStamp());
        },"A").start();
        new Thread(()->{
            int stamp = stampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"-->版本号："+stamp);
            try {
                TimeUnit.SECONDS.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //版本号不同，修改失败，类似于乐观锁
            System.out.println(stampedReference.compareAndSet(1, 666, stamp, stamp + 1));
            System.out.println(Thread.currentThread().getName()+"-->版本号："+stampedReference.getStamp());
        },"B").start();
    }
}
```

## 20、各种锁

-   公平锁与非公平锁
-   可重入锁
-   自旋锁
-   死锁
-   ..........
