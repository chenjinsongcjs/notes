# 单例模式（singleTon）

-   单例模式是一种创建型模式，在整个软件系统中，保证一个类只有一个实例，并且提供一个方法可获取该实例

### 单例模式的特点

-   构造器私有，不允许外部创建对象
-   提供外部获取对象的的静态方法

### 单例模式类图

说明

1\.  **单例** （Singleton） 类声明了一个名为 `get­Instance`获取实例的静态方法来返回其所属类的一个相同实例。
单例的构造函数必须对客户端 （Client） 代码隐藏。 调用 `获取实例`方法必须是获取单例对象的唯一方式。

类图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_U8uyDRU7Gv.png)

### 饿汉式单例模式

```java
public class Singleton {
    //构造器私有
    private Singleton(){

    }
    //饿汉式没有线程安全问题
    private final static Singleton instance = new Singleton();

    public static Singleton getInstance(){
        return instance;
    }
}
```

### 懒汉式单例模式（非线程安全）

```java
public class Singleton {
    //构造器私有
    private Singleton(){

    }
    private  static Singleton instance;

    public static Singleton getInstance(){
        //当对象部位空的时候创建对象，但是线程不安全
        //当多个线程同时进入if判断时会创建多个对象，破坏单例模式
        if(instance == null){
            instance = new Singleton();
        }
        return instance;
    }
}
```

### 懒汉式单例模式（synchronize加锁）

```java
public class Singleton {
    //构造器私有
    private Singleton(){

    }
    private  static Singleton instance;
    //加方法锁影响整体性能
    public static synchronized Singleton getInstance(){
        if(instance == null){
            instance = new Singleton();
        }
        return instance;
    }
}

```

### 懒汉式单例模式（DCL双重检测锁）

```java
public class Singleton {
    //构造器私有
    private Singleton(){

    }
    //volatile 关键字避免对象出现半初始化的情况
    private volatile static Singleton instance;

    public static  Singleton getInstance(){
        if(instance == null){
            synchronized (Singleton.class){
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### 枚举单例模式

```java
public enum Singleton {
    //枚举的构造器天然是不可被外部访问的
    instance
}
```

### 内部类单例模式

```java
public class Singleton {
    private Singleton() {

    }
    //私有静态内部类实现单例模式
    private static class InnerClass {
            private final static Singleton instance = new Singleton();
    }
    public static Singleton getInstance(){
        return InnerClass.instance;
    }
}
```

### 单例模式的应用场景

-   线程池一般一个系统中只需要一个
-   数据库连接池
-   系统环境信息（System.getProperties()）
-   上下文信息
-   ....................
