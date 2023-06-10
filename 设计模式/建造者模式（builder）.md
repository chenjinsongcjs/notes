# 建造者模式（builder）

-   建造者模式是一种创建型设计模式，又叫生成器模式，能够让你分步骤的创建对象，使用相同的代码创建不同的对象

说明

1\.  **生成器** （Builder） 接口声明在所有类型生成器中通用的产品构造步骤。

2\.  **具体生成器** （Concrete Builders） 提供构造过程的不同实现。 具体生成器也可以构造不遵循通用接口的产品。

3\.  **产品** （Products） 是最终生成的对象。 由不同生成器构造的产品无需属于同一类层次结构或接口。

4\.  **主管** （Director） 类定义调用构造步骤的顺序， 这样你就可以创建和复用特定的产品配置。

5\.  **客户端** （Client） 必须将某个生成器对象与主管类关联。 一般情况下， 你只需通过主管类构造函数的参数进行一次性关联即可。 此后主管类就能使用生成器对象完成后续所有的构造任务。 但在客户端将生成器对象传递给主管类制造方法时还有另一种方式。 在这种情况下， 你在使用主管类生产产品时每次都可以使用不同的生成器。

类图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_OfyUkfEKEn.png)

### 生成器

```java
public interface Builder {
    void setCarType(String carType);
    void setSeats(int seats);
    void setEngine(String engine);
    void setTransmission(String transmission);
    void setTripComputer(String tripComputer);
    void setGpsNavigator(String gpsNavigator);
}
```

### 具体生成器

```java
public class CarBuilder implements Builder {
    private String type;
    private int seats;
    private String engine;
    private String transmission;
    private String tripComputer;
    private String gpsNavigator;
    @Override
    public void setCarType(String carType) {
        this.type = carType;
    }

    @Override
    public void setSeats(int seats) {
        this.seats = seats;
    }

    @Override
    public void setEngine(String engine) {
        this.engine =engine;
    }

    @Override
    public void setTransmission(String transmission) {
        this.transmission = transmission;
    }

    @Override
    public void setTripComputer(String tripComputer) {
        this.transmission = tripComputer;
    }

    @Override
    public void setGpsNavigator(String gpsNavigator) {
        this.gpsNavigator = gpsNavigator;
    }

    public Car getResult(){
        return new Car(type,seats,engine,transmission,tripComputer,gpsNavigator,0);
    }
}


public class CarManualBuilder implements Builder {
    private String type;
    private int seats;
    private String engine;
    private String transmission;
    private String tripComputer;
    private String gpsNavigator;

    @Override
    public void setCarType(String carType) {
        this.type = carType;
    }

    @Override
    public void setSeats(int seats) {
        this.seats = seats;
    }

    @Override
    public void setEngine(String engine) {
        this.engine = engine;
    }

    @Override
    public void setTransmission(String transmission) {
        this.transmission = transmission;
    }

    @Override
    public void setTripComputer(String tripComputer) {
        this.tripComputer = tripComputer;
    }

    @Override
    public void setGpsNavigator(String gpsNavigator) {
        this.gpsNavigator = gpsNavigator;
    }

    public Manual getResult(){
        return new Manual(type,seats,engine,transmission,tripComputer,gpsNavigator);
    }
}


```

### 产品

```java
@Data
@ToString
@AllArgsConstructor
public class Car{
    private final String carType;
    private final int seats;
    private final String engine;
    private final String transmission;
    private final String tripComputer;
    private final String gpsNavigator;
    private double fuel = 0;
}

@Data
@ToString
@AllArgsConstructor
public class Manual {
    private final String carType;
    private final int seats;
    private final String engine;
    private final String transmission;
    private final String tripComputer;
    private final String gpsNavigator;
}


```

### 主管类，控制常用产品的生成

```java
public class Director {
    public void constructSportsCar(Builder builder) {
        builder.setCarType("SPORTS_CAR");
        builder.setSeats(2);
        builder.setEngine("v3.0");
        builder.setTransmission("SEMI_AUTOMATIC");
        builder.setTripComputer("cpu sportsCar");
        builder.setGpsNavigator("高德地图");
    }

    public void constructCityCar(Builder builder) {
        builder.setCarType("CITY_CAR");
        builder.setSeats(2);
        builder.setEngine("v1.2");
        builder.setTransmission("AUTOMATIC");
        builder.setTripComputer("cpu cityCar");
        builder.setGpsNavigator("百度地图");
    }

    public void constructSUV(Builder builder) {
        builder.setCarType("SUV");
        builder.setSeats(4);
        builder.setEngine("v2.5");
        builder.setTransmission("MANUAL");
        builder.setGpsNavigator("腾讯地图");
    }
}

```

### 测试

```java
public class MainTest {
    public static void main(String[] args) {
        Director director = new Director();
        CarBuilder carBuilder = new CarBuilder();
        director.constructCityCar(carBuilder);
        System.out.println(carBuilder.getResult());

        CarManualBuilder carManualBuilder = new CarManualBuilder();
        director.constructSportsCar(carManualBuilder);
        System.out.println(carManualBuilder.getResult());
    }
}

```
