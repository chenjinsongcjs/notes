# CompletableFuture

## Future和Callable接口

-   Future接口定义了操作异步任务执行一些方法，如获取异步任务的执行结果、取消任务的执行、判断任务是否被取消、判断任务执行是否完毕等。
-   Callable中定义了需要有返回值的任务需要实现的方法

## FutureTask

第1列

Thread只能接受Runable接口的实现类，Callable和`Runable`是不同的接口。要想Thread类能够运行Callable的实现类，必须要找到一个中间桥梁，FutureTask就是这个桥梁。

将Callable的实现类放入FutrueTask中，Thread就能开启一个线程运行call方法，并且可以通过FutrueTask的get方法获取，运行的结果。

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_tDRKyw2dvI.png)

-   FutrueTask的不足
    -   get方法阻塞：使用get方法获取运行结果的时候，不管运行是否结束，主线程都会被阻塞。效率比较低，建议放在程序的末尾处。
    -   isDone轮询：这种方式可以减缓get的阻塞行为但是，依然没有本质上解决问题
    ```java
    package com.xyz;

    import java.util.concurrent.ExecutionException;
    import java.util.concurrent.FutureTask;
    import java.util.concurrent.TimeUnit;
    import java.util.concurrent.TimeoutException;

    public class FutureTaskDemo {
        public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
            //创建FutureTask对象包装Callable
            FutureTask<Integer> task = new FutureTask<>(() -> {
                //模拟业务执行过程，测试get方法的阻塞过程
                TimeUnit.SECONDS.sleep(2);
                return 1024;
            });
            //创建一个线程执行Callable
            new Thread(task,"t1").start();
            //get阻塞获取线程执行的结果
    //        Integer integer = task.get();
            //超时获取线程执行结果，如果指定时间内没有执行完，就会抛出超时异常
    //        Integer integer = task.get(1, TimeUnit.SECONDS);
    //        System.out.println("获取执行结果："+integer);
            while (true){
                //isDone轮询是否线程是否执行完毕，如果执行完毕就获取结果
                if (task.isDone()){
                    System.out.println("获取执行结果："+task.get());
                    break;
                }
                else
                    System.out.println("CPU不断轮询");
            }
        }

    }


    ```

## CompletableFuture和CompletionStage的介绍

第1列

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T>
```

CompletableFuture实现了Futrue接口和CompletionStage接口，很好的支持异步任务和阶段性任务。

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_iOFuntAfQC.png)

第1列

和Linux中的管道符操作相似。

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_wsWbkurAOg.png)

第1列

CompletableFutrue的介绍

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_OpbZYNEk0I.png)

## CompletableFutrue的四个核心静态方法

```java
//没有返回值,使用自带线程池 ForkJoinPool.commonPool()
public static CompletableFuture<Void> runAsync(Runnable runnable)
//没有返回值，使用自定义线程池
public static CompletableFuture<Void> runAsync(Runnable runnable,Executor executor)
//有返回值，使用自带线程池 ForkJoinPool.commonPool()
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
//有返回值，使用自定义线程池
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,Executor executor)

```

```java
 public static void main(String[] args) throws InterruptedException, ExecutionException {

        ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 10, 10, TimeUnit.SECONDS, new LinkedBlockingDeque<Runnable>());
        //无返回值，使用自带线程池
        CompletableFuture<Void> hello =
                CompletableFuture.runAsync(() -> System.out.println(Thread.currentThread().getName()+" hello"));
        CompletableFuture<Integer> supplyAsync = CompletableFuture.supplyAsync(() ->{
            System.out.println(Thread.currentThread().getName());
            return 1024;
        }, executor);
        hello.get();
        System.out.println(supplyAsync.get());

}

```

> 自定义的线程池一定要记得关闭

-   方法的使案例

```java
public class JUC {
    public static void main(String[] args) throws InterruptedException, ExecutionException {

        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 2;
        }).thenApply(a -> a + 2)
                .whenComplete((u, v) -> {
                    System.out.println("结果：" + u);
                    System.out.println("异常：" + v);
                }).exceptionally(e -> {
                    e.printStackTrace();
                    return -1;
                });
        System.out.println(future.get());
    }
}

```

-   CompletableFutrue的优点
    -   异步任务结束时，会自动回调某个对象的方法
    -   异步任务异常时，会自动回调某个对象的方法
    -   主线程设置好回调后，不再关心异步任务的执行，异步任务可以按照某种顺序执行。

## 案例演示：（CompletableFuture应用）

-   函数式编程参见笔记：[https://www.wolai.com/pkmRPU2J42KxfVv45emUMV#suNmREHFu6xBXvWhE4Ejgi](https://www.wolai.com/pkmRPU2J42KxfVv45emUMV#suNmREHFu6xBXvWhE4Ejgi "https://www.wolai.com/pkmRPU2J42KxfVv45emUMV#suNmREHFu6xBXvWhE4Ejgi")
-   CompletableFuture中`get`方法和`join`的区别？
    -   相同点：get和join都会阻塞直到异步任务完成并获取结果
    -   不同点：get会捕获处理过的异常可以自己决定是抛出还是自己处理；join只会抛出未经检查的异常
    -   join不需要异常处理
-   使用场景

> 经常出现在等待某条 SQL 执行完成后，再继续执行下一条 SQL ，而这两条 SQL 本身是并无关系的，可以同时进行执行的。我们希望能够两条 SQL 同时进行处理，而不是等待其中的某一条 SQL 完成后，再继续下一条。同理，对于分布式微服务的调用，按照实际业务，如果是无关联step by step的业务，可以尝试是否可以多箭齐发，同时调用。

### 案例：（模拟获取不同商城的同一个商品的价格进行比较）

```java
package com.xyz;

import java.util.Arrays;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.TimeUnit;
import java.util.stream.Collectors;

public class CompareGoodsPrice {
    static List<NetMall> list = Arrays.asList(new NetMall("Taobao")
    ,new NetMall("pdd"),new NetMall("JD"));
    //模拟同步获取不同商城的同一个商品的价格
    public static List<String> findProductPriceSync(List<NetMall> malls,String productName){
        return malls.stream().map(mall -> String.format("%s price is %.2f", mall.getMallName(), mall.getPriceByName(productName)))
                .collect(Collectors.toList());
    }
    //模拟异步获取不同商城的同一个商品的价格
    public static List<String> findProductPriceASync(List<NetMall> malls,String productName){
        return malls.stream().map(
                mall-> CompletableFuture.supplyAsync(()->String.format("%s price is %.2f", mall.getMallName(), mall.getPriceByName(productName))
                )).collect(Collectors.toList()).stream().map(CompletableFuture::join).collect(Collectors.toList());
    }
    //测试
    public static void main(String[] args) {
        long startTime = System.currentTimeMillis();
        List<String> list1 = findProductPriceSync(list, "thinking in java");
        for (String element : list1) {
            System.out.println(element);
        }
        long endTime = System.currentTimeMillis();
        System.out.println("----costTime: "+(endTime - startTime) +" 毫秒");

        long startTime2 = System.currentTimeMillis();
        List<String> list2 = findProductPriceASync(list, "thinking in java");
        for (String element : list2) {
            System.out.println(element);
        }
        long endTime2 = System.currentTimeMillis();
        System.out.println("----costTime: "+(endTime2 - startTime2) +" 毫秒");

    }
}

class NetMall{
    private String mallName;

    public NetMall(String mallName){
        this.mallName = mallName;
    }
    public String getMallName(){
        return this.mallName;
    }
    //模拟获取商品的价格
    public double getPriceByName(String name){
        //这个过程比较耗时
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //多线程随机值模拟商品的价格
        return ThreadLocalRandom.current().nextDouble() + name.charAt(0);
    }
}

```

## CompletableFuture的常用方法

### 获取结果和触发计算

-   获取结果

```java
  public T get()//阻塞获取结果，有异常可以返回具体的异常（捕获的）
  public T get(long timeout, TimeUnit unit)//阻塞获取结果但是指定超时时间，超时会抛出超时异常
  public T getNow(T valueIfAbsent)//不会组设立即获取结构，如果不能立即获取结果，使用valueIfAbsent作为结果
  public T join()//阻塞获取结果，会抛出未经检查的异常
```

-   主动触发结算

```java
public boolean complete(T value)
//主动触发计算，如果是这个方法导致CompletableFuture状态改变为完成，则返回true，否则为false，
//如果异步任务没有完成，get等方法的返回值设置为value

//在Complete调用之前，任务完成，返回false,get获取的是计算的结果
//在Complete调用之后完成任务，返回true，get获取complete传入的值


public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 1024;
        });
//        //false
//        //1024
//        TimeUnit.SECONDS.sleep(3);
        System.out.println(future.complete(10));
        System.out.println(future.get());
    }

```

### 对计算结果进行处理

-   thenApply

```java
//计算结果存在依赖性，当前计算需要上一个步骤的结果，串行化计算
//串行化过程中如果发生异常，就会终止当前的计算，进行异常处理。
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn) 

public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 10)
                .thenApply(num -> num + 10)//串行化计算，如果出现异常就会终止
                .whenComplete((r, e) -> {//当计算完毕并且没有异常时获取结果
                    if (e == null)
                        System.out.println(r);
                }).exceptionally(t -> {//对异常进行处理
                    t.printStackTrace();
                    return -1;
                });
        System.out.println(future.join());

    }

```

-   handle

```java
//也是串行化计算，只是相对于上面的方法不同的是，如果有异常，不会终止计算，会携带异常结果往下计算
public <U> CompletableFuture<U> handle(BiFunction<? super T, Throwable, ? extends U> fn)


 public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            //暂停几秒钟线程
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("111");
            return 1024;
        }).handle((f, e) -> {
            int age = 10 / 0;
            System.out.println("222");
            return f + 1;
        }).handle((f, e) -> {
            System.out.println("333");
            return f + 1;
        }).whenCompleteAsync((v, e) -> {
            System.out.println("*****v: " + v);
        }).exceptionally(e -> {
            e.printStackTrace();
            return null;
        });
        future.get();
        System.out.println("-----主线程结束，END");


    }

```

第1列

> whenComplete和whenCompleteAsync的区别

-   whenComplete：执行当前任务的线程继续执行whenComplete任务
-   whenCompleteAsync：将whenCompleteAsync的任务继续交给线程池处理

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_uQe1Ll1bsd.png)

### 对计算结果进行消费

-   thenAceept  (接收任务的处理结果，进行消费，没有返回值  )

```java
 public CompletableFuture<Void> thenAccept(Consumer<? super T> action) 
 
 public static void main(String[] args) {
        CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> 10)
                .thenApply(n -> n + 10)
                .thenApply(n -> n + 10)
                .thenAccept(result -> System.out.println(result));
        future.join();
    }
```

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_oBvdQCL13W.png)

### 对计算速度进行选用

-   applyToEither（对计算速度进行选用，谁执行快就使用谁）

```java
  public <U> CompletableFuture<U> applyToEither(CompletionStage<? extends T> other, Function<? super T, U> fn) 
 
 public static void main(String[] args) {
        CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 10;
        });
        CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 20;
        });
        CompletableFuture<Void> future = future1.applyToEither(future2, f -> f + 10).thenAccept(r -> System.out.println(r));
        future.join();
    }
```

### 对计算结果进行合并

-   thenCombine
-   两个CompletionStage的任务完成之后一起提交给thenCombine进行处理  （先完成的需要等待）

```java
 public <U,V> CompletableFuture<V> thenCombine(
        CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn)
 
 public static void main(String[] args) {
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 10)
                .thenCombine(CompletableFuture.supplyAsync(() -> 20), (x, y) -> x + y)
                .thenCombine(CompletableFuture.supplyAsync(() -> 30), (a, b) -> a + b);
        System.out.println(future.join());
    }
```

[Java锁系列](https://www.wolai.com/qJehTuRNexuZMQv8km5cbL "Java锁系列")
