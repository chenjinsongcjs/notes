# 工厂方法模式（factoryMethod）

-   工厂方法模式是一种创建型设计模式，父类提供一个创建对象的方法，允许子类决定具体实例化的对象的类型
-   工厂方法主要是生产同一种类型的产品，抽象工厂可以生产不同类别的产品。

说明

1.  &#x20; **产品** （Product） 将会对接口进行声明。 对于所有由创建者及其子类构建的对象， 这些接口都是通用的。
2.  &#x20;**具体产品** （Concrete Products） 是产品接口的不同实现。
3.  **具体创建者** （Concrete Creators） 将会重写基础工厂方法， 使其返回不同类型的产品。

    注意， 并不一定每次调用工厂方法都会**创建**新的实例。 工厂方法也可以返回缓存、 对象池或其他来源的已有对象。

类图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_sVBhZrszfl.png)

1.  **创建者** （Creator） 类声明返回产品对象的工厂方法。 该方法的返回对象类型必须与产品接口相匹配。
    你可以将工厂方法声明为抽象方法， 强制要求每个子类以不同方式实现该方法。 或者， 你也可以在基础工厂方法中返回默认产品类型。
    注意， 尽管它的名字是创建者， 但它最主要的职责并**不是**创建产品。 一般来说， 创建者类包含一些与产品相关的核心业务逻辑。 工厂方法将这些逻辑处理从具体产品类中分离出来。 打个比方， 大型软件开发公司拥有程序员培训部门。 但是， 这些公司的主要工作还是编写代码， 而非生产程序员。

### 产品定义

```java
//抽象的汽车
public interface Car {
    void description();
}

//具体的汽车
public class RacingCar implements Car{
    @Override
    public void description() {
        System.out.println("this is a racing car，it is very fast");
    }
}


//具体的汽车
public class CommonCar implements Car{
    @Override
    public void description() {
        System.out.println("this is a common car, it run slowly");
    }
}


```

### 工厂定义

```java
//抽象工厂父类
public abstract class CarFactory {
    protected abstract Car ProductCar();
}

//具体的工厂决定生产的对象的类型
public class CommonCarFactory extends CarFactory {
    @Override
    protected Car ProductCar() {
        return new CommonCar();
    }
}


//具体的工厂决定生产的对象的类型
public class RacingCarFactory extends CarFactory{
    @Override
    protected Car ProductCar() {
        return new RacingCar();
    }
}

```

### 测试

```java
public class MainTest {
    public static void main(String[] args) {
        CommonCarFactory commonCarFactory = new CommonCarFactory();
        Car commonCar = commonCarFactory.ProductCar();
        commonCar.description();

        RacingCarFactory racingCarFactory = new RacingCarFactory();
        Car racingCar = racingCarFactory.ProductCar();
        racingCar.description();
    }
}

```
