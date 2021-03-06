<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [第三章：Spring高级话题>](#%E7%AC%AC%E4%B8%89%E7%AB%A0spring%E9%AB%98%E7%BA%A7%E8%AF%9D%E9%A2%98)
      - [<a name="spring">Spring Aware</a>](#a-namespringspring-awarea)
      - [<a name ="duo">多线程</a>](#a-name-duo%E5%A4%9A%E7%BA%BF%E7%A8%8Ba)
      - [<a name="jihua">计划任务(定时任务)</a>](#a-namejihua%E8%AE%A1%E5%88%92%E4%BB%BB%E5%8A%A1%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1a)
      - [<a name="tiao">条件注解@Conditional</a>](#a-nametiao%E6%9D%A1%E4%BB%B6%E6%B3%A8%E8%A7%A3conditionala)
      - [<a name="zu">组合注解与元注解</a>](#a-namezu%E7%BB%84%E5%90%88%E6%B3%A8%E8%A7%A3%E4%B8%8E%E5%85%83%E6%B3%A8%E8%A7%A3a)
      - [<a name="enable">@Enable*注解的工作原理</a>](#a-nameenableenable%E6%B3%A8%E8%A7%A3%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86a)
      - [<a name="ceshi">测试</a>](#a-nameceshi%E6%B5%8B%E8%AF%95a)
        - [上一篇：<a href="https://github.com/MrAlan/University/blob/master/SpringBoot%E5%AE%9E%E6%88%98/%E6%9C%89%E5%85%B3%E7%AC%AC%E4%BA%8C%E7%AB%A0%E5%90%84%E7%A7%8D%E4%B8%8D%E6%87%82.md">第二章——Spring 常用配置</a>](#%E4%B8%8A%E4%B8%80%E7%AF%87a-hrefhttpsgithubcommralanuniversityblobmasterspringbootmd%E7%AC%AC%E4%BA%8C%E7%AB%A0spring-%E5%B8%B8%E7%94%A8%E9%85%8D%E7%BD%AEa)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

*《springboot 实战》汪云飞编著*

<h1>第三章：Spring高级话题></h1>

---

#### <a name="spring">Spring Aware</a>

​	Spring 的依赖注入最大亮点就是你所有的Bean对Spring容器的存在是没有意识的。你可以将容器换成其他容器，如 Google Guice，这时Bean之间的耦合度很低。

​	但是实际项目中，不可避免的要用到Spring容器本身的功能资源，这时Bean就得意识到容器的存在，才能调用资源。这就是 Spring Aware ，让Bean与容器耦合。

​	以下是Spring 提供的 Aware接口

| BeanNameAware                  | 获得容器中Bean的名称                 |
| ------------------------------ | ------------------------------------ |
| BeanFactoryAware               | 获得当前bean factory                 |
| ApplicationConteAware*         | 获得当前的application                |
| MessageSourceAware             | 获得message source，获得文本信息     |
| ApplicationEventPublisherAware | 应用时间发布事件                     |
| ResourceLoaderAware            | 获得资源加载器，可以获得外部资源文件 |

​	Spring Aware 的目的是让Bean获得Spring容器的服务。(因为ApplicationContext 接口集成了MessageSource，ApplicationEventPublisher 和ResourceLoader 的接口，所以只要Bean继承了ApplicationContextAware 就可以获得Spring 容器的所有服务，但原则上不提倡这么做)



​	下面演示一下，BeanNameAware 和 ResourceLoaderAware ；往包里加一个test.txt，内容随意。如果是IDEA需要手动将其复制到对应的target/classes 包中

```java
package ch31.aware;

import org.apache.commons.io.IOUtils;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.context.ResourceLoaderAware;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.stereotype.Service;

@Service
public class AwareService implements BeanNameAware, ResourceLoaderAware {

  private String beanName;
  private ResourceLoader loader;

  /*
  	下面两个方法都是继承自上述两个接口，也不用过多的配置，当这个类被生成Bean之后，就会获得这两个属性
  	就像是类中的构造函数，调用了这两个方法。
  */
  @Override
  public void setBeanName(String name) {  
    beanName=name; //获得容器中Bean的名称
  }

  @Override
  public void setResourceLoader(ResourceLoader resourceLoader) {
    loader=resourceLoader; //获得资源加载器 
  }

  public void outputResult(){
    System.out.println("bean 的名称为"+beanName);
    Resource resource=loader.getResource("ch31/aware/test.txt");

    try{
      System.out.println("ResourceLoader 加载的文件内容为："+ IOUtils.toString(resource.getInputStream()));
    }catch (Exception e){
      e.printStackTrace();
    }
  }
}
```

```java
package ch31.aware;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan("ch31.aware")
public class AwareConfig {

}
```

```java
package ch31.aware;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
  public static void main(String []args){
    AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext(AwareConfig.class);
    AwareService awareService=context.getBean(AwareService.class);
    awareService.outputResult();
    context.close();
  }
}
```

---

#### <a name ="duo">多线程</a>

​	Spring 通过任务执行器 (***TaskExecutor***)  来实现多线程与并发编程。使用*ThreadPoolTaskExecutor* 可实现一个基于线程池的*TaskExecutor* 。而实际上开发中一般是异步的，要在配置类中通过***@EnableAsync*** 开启对异步任务的支持，并通过在实际执行的Bean 方法中使用 ***@Async*** 注解其是一个 异步任务：整个过程非常的easy，只要熟悉了Spring的配置之后。

```java
package ch32.taskexecutor;

import java.util.concurrent.Executor;
import org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.AsyncConfigurer;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

@Configuration
@ComponentScan
@EnableAsync
public class TaskExecutorConfig implements AsyncConfigurer {

    @Override
  public Executor getAsyncExecutor() {  //构造异步任务 执行器，被注解为异步任务的配置

     //在线程池 中初始化任务执行器  ，ThreadPoolTaskExecutor是Spring中最常用的
    ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
      //taskExecutor 被创建来为其他组件提供线程池调用的抽象。
      
    taskExecutor.setCorePoolSize(5); //线程池维护线程的最少数量

    taskExecutor.setMaxPoolSize(10); //线程池维护线程的最大数量

    taskExecutor.setQueueCapacity(25); //线程池使用的缓冲队列，当线程池满了之后，多余的就会进入缓冲队列

    taskExecutor.initialize();
    return taskExecutor;

  }
    
 @Override
  public AsyncUncaughtExceptionHandler getAsyncUncaughtException() {
    return null;
  }
}
```

```java
package ch32.taskexecutor;

import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

@Service
public class AsyncTaskService {
  @Async //定义为异步任务 , 
  public void executeAsyncTask(Integer i){
    System.out.println("执行异步任务: "+i);
  }

  @Async
  public void executeAsyncTaskPlus(Integer i){
    System.out.println("执行异步任务+1 :"+(i+1));
  }
}
```

```java
package ch32.taskexecutor;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

  public static void main(String[] args) {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(
        TaskExecutorConfig.class);
    AsyncTaskService asyncTaskService = context.getBean(AsyncTaskService.class);
    for (int i = 0; i < 10; i++) {
      asyncTaskService.executeAsyncTask(i); //两个异步任务 一起执行才能看得出来是异步的并发执行
      asyncTaskService.executeAsyncTaskPlus(i);
    }
    context.close();
  }
}
```

---

#### <a name="jihua">计划任务(定时任务)</a>

​	**@EnableScheduling** 开启对计划任务的支持，执行计划任务的用 **@Scheduled** 注解。Spring 支持多种类型的计划任务，包含下面三个属性  @Scheduled(fixDelay=3000)  类似这样

 1. cron ：用时间表达式（cron=" 0 28 11 ? * *"）（每天11.28分执行） 指示在指定时间执行

 2. fixedDelay ：上一个调用完成后，再次开始的延迟时间

 3. fixedRate ： 上一个调用开始后，下一个调用的延迟时间(不用等上一个跑完)，会造成重复执行的问题，数据量不大时，则会变成调用多次。不建议使用

    ```java
    package ch33.taskschedule;
    
    import java.text.SimpleDateFormat;
    import java.util.Date;
    import org.springframework.scheduling.annotation.Scheduled;
    import org.springframework.stereotype.Service;
    
    @Service
    public class ScheduledTaskService {
      private static final SimpleDateFormat dateFormat=new SimpleDateFormat("HH:mm:ss");
    
      @Scheduled(fixedRate = 5000)
      public void reportCurrentTime(){
        System.out.println("每隔5秒执行一次");
      }
    
      @Scheduled(cron = "0 28 11 ? * *")
      public void fixTimeExecution(){
        System.out.println("在指定时间"+dateFormat.format(new Date())+"执行");
      }
    }
    ```

    ```java
    package ch33.taskschedule;
    
    import org.springframework.context.annotation.ComponentScan;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.scheduling.annotation.EnableScheduling;
    
    @Configuration
    @ComponentScan
    @EnableScheduling
    public class TaskSchedulerConfig {
    
    }
    ```

    ```java
    package ch33.taskschedule;
    
    import org.springframework.context.annotation.AnnotationConfigApplicationContext;
    
    public class Main {
      public static void  main(String[]args){
        AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext(TaskSchedulerConfig.class);
    //    ScheduledTaskService taskService=context.getBean(ScheduledTaskService.class);
    //    taskService.reportCurrentTime();
    //    taskService.fixTimeExecution();  注释掉这三行之后
      }
    
    }
    ```

    ​	程序按照我们设想的情况执行。

    ![1541837760985](https://github.com/MrAlan/MyPostPicture/blob/master/8.png?raw=true)

    ```java
    package ch33.taskschedule;
    
    import org.springframework.context.annotation.AnnotationConfigApplicationContext;
    
    public class Main {
      public static void  main(String[]args){
        AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext(TaskSchedulerConfig.class);
        ScheduledTaskService taskService=context.getBean(ScheduledTaskService.class);
        taskService.reportCurrentTime();
        taskService.fixTimeExecution();  
      }
    
    }
    ```

     	

    ​	按照正常的Spring 配置，出现了和上面不一样的情况。这是由于我们手动调用了两个方法，但是并没有影响它后续的工作。这个差别体现除了，不用构造出 TaskScheduleService 的 Bean ，Spring就把这两个方法从bean中抽取出来，自动运行了。

    ![1541837688622](https://github.com/MrAlan/MyPostPicture/blob/master/9.png?raw=true)



---

#### <a name="tiao">条件注解@Conditional</a>

​	在2.4节中学到，通过活动的profile，可以获得不同环境下的Bean，如dev和prod。Spring 4 提出了一个更加通用的基于 条件的Bean 的创建。 即使用 @Conditional 注解。

​	@Conditional 根据满足某一个特定条件创建一个特定的 Bean 。比方说 当某一个jar包在一个类 路径下的时候，自动配置一个或多个 Bean ；或者只有某个Bean 被创建时才会创建另一个Bean的情况 。总的来说，就是根据特定条件来控制Bean的创建行为，这样我么可以利用这个特性进行一些自动配置。

​	下面以不同操作系统为条件，我们通过实现Condition接口并重写matches方法来构造判断条件

```java
package ch34.conditional;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class WindowsCondition implements Condition {

  @Override
  public boolean matches(ConditionContext conditionContext,
      AnnotatedTypeMetadata annotatedTypeMetadata) {
    return //conditionContext.getEnvironment().getProperty("os.name").contains("windows");
//就是 这么一个大小写 的错误 ，我查了整整 1个多小时，难受
  conditionContext.getEnvironment().getProperty("os.name").contains("Windows"); 
  }
}
```

```java
package ch34.conditional;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class LinuxCondition implements Condition {

  @Override
  public boolean matches(ConditionContext conditionContext,
      AnnotatedTypeMetadata annotatedTypeMetadata) {
    return conditionContext.getEnvironment().getProperty("os.name").contains("Linux");
  }
}
```

```java
package ch34.conditional;

public interface ListService {
   String showListCmd();
}
```

```java
package ch34.conditional;

public class LinuxListService implements ListService {

  @Override
  public String showListCmd() {
    return "ls"; //返回Linux下 目录
  }
}
```

```java
package ch34.conditional;

public class WindowsListService implements ListService {

  @Override
  public String showListCmd() {
    return "dir"; //返回 windows下的目录
  }
}
```

```java
package ch34.conditional;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

// ComponentScan 就是用来搜索Bean的 ，本包中用到的Bean 都在下面有定义了，就不需要了
@Configuration
public class ConditionConfig {
  @Bean
  @Conditional(WindowsCondition.class)
  public ListService windowsListService(){
    return new WindowsListService();
  }

  @Bean
  @Conditional(LinuxCondition.class)
  public ListService linuxListService(){
    return new LinuxListService();
  }

}
```

```java
package ch34.conditional;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
 
  public static void main(String[]args){
    AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext(ConditionConfig.class);
    ListService listService=context.getBean(ListService.class);
      
    System.out.println(context.getEnvironment().getProperty("os.name")
        + "系统下的列表命令为: "
        + listService.showListCmd()
    );
  }

}
```

![1541840905756](https://github.com/MrAlan/MyPostPicture/blob/master/10.png?raw=true)

结果发现出错了，No qualifying bean of type：  NoSuchBeanDefinitionException 没有找到这个Bean。

经过一个多小时的奋战，终于找出了错误。  我在写WindowsCondition的时候，conditionContext.getEnvironment().getProperty("os.name").contains("windows"); 里的 windows  w没有大写。

---

#### <a name="zu">组合注解与元注解</a>

​	从Spring2 开始就大量采用注解来配置注入 Bean以及切面相关的配置。多个注解用到统一个类中会相当啰嗦。Spring的很多注解都可以作为元注解，@Configuration 就是一个组合@Component的注解，这个类其实也是个Bean。下面我们将 @Configuration 和 @ComponentScan 两个注解组合到一起。

```java
package ch35.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Target(ElementType.TYPE)   //表示对一个类 进行注解
@Retention(RetentionPolicy.RUNTIME) //注解的 生存周期
@Documented // 嵌入javadoc文件中   这三个就像是固定配置
@Configuration
@ComponentScan
public @interface WiselyConfiguration {
  String [] value() default {};
}
```

```java
package ch35.annotation;

import org.springframework.stereotype.Service;

@Service
public class DemoService {

  public void outputResult() {
    System.out.println("从组合注解配置照样获得的 bean");
  }
}
```

```java
package ch35.annotation;
@WiselyConfiguration("ch35.annotation")
public class DemoConfig {

}
```

```java
package ch35.annotation;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
  public static void main(String[]args){
    AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext(DemoConfig.class);
    DemoService service=context.getBean(DemoService.class);
    service.outputResult();
    context.close();
  }

}
```

---

#### <a name="enable">@Enable*注解的工作原理</a>

我们之前通过 

​	@EnableAspectJAutoProxy 开启对AspectJ 自动代理的支持，

​	@EnableAsync 开启异步方法的支持

​	@EnableScheduling 开启计划任务的支持；

接下来的第二部分我们将学习

​	 @EnableWebMvc 开启Web MVC 的配置支持

第三部分 

​	@EnableJpaRepositories 开启 Spring Data JPA Repository 的支持

​	@EnableTransactionManagement 开启注解式事务的支持

​	@EnableCaching 开启注解式的缓存支持

通过简单的 @Enable* 来开启一项功能的支持，从而避免自己配置大量的代码，大大降低了使用难度。那么这个神奇的功能实现原理是什么呢？

​	作者通过观察这些@Enable* 注解的源码，发现所有的注解都有一个@Import 注解(用来导入配置类的) ,这也就是意味着 开启这些功能的同时 导入了一些自动配置的Bean，导入的方式分为以下三种。

1.第一类：直接导入配置类

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(SchedulingConfiguration.class)  //直接导入配置类
@Documentd
public @interface EnableScheduling{
    
}
```

2.第二类：依据条件选择配置类

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(AsyncConfigurationSelector.class)  //
@Documentd
public @interface EnableAsync{
    Class<? extends Annotation> annotation() default Annotation.class;
    boolean proxyTargetClass() default false;
    AdviceMode mode() default AdviceMode.PROXY;
    int order() default Ordered.LOWEST_PRECEDENCE;
}
```

AsyncConfiguration 通过条件来选择需要导入的配置类， 它的根接口为ImportSelector ，这个接口需要重写selectImorts方法， 在此方法内进行事先条件判断 。

3.第三类：动态注册 Bean

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(AspectJAutoProxyRegister.class)  
@Documentd
public @interface EnableScheduling{
    boolean proxyTargetClass() default false;
}
```



---

#### <a name="ceshi">测试</a>

​	测试时开发工作中不可缺少的部分。单元测试只是针对当前开发的类和方法进行测试，可简单的通过模拟依赖(Mock) 来实现，对于运行环境没有依赖。但是仅仅是单元测试是不够的，它只能验证当前类或方法能否正常工作，而我们想要知道系统的各个部分组合在一起是否能正常工作，这就是 **集成测试**的存在意义。

​	集成测试一般需要来自不同层的不同对象的交互，如数据库，网络连接，IoC容器等。其实我们也可以通过正常运行程序，进行一些验证操作来进行测试。但是手工测试难免不准确，集成测试提供了一种无须部署或运行程序来完成系统各部分是否正常协同工作的能力。

​	Spring 通过 **Spring TestContext FrameWork** 对集成测试 提供顶级支持。不依赖于特定的测试框架，既可以使用JUnit 也可以使用TestNG。

​	基于Maven 构建的项目结构默认有关于测试的目录：src/test/java （测试代码），src/test/resources （测试资源）。

​	Spring 提供了一个 SpringJUnit4ClassRunner 类，它提供 *Spring TestContext FrameWork*功能。 通过@ContextConfiguration 来配置 Application Context 。通过 @ActiveProfiles 确定活动的 profile。

​	在使用Spring测试之后，我么前面的例子 运行部分可以用 Spring 测试来检验功能是否运行正常。集成测试涉及程序中的各个分层，在后续部分会有更多的描述。

​	先将Spring的测试依赖包 加到maven中。

```
	<dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>4.2.4.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
    </dependency>
```

```java
package ch37.fortest;
//没有Spring注解 说这个是Bean ，ComponentScan 自动扫描不到，但是可以手动返回一个Bean
//如下面一块 那样 注解一个 @Bean  
public class TestBean { 
    //这里之所以没有注解为Bean ，是因为有两种环境(dev,prod)，如果这里注解为Bean了就只有一种情况， 
 //就只有一种情况，所以在 配置类中 用了两个方法 返回了两个Bean 
  private String content;  

  public TestBean(String content){
    this.content=content;
  }

  public String getContent() {
    return content;
  }

  public void setContent(String content) {
    this.content = content;
  }
}
```

```java
package ch37.fortest;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
public class TestConfig {
  @Bean  //要返回一个 Bean  与Autowired 对应
  @Profile("dev")
  public TestBean devTestBean(){
    return new TestBean("from development profile");
  }

  @Bean
  @Profile("prod")
  public TestBean prodTestBean(){
    return new TestBean("from production profile");
  }
}
```

```java
//这是在Test的路径内
import ch37.fortest.TestBean;
import ch37.fortest.TestConfig;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestConfig.class})
@ActiveProfiles("prod")
public class DemoBeanIntegrationTests {
  @Autowired   //自动匹配个 TestBean  。 与 上面配置文件 返回的Bean对应 
  private TestBean testBean;
  @Test
  public void prodBeanShouldInject(){
    String expected="from production profile";  //expected  即 @ActiveProfiles()里的参数
    String actual=testBean.getContent();
    Assert.assertEquals(expected,actual);
  }

}
```



---

在看这本书的时候，我也找了网上的资料，网上很多看这本书之后写的文章。但是感觉要么没有代码，要么排版就很乱，可能我自己也有这个问题吧。只有看别人的东西才能挑的出错。

##### 上一篇：<a href="https://github.com/MrAlan/University/blob/master/SpringBoot%E5%AE%9E%E6%88%98/%E6%9C%89%E5%85%B3%E7%AC%AC%E4%BA%8C%E7%AB%A0%E5%90%84%E7%A7%8D%E4%B8%8D%E6%87%82.md">第二章——Spring 常用配置</a>
