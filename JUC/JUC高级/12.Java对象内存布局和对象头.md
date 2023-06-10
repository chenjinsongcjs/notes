# Java对象内存布局和对象头

-   Object object = new Object()谈谈你对这句话的理解?一般而言JDK8按照默认情况下，new一个对象占多少内存空间
    -   新建出来的对象是放在堆空间的伊甸园区
    -   object变量是放在栈空间的局部变量表中
    -   类型信息放在方法区中，每个对象的对象头中有指向方法区类型的指针
    -   一般new一个没有成员变量的普通对象占16字节空间（markword+类型指针（可能被压缩））

## 对象在内存中的布局

第1列

-   对象头
-   实例数据
-   对齐填充（保证整个对象是8字节的整数倍）

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_MGsGSE7tDm.png)

### 对象头

-   对象标记（MarkWord）

第1列

在64位系统中，Mark Word占了8个字节，类型指针占了8个字节，一共是16个字节

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_HfGPsFqInW.png)

-   MarkWork默认存储对象的HashCode、分代年龄和锁标志位等信息。这些信息都是与对象自身定义无关的数据，**所以MarkWord被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据。** 它会根据对象的状态复用自己的存储空间，也就是说在运行期间`MarkWord`里存储的数据会随着锁标志位的变化而变化。

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_7XG9dwv4Te.png)

-   类型指针（类元信息）

第1列

对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_Of0ZpSDqlK.png)

-   如果是数组，对象头中还包含数组的长度

### 实例数据

-   存放类的属性(Field)数据信息，包括父类的属性信息，私有的也包含，窄类型会插到宽类型中

### 对齐填充

-   虚拟机要求对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了字节对齐这部分内存按8字节补充对齐。
-   openJDK官方术语

<http://openjdk.java.net/groups/hotspot/docs/HotSpotGlossary.html>

### 64位虚拟机的对象头

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_pyQf8oXRx3.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_3tzvvNWopR.png)

### java对象展示，深入了解java对象信息

<http://openjdk.java.net/projects/code-tools/jol/>

```java
   public static void main(String[] args) {
        //获取当前虚拟机的详细信息
        System.out.println(VM.current().details());
        ////所有的对象分配的字节都是8的整数倍。
        System.out.println(VM.current().objectAlignment());
    }
    //JOL的依赖
    <dependency>
        <groupId>org.openjdk.jol</groupId>
        <artifactId>jol-core</artifactId>
        <version>0.16</version>
    </dependency>
```

-   对Object object = new Object()的解析

```java
public static void main(String[] args) {
        Object object = new Object();
        System.out.println(ClassLayout.parseInstance(object).toPrintable());
    }
 //输出信息
java.lang.Object object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000000000000005 (biasable; age: 0)
  8   4        (object header: class)    0x00001000
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

//分析
markword  占8个字节
类型指针   占4个字节（压缩指针）-XX:+UseCompressedClassPointers  默认开启压缩类型指针
对齐填充4个字节


//字段解释
OFFSET  偏移量，也就是到这个字段位置所占用的byte数
SIZE  后面类型的字节大小
TYPE  是Class中定义的类型
DESCRIPTION  DESCRIPTION是类型的描述
VALUE  VALUE是TYPE在内存中的值

```
