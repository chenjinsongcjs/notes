# 代理模式（proxy）

-   **代理模式**是一种结构型设计模式， 让你能够提供对象的替代品或其占位符。 代理控制着对于原对象的访问， 并允许在将请求提交给对象前后进行一些处理。

描述

1\.  **服务接口** （Service Interface） 声明了服务接口。 代理必须遵循该接口才能伪装成服务对象。

2\.  **服务** （Service） 类提供了一些实用的业务逻辑。

3\.  **代理** （Proxy） 类包含一个指向服务对象的引用成员变量。 代理完成其任务 （例如延迟初始化、 记录日志、 访问控制和缓存等） 后会将请求传递给服务对象。 通常情况下， 代理会对其服务对象的整个生命周期进行管理。

4\.  **客户端** （Client） 能通过同一接口与服务或代理进行交互， 所以你可在一切需要服务对象的代码中使用代理。

类图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_-oMFW8MkC1.png)

### 静态代理同装饰者模式

### JDK动态代理

-   必须实现相同的接口，才能进行代理

```java
public interface Person {
    void say();
}
public class Student implements Person{
    @Override
    public void say() {
        System.out.println("我是学生。。。");
    }
}

public class MainTest {
    public static void main(String[] args) {
        Student student = new Student();
        //创建当前对象的动态代理
        Person instance = (Person) Proxy.newProxyInstance(student.getClass().getClassLoader(), student.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("说话前。。。");
                Object invoke = method.invoke(student, args);
                System.out.println("说话后。。。");
                return invoke;
            }
        });
        instance.say();
    }
}


```

```java
public class JdkTiktokProxy<T> implements InvocationHandler {


    private T target;
    //接受被代理对象
    JdkTiktokProxy(T target){
        this.target = target;
    }
    /**
     * 获取被代理对象的  代理对象
     * @param t
     * @param <T>
     * @return
     */
    public static<T> T getProxy(T t) {

        /**
         * ClassLoader loader, 当前被代理对象的类加载器
         * Class<?>[] interfaces, 当前被代理对象所实现的所有接口
         * InvocationHandler h,
         *  当前被代理对象执行目标方法的时候我们使用h可以定义拦截增强方法
         */
        Object o = Proxy.newProxyInstance(
                t.getClass().getClassLoader(),
                t.getClass().getInterfaces(), //必须接口
                new JdkTiktokProxy(t));
        return (T)o;
    }


    /**
     * 定义目标方法的拦截逻辑；每个方法都会进来的
     *
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy,
                         Method method,
                         Object[] args) throws Throwable {

        //反射执行
        System.out.println("真正执行被代理对象的方法");
        Object invoke = method.invoke(target, args);
        System.out.println("返回值：一堆美女");
        return invoke;
    }
}

```

### cglib动态代理

-   被代理类不能是final的，必须能创建子类

```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>
```

```java
public class CglibProxy {

    public static <T> T createProxy(T t){
        //创建增强器
        Enhancer enhancer = new Enhancer();
        //要为哪一个类增强就为那个类创建子类
        enhancer.setSuperclass(t.getClass());
        //设置回调
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object o, //被代理对象
                                    Method method, //被代理对象的方法，主要提供一些方法的元信息
                                    Object[] objects, //参数
                                    MethodProxy methodProxy ) throws Throwable {
                System.out.println("方法执行之前。。。");
                //相当于执行当前代理方法会一直循环，
//                Object invoke = methodProxy.invoke(o, objects);
                Object invoke = methodProxy.invokeSuper(o, objects);
                System.out.println("方法执行之后。。。");
                return invoke;
            }
        });
        //创建代理对象
        Object o = enhancer.create();
        return (T) o;
    }
}


public class MainTest {
    public static void main(String[] args) {
        Student student = new Student();
        Student proxy = CglibProxy.createProxy(student);
        proxy.say();

    }
}

```

### 使用场景

-   MyBatis的mapper到底是什么？怎么生成的？
-   Seata的DataSourceProxy是什么？
-   DruidDataSource存在的Proxy模式
