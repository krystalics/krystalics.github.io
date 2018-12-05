### 有关于前面的配置像Maven，IDEA ULtimate 版的就不细说了。这属于基础的东西。这篇文章就从我不懂的地方说起吧。  
### 0. <a href="#zhujie">注解</a>
### 1. <a href="#di"> DI与IoC</a>
### 2. <a href="#spring">Spring与Spring容器</a>
### 3. <a href="#java">Java配置与注解有什么异同</a>
### 4. <a href="#daili">java代理</a>
### 5. <a href="#aop">AOP</a>

---
<a name="zhujie">注解</a>
---

**下面一大段来参考** https://blog.csdn.net/briblue/article/details/73824058 <br>
关于注解首先来段比较官方的解释：
>注解是一系列元数据，它提供数据用来解释程序代码，但是注解并非是所解释的代码本身的一部分。注解对于代码的运行效果没有直接影响。
注解有许多用处，主要如下： 
>- 提供信息给编译器： 编译器可以利用注解来探测错误和警告信息 
>- 编译阶段时的处理： 软件工具可以用来利用注解信息来生成代码、Html文档或者做其它相应处理。 
>- 运行时的处理： 某些注解可以在程序运行的时候接受代码的提取


在我的理解中，这段文字可以说是狗屁不通。谈到注解很容易就想到注释，注释是给我们人看的，但是有时候写一大段文字只有其中几个关键字是有效的。在我的理解中注解释高度精炼过的注释，而且机器也能看懂。相当于对被注解的东西打上了一个标签。可以理解为将某些固定的注释贴在了这上面
```java
public @interface TestAnnotation {
    // @interface 关键字定义注解 
}

@TestAnnotation
public class A{
    // 对A贴上TestAnnotation 标签
}
```
在Java中想要我们定义的注解能够生效，必须涉及到元注解，即官方JDK给我们的注解。
1. @Retention 英文意为保留期的意思。当 @Retention 应用到一个注解上的时候，它说明了这个注解的的存活时间。它有三个取值
>1. RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。
>2. RetentionPolicy.CLASS 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。 
>3. RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。

2. @Documented 它的作用是能够将注解中的元素包含到 Javadoc 中去
3. @Target 指定了注解运用的地方 没有指定的条件下都可以应用，取值如下
>- ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
>- ElementType.CONSTRUCTOR 可以给构造方法进行注解
>- ElementType.FIELD 可以给属性进行注解
>- ElementType.LOCAL_VARIABLE 可以给局部变量进行注解
>- ElementType.METHOD 可以给方法进行注解
>- ElementType.PACKAGE 可以给一个包进行注解
>- ElementType.PARAMETER 可以给一个方法内的参数进行注解
>- ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举

4. @Inherited ：如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。
```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@interface Test {}


@Test
public class A {}


public class B extends A {}  //B就继承了 Test注解 
```
5. @Repeatable 让注解可重复在一个类上使用
```java
@interface Persons {
    Person[]  value();
}


@Repeatable(Persons.class) //@Repeatable 后面括号中的类相当于一个容器注解。就是用来存放其它注解的地方。它本身也是一个注解。
@interface Person{
    String role default "";
}


@Person(role="artist")
@Person(role="coder")
@Person(role="PM")
public class SuperMan{

}
```
注解只有属性没有方法。应用注解时必须赋值这些属性。如下
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {

    int id() default 0;//虽然是属性，但是定义方法与一般类不同

    String msg() default "hello";//定义了默认值后，也可以不用对属性赋值，就像调用方法一样

}
@TestAnnotation(id=3,msg="hello annotation") //对属性进行赋值
public class Test {

}

//当注解中只有一个属性时，赋值时可以不用加 attr= ,而是直接像传参一样 
@TestAnnotation("hwllo world") //假设 只有msg属性，
public class B{}

//没有属性时 可以把括号都省略了
@TestAnnotation，
public class B{}
```

除了元注解之外，JDK还提供了一些基本的注解
1. @Deprecated 用来标记过时的元素 a.~~method()~~ 调用该方法时出现这样的标志
2. @Override 提示子类要复写父类中被 @Override 修饰的方法
3. 还有其他就不赘述

---
<a name="di">依赖注入</a>
---
很长一段时间我只是听过依赖注入，控制反转这两个词，我对它们只很模糊的认识。每次查完百度百科或者是哪篇博客文章我可能会对它有一点认知，等时间过了一段之后就又会忘记。直到我听了一节《软件体系结构》的课，讲设计原则SOLID时，刚好也把DI和IoC也过了一遍。

那是在讲继承和组合的时候给我们举的例子：
```java
    /*
    Suppose we want a variant of HashSet that keeps track of the number of attempted insertions.
    So we subclass HashSet as follows
    下面这段程序是想要知道Hashset加进了多少个元素。
    */
    public class InstrumentHashSet extends HashSet{
        private int addCount=0;
        public InstrumentHashSet(Collection c){super(c);}
        public InstrumentHashSet(int initCap,float loadFactor){
            super(initCap,loadFactor);
        }

        public boolean add(Object o){
            addCount++;
            return super.add(o);
        }

        public boolean addAll(Collection c){
            addCount+=o.size();
            return super.addAll(c); //HashSet是JDK提供的，它的addAll()里是循环调用add()，而add()方法已经被 InstrumentHashSet 重写了，所以addCount被加了两遍
        }
        public int getAddCount(){
            return addCount;
        }

        public static void main(String []args){
            InstrumentHashSet s=new InstrumentHashSet();
            s.addAll(Arrays.asList(new String[]{"1","2","3"}));//加了3个元素进去
            System.out.print(s.getAddCount()); // 结果却是： 6
        }
    }
    
```
![关于上述代码的解释样图](https://github.com/MrAlan/MyPostPicture/blob/master/1.png?raw=true)

```java
    public class InstrumentSet implements Set{
        private Set s;
        private int addCount=0;
        public InstrumentSet(Set s){this.s=s;} //组合 Composition
        
        public boolean add(Object o){
            addCount++;
            return s.add(o);
        }

        public boolean addAll(Collection c){
            addCount+=o.size();
            return s.addAll(c);
        }
        public int getAddCount(){
            return addCount;
        }
        ... //还有一些 Set必备的方法，毕竟是继承接口  这里用Set而不是HashSet是为了拓展性考虑
       // 毕竟Set是HashSet的父类

        public static void main(String []args){
            List list=new ArrayList();
            Set s=new InstrumentHashSet(new TreeSet(list));// 这里可以是TreeSet 而不仅仅是HashSet ，这就是用父类Set的好处，
            s.addAll(Arrays.asList(new String[]{"1","2","3"}));
            System.out.print(s.getAddCount()); //结果为3
        }

    }
```
上面两段代码大致的讲述了继承和组合，二者实现方式相差无几，但是结果却不同。因为继承时我们往往只是用父类的几个方法，却没想到会干扰整个生态。这并不是说继承不好，而是继承被滥用了(这是老师原话)，所以更推荐用Composition组合。

花了一大段讲继承与组合，就是为了引入DI依赖注入，IoC控制反转；<br>

![继承图片](https://github.com/MrAlan/MyPostPicture/blob/master/2.png?raw=true) AgentPassenger是继承而来<br>
![组合图片](https://github.com/MrAlan/MyPostPicture/blob/master/3.png?raw=true)将Passenger与Agent组合进Person构成AgentPassenger <br>

![组合与继承](https://github.com/MrAlan/MyPostPicture/blob/master/4.png?raw=true)将person可能的各个角色抽象成PersonRole 组合到Person中。

```java
    Person person=new Person(passenger,agent);//实质上我们不仅可以传入这两个，还有更多的参数可以传入，来决定是怎样一个Person

  
```
 **在继承中本来是由Person控制Passenger，Agent再进而控制AgentPassenger，但是****这里是由我们传进的参数控制Person，这是我理解中的控制反转IoC，Person 依赖于我们传进的参数，这是我理解的 DI依赖注入。很明显拓展性更好，耦合度更低**

___

<a name="spring">Spring与Spring容器</a>
---

**下面一部分参考** https://www.cnblogs.com/chenbenbuyi/p/7470834.html
在Spring出来之前,也就是没有依赖注入DI之前Java应用程序之间的类与类要实现互相功能协作是比较直观的，如果A类需要依赖B，就在A类中创建一个B类的对象。这就等于A类需要负责B类对象整个生命周期的管理。在系统不复杂时例如只有4，5个类时这并不会有不好的影响，但是更大一点的系统里面类与类之间的关系错综复杂，依赖关系也是。这种系统的耦合度非常高，复杂度以及修改难度很大。

Spring就是负责帮我们创建对象，搞定依赖关系的第三方组件。当某个类需要依赖另外的对象时，Spring会负责它的创建并交给它使用，通过依赖注入的方式。而Spring是如何注入的呢？就好像苹果并不在本部生产，而是通过第三方代工，比如富士康工厂。而Spring中生产对象的地方不叫工厂，而是叫做容器。容器的概念在java中有个很有名的例子。Tomcat就是一个正在运行的Servlet的web容器。

容器也负责创建对象的管理。容器是Spring框架实现功能的核心，还负责对象的整个生命周期，不只是创建，还包括装配，销毁。把对象的控制权交给容器，这就叫控制反转，IoC容器。

在一开始的时候Spring是用xml做配置文件，什么叫配置文件呢，就是告诉Spring系统中各类的依赖关系，后来有了注解配置和Java注解配置，xml就没用那么多了。

在学习SpringBoot的过程中还遇到了上下文context,(我最早接触到context还是学Android时) ， 应用上下文是Spring容器的抽象。容器大致分两类
1. BeanFactory 最简单的容器只提供基本的DI
2. 还有就是继承了BeanFactory的 应用上下文 
    1. ApplicationContext 提供更多企业级的服务，例如解析配置文本信息等这也是应用上下文实例对象最常见的应用场景
    2. AnnotationConfigApplicationContext: 从一个或多个基于java的配置类上加载上下文定义，使用于java注解的方式
    3. ClassPathXMLApplicationContext: 从类路径下的一个或多个基于xml配置文件中加载上下文定义，使用于xml配置方式
    4. FileSystemXmlApplicationContext: 从文件系统下的一个或多个xml文件上加载上下文定义，也就是说系统盘符中加载xml配置文件
    5. AnnotationConfigWebApplicationContext: 专门为web应用准备的，适用于注解方式
    6. XmlWebApplicationContext: 从web应用下的一个或多个xml配置文件加载上下文定义，适用于xml配置方式。

总的来说，无论基于xml还是java注解我们将需要管理的对象(Spring中称之为Bean)之间的写作关系配置好一般会有专门的Config类，在Main类中利用应用上下文对象加载进Spring容器 ，就能够运行程序了。

---

<a name='java'>Java配置与注解异同</a>
---

**注解配置：**
​    @Service 在业务逻辑层Service层里使用
​    @Component 组件 无明确角色 
​    @Repository 在数据访问层dao使用
​    @Controlle 在控制层(Action)

**Java配置：**
​    @Configuration 相当于xml配置文件
​    @Bean  用到方法上 表示当前方法返回值是一个Bean

这两种方法区别在于如果使用注解的方式，那么如果你要在Service层，DAO层时候需要在类上进行注解。

那么问题又来了什么是 *Service*层和*DAO层*呢？这又涉及到了分层概念；Java中的*Aciton Service Dao*层次划分是现在最基本的分层方式，结合了SSH(Spring Struts Hibernate)架构。
 1. Service层： 管理具体功能，引用对应的Dao数据库操作
 2. Action层： 管理业务(Service)调度和管理跳转，Action只负责管理，而Service负责实施，结合Struts配置文件，跳转到指定页面也接受页面的请求数据
 3. Dao 只完成增删改查。的封装，具体实现不负责。使用Hibernate连接数据库，操作数据库。

简单来说，Service就像是厨师，炒出具体的菜式，而Action就是服务员负责点菜上菜，Dao就是厨房的学徒，只是处理食材并不掌勺。

上文中谈到的SSH框架 ，Struts 控制作用   Hibernate 操作数据库  Spring 解耦

Spring负责把它们联系起来，因为Struts和Hibernate都要注入Spring配置文件。具体就不详解了。
```java
package di;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
//注解配置
@Service 
public class UseFunctionService {
    @Autowired
    FunctionService functionService;

    public String sayHello(String word) {
        return functionService.toHello(word);
    }
}

```

```java
package javaconfig;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JavaConfig {
    //通过这种方式，获得spring的依赖注入
    @Bean
    public UseFunctionService useFunctionService () {
        return new UseFunctionService ();
    }
}
```
两种方式无所谓优劣，一般来说：
1. 涉及到全局配置的，如数据库相关，MVC相关配置等，就用 Java配置
2. 涉及到业务配置的 就使用注解配置
---

<a name='daili'>Java代理</a>
---

**代理模式**：为其他对象提供一种代理以控制当前对象的访问：![代理示意图](https://github.com/MrAlan/MyPostPicture/blob/master/6.png?raw=true)<br>
实际上有三种代理模式：

1.**静态代理** 使用时需要定义接口，或者父类，被代理对象与代理对象Proxy 一起实现接口或继承。*这样便于管理，如果没有实现同一个接口，事实上也是可以的，不过实践证明实现同一个接口的代理更好用。* (上面斜体部分是个人猜测)


```java
public interface A{
    void test();
}

public class B implements A{
    void test(){
        System.out.println("我是被代理的");
    }
}

public class ProxyB implements A{
    private A target;
    public ProxyB(A target){
        this.target=target;
    }
    public void  test(){
        System.out.println("开始代理...");
        target.test();
        System.out.println("提交事务...")
    }
}

public class Main{
    public static void main(String []args){
        B taget =new B();
        ProxyB proxy=new ProxyB(target);
        proxy.test();
    }
}

```

**虽然可以不修改目标对象，就可以进行功能拓展，但是代理对象一样要实现目标对象的接口，也很麻烦,而且一旦接口改了，代理对象与目标对象都要进行更改**

2.**动态代理(JDK代理)**： 代理对象不需要实现接口 ，代理对象的生产是用的JDK的API，动态的在内存中创建代理对象，也叫接口代理。代理类所在的包 **java.lang.reflect.Proxy** 。JDK实现代理只需要调用下面这个方法

*newProxyInstance(ClassLoader loder,Class<?>[]interfaces,InvocationHandler h)*

ClassLoader loader 指定目标对象使用类加载器
Class<?> interfaces 目标对象实现的接口类型，使用泛型方式来确认类型
InvocationHandle h 事件处理器

```java
//沿用上述例子的 A 与 B

public class ProxyFactory{
    public Object target;//运用Object 保证对所有对象都成立
    public ProxyFactory(Object target){
        this.target=target;
    }
    public Object getProxyInstance(){
        return Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            new InvocationHandler(){
                @override
                public Object invoke(Object proxy,Method method,Object [] args) throws Throwable{
                    System.out.println("开始事务111");
                    object returnValue=method.invoke(target,args);
                    System.out.print("提交事务111");
                    return returnValue;
                    //这个代理工厂，可以说是所有需要代理的类都可以这么写，只是 开始事务，和提交事务不一样
                }
            }
        )
    }//返回Object对象，可以被转成需要用到的对象
}

public class Main{
    public static void main(String[]args){
        A target=new B();
        System.out.println(target.getClass());//打印出原始类型
        A proxy = (A) new ProxyFactory(target).getProxyInstance();//将得到的对象强制转为 A类型的
        System.out.println(proxy.getClass());//打印出 代理的原始类型
        proxy.test();
    }
}

//动态代理是通过getClassProxy0()生成代理类，JDK生成 最终的真正的代理类，它继承自Proxy并实现了我们定义的接口。
//通过Proxy.newProxyInstance()生成了代理类的实例对象，创建对象时传入InvocationHandler实例，获得增强方法
//调用新实例的方法，在此例中为 test() 即原InvocationHandler.invoke()方法

```

**注意 ：目标对象一定要实现接口，不然不能动态代理,这是由于JDK内部实现方式决定的，具体就不细究了**

3.**Cglib代理** 之前的两种代理都要求目标对象是实现了接口的，而Cglib代理原理是对目标类生成一个子类，并覆盖其中方法实现增强，但由于采用的继承不能用*final*修饰的类代理，因为final修饰的类是不可继承的。


```java
public class C{
    public void print(){
        System.out.println("我要打印东西");
    }
}

public class ProxyFactory implements MethodInterceptor{
    private Object target;
    public ProxyFactory (Object target){
        this.target=target;
    }
    public Object getProxyInstance(){
        Enhancer en=new Enhancer();//工具类
        en.setSuperClass(target.getClass());//设置被代理类为父类
        en.setCallback(this); //回调函数设置
        return en.create(); //创建 子类，即代理对象
    }

    @Override
    public Object intercept(Object obj,Method method,Object[]args,MethodProxy,proxy) throws Throwable{
        System.out.println("开始事务");
        Object returnValue=method.invoke(target,args);
        System.out.println("提交事务");
        return returnValue;
    }
}

public class Main{
    public static void main(String[]args){
        C target=new C();
        C proxy=(C)new ProxyFactory(target).getInstance();
        proxy.print();
    }
}

```

**Cglib子类代理实现方法**
1. 需要引入cglib的jar文件，但是Spring核心包中已经有了Cglib功能，所以直接引入Spring-core.jar即可
2. 引入功能包之后，在内存中动态构建子类，代理类不能为final
3. 而目标对象中如果有 **final/static** 修饰的方法，不会被拦截也不会被增强。

程序员只参与三个部分：
1. 定义普通业务组件
2. 定义切入点，一个切入点可能横切多个业务组件
3. 定义增强处理，曾强处理就是在AOP框架为普通业务组件织入的处理动作

一旦定义了合适的切入点和增强处理，AOP框架会自动生成AOP代理，  代理对象的方法=增强处理+被代理对象的方法

[AOP](#aop)
---

为什么在前面介绍代理呢？ 因为 AOP的实质是对代理模式的应用

**下面一段参考自** http://www.cnblogs.com/xrq730/p/4919025.html <br>

日常编程中完成某个业务中写了一堆代码，最后优化的时候发现就两句代码真正完成了业务，其他代码业务相关程度不高只是为了配置核心业务所需的环境。这种代码我们可以抽象成一个工具类。更抽象一点，各个业务模块的功能组件中除了完成相关业务功能外都有涉及日志，事务，安全控制等额外的操作等。这些模块并不是核心功能却又不可或缺。

所以就有了 AOP ，面向切面编程 。将遍布应用各处的功能分离出来形成重用的组件。可以说是OOP的补充与完善。OOP引入封装，继承，多态等概念来建立一种对象层次结构。用于模拟公共行为的一个集合。不过OOP允许开发者定义纵向的关系，但不适合定义横向关系，例如日志功能。可以说AOP是一个可以在不修改某个类本身就加强类功能的东西。就像是aop把该类包围了，拦截了所有该类与其他类的通信，并对其进行了加强。可以理解为一个拦截器框架，但会拦截类中的所有方法。就像对一个目标类的代理，增强了目标累的所有方法。<br>

![aop结构如图](https://github.com/MrAlan/MyPostPicture/blob/master/5.png?raw=true)

来个AspectJ(java的aop框架)的简单例子：运行aspectj需要配置环境和编译器。具体就不细说了
```java
public class Hello {

    public void sayHello(){
        System.out.println("hello!");
    }
    public static void main(String args[]){
        Hello hello =new Hello();
        hello.sayHello();
    }
}

```
```java
public aspect Demo { //aspect 关键字定义一个切面，可以是单独的日志切面

    /*
    切点就是那些需要应用切面的方法,如需要在sayHello方法执行前后进行权限验证和日志记录，那么就需要捕捉该方法，而pointcut就是定义这些需要捕捉的方法（常常是不止一个方法的），这些方法也称为目标方法
    */
    /*
    关键字pointcut，定义切点，后面跟着函数名称，最后编写匹配表达式，此时函数一般使用call()或者execution()进行匹配，上述的recordLog()是函数名称我们可以自定义。* 表示返回值， sayHello(..) 表示任意参数。  下面一句的意思是 :
    用pointcut 定义recordLog()切点函数 拦截Hello.sayHello()方法
    */
    pointcut recordLog():call(* Hello.sayHello(..));

    after():recordLog(){ // after() 在目标方法之后执行。定义日志记录切点
        System.out.println("sayHello方法执行后记录日志");
    }

    pointcut authCheck():call(* Hello.sayHello(..));//定义权限验证切点
    //(实际开发中日志和权限一般会放在不同的切面中,这里仅为方便演示)

    before():authCheck(){ // before() 在目标方法之前执行
        System.out.println("sayHello方法执行前验证权限"); 
    }

}
//文件时  .aj 文件
```
有5种通知：
1. before 目标方法执行前执行，前置通知
2. after 目标方法执行后执行，后置通知
3. after returning 目标方法返回时执行 ，返回通知
4. after thorwing 目标方法抛出异常时执行， 异常通知
5. around 在目标函数执行中 执行可控制目标函数是否执行，环绕通知

关于 通知的语法 ：中括号表示可有可无<br>
[返回值类型] 通知函数名称(参数) [returning(参数)/thorwing(参数)] 切点函数{函数体}
```java
在 aspect 文件中
after()returning(int x):get(){
    System.out.println('返回值为：'+x);
}

after()thorwing(Exception e):sayHello2(){
    System.out.println('抛出异常：'+e.toString());
}

返回值  通知函数  切点函数
Object around():aroundAdvice(){
    System.out.println('sayAround执行前 执行');
    Object result=proceed();
    System.out.println('sayAround执行后 执行');
    return result;
}

在java文件中

@AfterReturning(value="execution(* HelloWorld.sayHello(..))",returning="x")

@Before("execution(* HelloWorld.sayHello(..))")

@AfterThrowing(value="execution(* HelloWorld.sayHello(..))",throwing="exception")

@Pointcut  ....

//任意返回值，任意名称，任意参数的公共方法
execution(public * *(..))
//匹配com.zejian.service包及其子包中所有类的所有方法
within(com.zejian.service..*)
//匹配以set开头，参数为int类型，任意返回值的方法
execution(* set*(int))
```
为了方便类型（如接口、类名、包名）过滤方法，Spring AOP 提供了within关键字。其语法格式如下：
`within(<type name>)`   type name  使用包名或者类名替换即可，如
`@Pointcut("within(com.zejian.dao..*)")`

上述程序运行之后 结果为：<br>
sayHello方法执行前验证权限<br>
hello<br>
sayHello方法执行后记录日志<br>
很像是代理，却比代理简单太多了，从代码层面看。

把切面应用到目标函数的过程称为织入(weaving)，目标函数统称为连接点，切入点(pointcut)的定义正是从这些连接点中过滤出来的。而织入又分为静态与动态，其中Spring AOP 应用的是动态织入:运行时动态将要增强的代码织入到目标类中，这样往往是通过动态代理技术完成的，如Java JDK的动态代理(Proxy，底层通过反射实现)或者CGLIB的动态代理(底层通过继承实现) ，静态织入就不说了。

![关于AspectJ中的一些概念理解图](https://github.com/MrAlan/MyPostPicture/blob/master/7.png?raw=true)

---
写这篇长文花了我5天的时间，从开始看SpringBoot的书，里面各种我不懂的东西，我把他们记录下来是为了我更好的理解整个开发环境。写的不好，但是用心。

##### 下一篇：<a href="https://github.com/MrAlan/University/blob/master/SpringBoot%E5%AE%9E%E6%88%98/%E7%AC%AC%E4%B8%80%E7%AB%A0.md">第一章——Spring 基础</a> 