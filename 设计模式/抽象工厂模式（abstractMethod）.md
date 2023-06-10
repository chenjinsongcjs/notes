# 抽象工厂模式（abstractMethod）

-   抽象工厂是一种创建型的设计模式，它能创建一系列相关的对象， 而无需指定其具体类。是工厂方法的进一步抽象
-   可以为不同的环境适配相同的东西，生成不同风格的同类产品

说明

1\.  **抽象产品** （Abstract Product） 为构成系列产品的一组不同但相关的产品声明接口。

2\.  **具体产品**（Concrete Product） 是抽象产品的多种不同类型实现。 所有变体 （维多利亚/现代） 都必须实现相应的抽象产品 （椅子/沙发）。

3\.  **抽象工厂** （Abstract Factory） 接口声明了一组创建各种抽象产品的方法。

4\.  **具体工厂**（Concrete Factory） 实现抽象工厂的构建方法。 每个具体工厂都对应特定产品变体， 且仅创建此种产品变体。

类图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_fDipt7LyV4.png)

5\.  尽管具体工厂会对具体产品进行初始化， 其构建方法签名必须返回相应的\_抽象\_产品。 这样， 使用工厂类的客户端代码就不会与工厂创建的特定产品变体耦合。 **客户端** （Client） 只需通过抽象接口调用工厂和产品对象， 就能与任何具体工厂/产品变体交互。

### 产品定义

```java
//抽象的产品1
public interface CheckBox {
    void print();
}

//抽象的产品2
public interface Button {
    void print();
}

//不同环境的事现类
public class LinuxButton implements Button{
    @Override
    public void print() {
        System.out.println("this is a button of linux");
    }
}
//不同环境的具体实现类
public class WindowsButton implements Button{
    @Override
    public void print() {
        System.out.println("this is a button of windows");
    }
}


public class LinuxCheckBox implements CheckBox{
    @Override
    public void print() {
        System.out.println("this is a checkbox of linux");
    }
}

public class WindowsCheckBox implements CheckBox
{
    @Override
    public void print() {
        System.out.println("this is a checkbox of windows");
    }
}

```

### 工厂定义

```java
//抽象的工厂，可以生产一系列相关的东西，具体的类型由实现类决定
public interface AbstractFactory {
    Button productButton();
    CheckBox productCheckBox();
}

//具体工厂实现，抽象工厂主要是将不同类型的同一类产品进行统一生成
public class LinuxFactory implements AbstractFactory{
    @Override
    public Button productButton() {
        return new LinuxButton();
    }

    @Override
    public CheckBox productCheckBox() {
        return new LinuxCheckBox();
    }
}

public class WindowsFactory implements AbstractFactory{
    @Override
    public Button productButton() {
        return new WindowsButton();
    }

    @Override
    public CheckBox productCheckBox() {
        return new WindowsCheckBox();
    }
}

```

### 测试

```java
public class MainTest {
    public static void main(String[] args) {
        WindowsFactory windowsFactory = new WindowsFactory();
        Button windowsButton = windowsFactory.productButton();
        CheckBox windowsCheckBox = windowsFactory.productCheckBox();
        windowsButton.print();
        windowsCheckBox.print();
        System.out.println("--------------");
        LinuxFactory linuxFactory = new LinuxFactory();
        Button linuxButton = linuxFactory.productButton();
        CheckBox linuxCheckBox = linuxFactory.productCheckBox();
        linuxButton.print();
        linuxCheckBox.print();
    }
}
```
