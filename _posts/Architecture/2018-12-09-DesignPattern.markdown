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

<img src="https://github.com/krystalics/MyPostPicture/blob/master/%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F%E7%B1%BB%E5%9B%BE.png？raw=true">

像上面这个类图中，Invoker.executeCommand() 就最终调用了Receiver.action()，并且没有接触到对方。通过接口将引用传递过去。[具体代码见这个库](https://github.com/krystalics/DesignPattern/tree/master/src/com/company/Command)



















---

参考资料：

Java设计模式 耿祥义

武汉理工大学——陈明俊

[面向对象的5个基本原则](https://www.jianshu.com/p/0e71b4967c36)