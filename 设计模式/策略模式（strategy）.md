# 策略模式（strategy）

-   **策略模式**是一种行为设计模式， 它能让你定义一系列算法， 并将每种算法分别放入独立的类中， 以使算法的对象能够相互替换。

第1列

1\.  **上下文** （Context） 维护指向具体策略的引用， 且仅通过策略接口与该对象进行交流。

2\.  **策略** （Strategy） 接口是所有具体策略的通用接口， 它声明了一个上下文用于执行策略的方法。

3\.  **具体策略** （Concrete Strategies） 实现了上下文所用算法的各种不同变体。

4\. 当上下文需要运行算法时， 它会在其已连接的策略对象上调用执行方法。 上下文不清楚其所涉及的策略类型与算法的执行方式。

5\. **客户端** （Client） 会创建一个特定策略对象并将其传递给上下文。 上下文则会提供一个设置器以便客户端在运行时替换相关联的策略。

类图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_4G0nJRUW77.png)



### 代码示例

<https://refactoringguru.cn/design-patterns/strategy/java/example>

#### 使用场景

-   使用策略模式可以避免使用多重条件语句，如 if...else 语句、switch...case 语句
-   什么是Spring的 InstantiationStrategy
-   线程池拒绝策略
