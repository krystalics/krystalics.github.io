### 设计模式--Java

---

**设计模式的关键在于组合与抽象**，通过各种方式在对象之间建立起松耦合关系。现在的趋势都是主程序中负责建立环境，装配对象，启动项目。达到一种伞形的结构，对用户是伞柄，用户只需要花费最少的时间来理解和利用整个系统。

每一个设计模式都是在描述一个在我们周围不断重复发生的问题，以及该问题解决方案的核心。目前被公认在设计模式领域最具有影响力的著作是《Design Patterns: Elements of Reusable Object—Oriented Software》简称《设计模式》，这就是著名的GOF(Gang of Four)之书，这个领域的奠基之作。

##### 面向对象的五个基本原则：

被称为SOLID原则，介绍请看 [维基百科](https://zh.wikipedia.org/wiki/SOLID_(%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1))  [面向对象基本原则](https://www.jianshu.com/p/0e71b4967c36)  

记住设计一个类的时候，不让该类面向具体类，而是面向接口。



##### 1.命令模式（别名：动作，事务）

将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可撤销的操作。

许多设计中经常会涉及到一个对象请求另一个对象调用其方法到达某种目的。如果直接调用，如下

```java
public class B{
    A a; //直接依赖A类，创建一个对象
}
```

耦合度高，而且有时候无法或不希望直接引用A，这种时候就可以用命令模式。

例子：比如我想要将老家特产送给我表弟，但是到我表弟家太远了不能直接送到，我会将东西交给快递，让顺丰快递给我表弟。通过中间人介绍的都是这种模式。

模式中包括四种角色：1. Receiver   2. Command 接口  3.Concrete Command   4.Invoker(请求者)

<img src="https://github.com/krystalics/MyPostPicture/blob/master/58.png?raw=true">

像上面这个类图中，Invoker.executeCommand() 就最终调用了Receiver.action()，并且没有接触到对方。通过接口将引用传递过去。[具体代码见这个库](https://github.com/krystalics/DesignPattern/tree/master/src/com/company/command)



##### 2.观察者模式（别名：依赖，发布-订阅）

​	定义对象间的一种**一对多的依赖关系**，当一个对象的**状态发生变化**时，**所有依赖于它的对象都得到通知，并自动更新。**

例子：我表哥把自己的信息放在了相亲网站上，其中百合网新登记了一些女生，通知他和其他人这个消息。

观察者模式中有四种角色：

1. Subject 主题  **就是相亲网站**

2. Observer 观察者  **在相亲网站登记过的人**
3. 具体主题 **就是百合网**
4. 具体观察者 是**我表哥和其他在相亲网站上登记过的人**

<img src="https://github.com/krystalics/MyPostPicture/blob/master/59.png?raw=true">

 [代码传送](https://github.com/krystalics/DesignPattern/tree/master/src/com/company/observer)



##### 3.装饰模式（别名：包装器）

​	动态地给对象添加一些额外的职责。就功能来说装饰模式相比生成子类更为灵活。Spring的AOP就是类似这样的。不需要改变原有类的代码，动态多展功能的一种成熟模式。

例子：就像是手枪可以装上消音器，我们日常生活中的衣服也像是这种情况。在不改变我们身体情况下，改变我们的外观。装饰模式的结构中包括四种角色：

- 抽象组件 Component   =>  人类
- 具体组件 = >  我，其他人类
- 装饰 Decorator  =>  衣服
- 具体装饰   =>   衬衫，T-shift...

<img src="https://github.com/krystalics/MyPostPicture/blob/master/60.png?raw=true">

[代码传送](https://github.com/krystalics/DesignPattern/tree/master/src/com/company/decorator)



##### 4.策略模式（别名：政策）

​	定义一系列算法，把它们一个个封装起来，并且使它们可相互替换。本模式使得算法可独立于使用它的客户而变化。它是处理算法的不同变体的一种成熟模式，策略模式是通过接口或者抽象封装算法的标识，即在接口中定义一个抽象方法，实现该接口的类将实现接口中的抽象方法。

策略模式的结构中包括三种角色：

- 策略 Strategy
- 具体策略
- 上下文 Context

<img src="https://github.com/krystalics/MyPostPicture/blob/master/61.png?raw=true">

[代码传送](https://github.com/krystalics/DesignPattern/tree/master/src/com/company/strategy)



##### 5.适配器模式（别名：包装器）**

​	将一个类的接口转换成客户希望的另外一个接口。Adapter模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

​	适配器模式是将一个类的接口（被适配者）转换成客户希望的另外一个接口（目标）的成熟模式，该模式中涉及有目标、被适配者和适配器。适配器模式的关键是建立一个适配器，这个适配器实现了目标接口并包含有被适配者的引用。

​	实例：用户已有一个两相的插座，但最近用户又有了一个新的三相插座。用户现在已经有一台洗衣机和一台电视机，洗衣机按着三相插座的标准配有三相插头，而电视机按着两相插座的标准配有两相插头。现在用户想用新的三相插座来使用洗衣机和电视机。

模式中包含三种角色：

- 目标(Target)
- 被适配者(Adaptee)
- 适配器(Adapter)

<img src="https://github.com/krystalics/MyPostPicture/blob/master/62.png?raw=true">

​	[代码传送](https://github.com/krystalics/DesignPattern/tree/master/src/com/company/adapter)

​	•目标（Target）和被适配者（Adaptee）是完全解耦的关系。

​	•适配器模式满足“开-闭原则”。当添加一个实现Adaptee接口的新类时，不必修改Adapter，Adapter就能对这个新类的实例进行适配。

##### 6.责任链模式：

​	使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

​	责任链模式是使用多个对象处理用户请求的成熟模式，责任链模式的关键是将用户的请求分派给许多对象，这些对象被组织成一个责任链，即每个对象含有后继对象的引用，并要求责任链上的每个对象，如果能处理用户的请求，就做出处理，不再将用户的请求传递给责任链上的下一个对象；如果不能处理用户的请求，就必须将用户的请求传递给责任链上的下一个对象。

实例：用户提交一个人的身份证号码，想知道该人是在北京，上海或者是天津居住。

模式结构中的角色：

- 处理者(Handle)
- 具体处理者(ConcreteHandler)

<img src="https://github.com/krystalics/MyPostPicture/blob/master/63.png?raw=true">

[代码传送](https://github.com/krystalics/DesignPattern/tree/master/src/com/company/responsibility_chain)

•责任链中的对象只和自己的后继是低耦合关系，和其他对象毫无关联，这使得编写处理者对象以及创建责任链变得非常容易。



•使用责任链的用户不必知道处理者的信息，用户不会知道到底是哪个对象处理了它的请求。



##### 7.外观模式：

​	为系统中的一组接口提供一个一致的界面，Façade模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。

​	外观模式是简化用户和子系统进行交互的成熟模式，外观模式的关键是为子系统提供一个称作外观的类，该外观类的实例负责和子系统中类的实例打交道。当用户想要和子系统中的若干个类的实例打交道时，可以代替地和子系统的外观类的实例打交道。

实例：  报社的广告系统有三个类CheckWord、Charge和TypeSetting类，各个类的职责如下：CheckWord类负责检查广告内容含有的字符数量；Charge类的实例负责计算费用；TypeSetting的实例负责对广告进行排版。使用外观模式简化用户和上述子系统所进行的交互。

模式中的角色：

- 子系统(Subsystem)
- 外观(Facade)

<img src="https://github.com/krystalics/MyPostPicture/blob/master/64.png?raw=true">

[代码传送](https://github.com/krystalics/DesignPattern/tree/master/src/com/company/facade)

•使客户和子系统中的类无耦合。

•外观只是提供了一个更加简洁的界面，并不影响用户直接使用子系统中的类。

•子系统中任何类对其方法的内容进行修改，不影响外观的代码。



##### 8.单例模式：

​	保证一个类仅有一个实例，并提供一个访问它的全局访问点。    单件模式是关于怎样设计一个类，并使得该类只有一个实例的成熟模式，该模式的关键是将类的构造方法设置为private权限，并提供一个返回它的唯一实例的类方法。

模式中的角色：

- 单例类(Singleton)

<img src="https://github.com/krystalics/MyPostPicture/blob/master/65.png?raw=true">

[代码传送](https://github.com/krystalics/DesignPattern/tree/master/src/com/company/singleton)

•单件类的唯一实例由单件类本身来控制，所以可以很好地控制用户何时访问它。

---

​	参考资料：

Java设计模式 耿祥义

武汉理工大学——陈明俊

[面向对象的5个基本原则](https://www.jianshu.com/p/0e71b4967c36)