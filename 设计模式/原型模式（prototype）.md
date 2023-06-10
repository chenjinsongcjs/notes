# 原型模式（prototype）

-   原型模式属于创建者类型，可以使用原型模式复制已有的对象，而无需使代码依赖他们所属的类
-   原型模式已经和java浑然一体。

### 原型模式类图

说明

1\.  **原型** （Prototype） 接口将对克隆方法进行声明。 在绝大多数情况下， 其中只会有一个名为 `clone`克隆的方法。

2\.  **具体原型** （Concrete Prototype） 类将实现克隆方法。 除了将原始对象的数据复制到克隆体中之外， 该方法有时还需处理克隆过程中的极端情况， 例如克隆关联对象和梳理递归依赖等等。

3\.  **客户端** （Client） 可以复制实现了原型接口的任何对象。

类图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_GXyHXjAXDo.png)

### 代码实现

```java
@Data
@AllArgsConstructor
@ToString
public class User {
    private String name;
    private int age;
    //实现浅拷贝
    @Override
    protected Object clone() throws CloneNotSupportedException {
        User user = new User(this.name,this.age);
        return user;
    }
}

public static void main(String[] args) throws CloneNotSupportedException {
      User user = new User("张三",18);
      Object userCopy = user.clone();
      System.out.println(user);
      System.out.println(userCopy);
      System.out.println(user == userCopy);
}
//Output:
//User(name=张三, age=18)
//User(name=张三, age=18)
//false

```

### 深拷贝和浅拷贝

-   浅拷贝：当拷贝对象的时候，只拷贝基本数据类型或者不可变类型（string）给新对象，而引用类型的对象没有进行拷贝，而是将引用对象的地址拷贝给新对象。
-   深拷贝：不管是基本数据类型还是引用数据类型都完全复制一份给新对象

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_EnWaaPteIx.png)

-   实现深拷贝的方法
    1.  类中的每个应用属性都实现clone方法，然后再clone的时候进行递归拷贝
    2.  通过序列化和反序列化实现深拷贝

```java
@Data
@AllArgsConstructor
@ToString
public class User implements Cloneable, Serializable {
    private String name;
    private int age;
    
    //通过序列化和反序列化实现clone
    @SneakyThrows
    @Override
    protected Object clone() throws CloneNotSupportedException {
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        ObjectOutputStream write = new ObjectOutputStream(outputStream);
        write.writeObject(this);

        ByteArrayInputStream inputStream = new ByteArrayInputStream(outputStream.toByteArray());
        ObjectInputStream read = new ObjectInputStream(inputStream);
        Object object = read.readObject();
        return object;
    }
}

```

### 原型模式的应用场景

-   资源优化
-   性能和安全要求
-   一个对象多个修改这的场景
