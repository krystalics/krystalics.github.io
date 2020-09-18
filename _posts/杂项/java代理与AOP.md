## <a name='daili'>Java代理</a>

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



## [AOP](#aop)

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

------

