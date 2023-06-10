# 中介者模式（mediator）

-   中介者模式(Mediator Pattern)：用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，减少对象间混乱的依赖关系，从而使其耦合松散，而且可以独立地改变它们之间的交互。对象行为型模式

#### 具体信息

<https://refactoringguru.cn/design-patterns/mediator>

#### 使用场景

-   SpringMVC 的 DispatcherServlet是一个中介者，他会提取Controller、Model、View来进行调用。而无需controller直接调用view之类的渲染方法
-   分布式系统中的网关
-   迪米特法则的一个典型应用
