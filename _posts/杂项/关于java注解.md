**下面一大段来参考** https://blog.csdn.net/briblue/article/details/73824058 <br>
关于注解首先来段比较官方的解释：

> 注解是一系列元数据，它提供数据用来解释程序代码，但是注解并非是所解释的代码本身的一部分。注解对于代码的运行效果没有直接影响。
> 注解有许多用处，主要如下： 
>
> - 提供信息给编译器： 编译器可以利用注解来探测错误和警告信息 
> - 编译阶段时的处理： 软件工具可以用来利用注解信息来生成代码、Html文档或者做其它相应处理。 
> - 运行时的处理： 某些注解可以在程序运行的时候接受代码的提取

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

> 1. RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。
> 2. RetentionPolicy.CLASS 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。 
> 3. RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。

1. @Documented 它的作用是能够将注解中的元素包含到 Javadoc 中去
2. @Target 指定了注解运用的地方 没有指定的条件下都可以应用，取值如下

> - ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
> - ElementType.CONSTRUCTOR 可以给构造方法进行注解
> - ElementType.FIELD 可以给属性进行注解
> - ElementType.LOCAL_VARIABLE 可以给局部变量进行注解
> - ElementType.METHOD 可以给方法进行注解
> - ElementType.PACKAGE 可以给一个包进行注解
> - ElementType.PARAMETER 可以给一个方法内的参数进行注解
> - ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举

1. @Inherited ：如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。

```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@interface Test {}


@Test
public class A {}


public class B extends A {}  //B就继承了 Test注解 
```

1. @Repeatable 让注解可重复在一个类上使用

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



## <a name='java'>Java配置与注解异同</a>

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