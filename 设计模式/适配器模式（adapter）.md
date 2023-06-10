# 适配器模式（adapter）

-   适配器模式是一种结构型设计模式，它能使接口不兼容的两个对象互相合作
-   它分为类适配器（通过继承实现）和对象适配器（通过组合实现）
-   适配器主要提供一个中间层类去适配那些第三方类、遗留类或者提供怪异接口的类

### 对象适配器

描述

1\.  **客户端** （Client） 是包含当前程序业务逻辑的类。

2\.  **客户端接口** （Client Interface） 描述了其他类与客户端代码合作时必须遵循的协议。

3\.  **服务** （Service） 中有一些功能类 （通常来自第三方或遗留系统）。 客户端与其接口不兼容， 因此无法直接调用其功能。

4\.  **适配器** （Adapter） 是一个可以同时与客户端和服务交互的类： 它在实现客户端接口的同时封装了服务对象。 适配器接受客户端通过适配器接口发起的调用， 并将其转换为适用于被封装服务对象的调用。

类图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_V1gdRy2qRb.png)

5\. 客户端代码只需通过接口与适配器交互即可， 无需与具体的适配器类耦合。 因此， 你可以向程序中添加新类型的适配器而无需修改已有代码。 这在服务类的接口被更改或替换时很有用： 你无需修改客户端代码就可以创建新的适配器类。

### 实例代码（方钉与圆孔适配）

```java

//圆孔类，要让方钉能够放的进去
@AllArgsConstructor
public class RoundHole {
    private double radius;
    //判断圆钉是否能够放入圆孔
    public boolean fits(RoundPeg roundPeg){
        return this.radius >= roundPeg.getRadius();
    }

}


@Getter
@AllArgsConstructor
@NoArgsConstructor
public class RoundPeg {
    private double radius;
}


@Getter
@AllArgsConstructor
public class SquarePeg {
    private double width;
}


public class Adapter extends RoundPeg{
    private SquarePeg squarePeg;
    public Adapter(SquarePeg squarePeg) {
        this.squarePeg = squarePeg;
    }
    //适配核心，转换
    @Override
    public double getRadius() {
        return (Math.sqrt(Math.pow((squarePeg.getWidth() / 2), 2) * 2));
    }
}

public class MainTest {
    public static void main(String[] args) {
        RoundHole roundHole = new RoundHole(5);
        RoundPeg roundPeg = new RoundPeg(5);
        System.out.println("圆钉配圆孔： "+roundHole.fits(roundPeg));

        SquarePeg squarePeg = new SquarePeg(2);
        Adapter adapter = new Adapter(squarePeg);
        System.out.println("方钉适配圆孔："+roundHole.fits(adapter));
    }
}

```

### 使用场景

-   Tomcat如何将Request流转为标准Request
-   Spring AOP中的AdvisorAdapter是什么
-   Spring MVC中经典的HandlerAdapter是什么
-   SpringBoot 中 WebMvcConfigurerAdapter为什么存在又取消
