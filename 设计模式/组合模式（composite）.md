# 组合模式（composite）

-   组合模式是一种结构型设计模式，你可以使用它将对象组合成树状结构，并且能像使用独立对象一样使用他们。
-   特点：组合类中有一组相似的对象

第1列

1\.  **组件** （Component） 接口描述了树中简单项目和复杂项目所共有的操作。

2\.  **叶节点** （Leaf） 是树的基本结构， 它不包含子项目。
一般情况下， 叶节点最终会完成大部分的实际工作， 因为它们无法将工作指派给其他部分。

3\.  **容器** （Container）——又名 “组合 （Composite）”——是包含叶节点或其他容器等子项目的单位。 容器不知道其子项目所属的具体类， 它只通过通用的组件接口与其子项目交互。
容器接收到请求后会将工作分配给自己的子项目， 处理中间结果， 然后将最终结果返回给客户端。

4\.  **客户端** （Client） 通过组件接口与所有项目交互。 因此， 客户端能以相同方式与树状结构中的简单或复杂项目交互。

类图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_J-n-AmD9Td.png)

### 示例代码

```java
//顶层抽象
public abstract class Component {
    protected  String name;
    public Component(String name){
        this.name = name;
    }
    public abstract void add(Component c);
    public abstract void remove(Component c);
    public abstract void display(int depth);

}
//叶子，总之节点
public class Leaf extends Component{

    public Leaf(String name){
        super(name);
    }

    @Override
    public void add(Component c) {
        System.out.println("Cannot add a leaf");
    }

    @Override
    public void remove(Component c) {
        System.out.println("Cannot remove from a leaf");
    }

    @Override
    public void display(int depth) {
        String str = "";
        for (int i=0; i<depth; i++){
            str = str + "-";
        }
        System.out.println(str + name);
    }
}
//组合类
public class Composite extends Component{

    public Composite(String name){
        super(name);
    }
    //含有一组相似的对象
    private List<Component> children = new ArrayList<>();

    @Override
    public void add(Component c) {
        children.add(c);
    }

    @Override
    public void remove(Component c) {
        children.remove(c);
    }

    @Override
    public void display(int depth) {
        String str = "";
        for (int i=0; i<depth; i++){
            str = str + "-";
        }
        System.out.println(str + name);
        for (Component component : children){
            component.display(depth + 2);
        }
    }

```

### 应用场景

-   如果你需要实现树状对象结构， 可以使用组合模式。
-   如果你希望客户端代码以相同方式处理简单和复杂元素， 可以使用该模式。
