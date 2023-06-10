# 装饰模式（decorator）

-   **装饰模式**是一种结构型设计模式， 允许你通过将对象放入包含行为的特殊封装对象中来为原对象绑定新的行为。简单的说就是对原来对象的功能进行增强。

描述

1\.  **部件** （Component） 声明封装器和被封装对象的公用接口。

2\.  **具体部件** （Concrete Component） 类是被封装对象所属的类。 它定义了基础行为， 但装饰类可以改变这些行为。

3\.  **基础装饰** （Base Decorator） 类拥有一个指向被封装对象的引用成员变量。 该变量的类型应当被声明为通用部件接口， 这样它就可以引用具体的部件和装饰。 装饰基类会将所有操作委派给被封装的对象。

4\.  **具体装饰类** （Concrete Decorators） 定义了可动态添加到部件的额外行为。 具体装饰类会重写装饰基类的方法， 并在调用父类方法之前或之后进行额外的行为。

5\.  **客户端** （Client） 可以使用多层装饰来封装部件， 只要它能使用通用接口与所有对象互动即可。

类图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_SecugFdR8r.png)

### 代码示例

```java
//磁盘读写接口
public interface DataSource {
    void writeData(String data);
    String readData();
}
//简单读写
public class FileDataSource implements DataSource {
    private String name;

    public FileDataSource(String name) {
        this.name = name;
    }

    @Override
    public void writeData(String data) {
        System.out.Println("简单的写数据");
    }

    @Override
    public String readData() {
        return "简单的读数据";
    }
}

//抽象装饰者含有，抽象组件的引用
public class DataSourceDecorator implements DataSource {
    private DataSource wrappee;

    DataSourceDecorator(DataSource source) {
        this.wrappee = source;
    }

    @Override
    public void writeData(String data) {
        wrappee.writeData(data);
    }

    @Override
    public String readData() {
        return wrappee.readData();
    }
}
//将功能的改变，具体装饰者
public class EncryptionDecorator extends DataSourceDecorator {

    public EncryptionDecorator(DataSource source) {
        super(source);
    }

    @Override
    public void writeData(String data) {
        super.writeData(encode(data));
    }

    @Override
    public String readData() {
        return decode(super.readData());
    }

    private String encode(String data) {
        return new String("encode"+data);
    }

    private String decode(String data) {
        return new String("decode"+data);
    }
}

```

### 使用场景

1.  如果你希望在无需修改代码的情况下即可使用对象， 且希望在运行时为对象新增额外的行为， 可以使用装饰模式。
2.  如果用继承来扩展对象行为的方案难以实现或者根本不可行， 你可以使用该模式。

-   SpringSession中如何进行session与redis关联？HttpRequestWrapper
-   MyBatisPlus提取了QueryWrapper，这是什么？
-   Spring中的BeanWrapper是做什么？
-   Spring Webflux中的 WebHandlerDecorator？
