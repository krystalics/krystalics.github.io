<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [<a name='1'>Bean的scope</a>](#a-name1bean%E7%9A%84scopea)
- [<a name='2'>Spring EL 和资源调用</a>](#a-name2spring-el-%E5%92%8C%E8%B5%84%E6%BA%90%E8%B0%83%E7%94%A8a)
- [<a name='3'>Bean的初始化和销毁</a>](#a-name3bean%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96%E5%92%8C%E9%94%80%E6%AF%81a)
- [<a name='4'>Profile</a>](#a-name4profilea)
- [<a name='5'>事件 (Application Event)</a>](#a-name5%E4%BA%8B%E4%BB%B6-application-eventa)
      - [上一篇：<a href="https://github.com/MrAlan/University/blob/master/SpringBoot%E5%AE%9E%E6%88%98/%E7%AC%AC%E4%B8%80%E7%AB%A0.md">第一章——Spring基础</a>](#%E4%B8%8A%E4%B8%80%E7%AF%87a-hrefhttpsgithubcommralanuniversityblobmasterspringbootmd%E7%AC%AC%E4%B8%80%E7%AB%A0spring%E5%9F%BA%E7%A1%80a)
      - [下一篇：<a href="https://github.com/MrAlan/University/blob/master/SpringBoot%E5%AE%9E%E6%88%98/%E7%AC%AC%E4%B8%89%E7%AB%A0.md">第三章——Spring高级话题</a>](#%E4%B8%8B%E4%B8%80%E7%AF%87a-hrefhttpsgithubcommralanuniversityblobmasterspringbootmd%E7%AC%AC%E4%B8%89%E7%AB%A0spring%E9%AB%98%E7%BA%A7%E8%AF%9D%E9%A2%98a)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

*《Springboot 实战》汪云飞编著*

---
<a name='1'>Bean的scope</a>
---
scope描述的是Spring容器如何新建Bean实例的，Spring的Scope有以下几种，都是通过@Scope注解来实现的。
1. **Singleton**: 一个Spring容器只有一个Bean实例，全容器共享一个Bean。默认是Singleton => @Scope("singleton1")
2. **Prototype**: 每次调用新建一个Bean的实例
3. **Request**: Web项目中，给每个http request新建一个Bean实例
4. **Session**: Web项目中，给每个http session 新建一个Bean实例
5. **GlobalSession**： 这个只在portal应用中有用，给每个global http session 新建一个Bean实例

demo演示一下singleton和prototype的：分别从Spring容器中获得2次Bean，判断Bean的实例是否相等
```java
package ch4.scope;

import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Service;

@Service
@Scope("prototype")
public class DemoPrototypeService {

}
------------------------------------

package ch4.scope;

import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Service;

@Service
@Scope("singleton") //默认是这个 所以不写也没关系
public class DemoSingletonService {

}
-------------------------------------

package ch4.scope;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan   //后面没加参数，默认是扫描本包的所有类
public class ScopeConfig {

}
--------------------------------------

package ch4.scope;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

  public static void main(String[] args) {
    /*
    * 这个例子 是用来证实 singleton 是唯一实例，而其他的不是
    * */
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(
        ScopeConfig.class); //从配置中 获取容器
    DemoSingletonService ds1 = context.getBean(DemoSingletonService.class); //得到bean
    DemoSingletonService ds2 = context.getBean(DemoSingletonService.class);

    DemoPrototypeService dp1 = context.getBean(DemoPrototypeService.class);
    DemoPrototypeService dp2 = context.getBean(DemoPrototypeService.class);

    System.out.println("ds1 与 ds2 是否相等 +" + ds1.equals(ds2));
    System.out.println("dp1 与 dp2 是否相等 +" + dp1.equals(dp2));

  }
}

```
其他的Scope就没有演示了，等遇到了google就可以了。

---
<a name='2'>Spring EL 和资源调用</a>
---
**Spring EL-Spring** 表达式语言，支持在**xml**和**注解**中使用表达式，类似于JSP的EL表达式语言。在Spring开发日常中经常涉及调用各种资源的情况，包含普通文件，网址，配置文件，系统环境变量等。这些东西很麻烦，我们可以统一采用Spring的表达式语言实现 **资源注入**

Spring主要在 注解@Value的参数中使用表达式。注入文件内容时可能比较麻烦，所以引入依赖*commons-io*
```maven
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</groupId>
    <version>2.4</version>
</dependency>
```
新建test.txt，内容随意：用来做文件内容注入的demo。再创建一个test.properties文件,内容如下。注入properties文件： java中的properties文件是一种配置文件，主要用于表达配置信息，文件类型为**.properties*，格式为文本文件，文件的内容是格式是"键=值"的格式
```java
book.author=wangyunfei
book.name=spring boot
```
文件中，可以用"#"来作注释，properties文件在Java编程中用到的地方很多，操作很方便。  

**Attention*** ：如果是用IDEA做IDE的话，test.txt与test.properties需要手动复制到**target/classes**文件夹的对应目录下。target文件夹是IDE自动生成的，存放class或者包文件的地方。只有这样 @PropertySource("classpath:....")才可以搜索到。否则会报错

```java
package ch22.el;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

@Service
public class DemoService {
  @Value("其他类的属性")  //注入 普通字符
  private String another;

  public String getAnother() {
    return another;
  }

  public void setAnother(String another) {
    this.another = another;
  }
}

-------------------

package ch22.el;

import org.apache.commons.io.IOUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.env.Environment;
import org.springframework.core.io.Resource;

// 注入配置配件需要使用 @PropertySource 指定文件地址
//若是用 @Value注入，则使用 PropertySourcesPlaceholderConfigure 的Bean

@Configuration
@ComponentScan
@PropertySource("classpath:ch22/el/test.properties")  //这里的路径 指的是包的路径，java之后 开始算起
public class ElConfig {

  @Value("I love you")  //注入普通的属性
  private String normal;

  @Value("#{systemProperties['os.name']}")  //注入操作系统属性 注意是#
  private String osName;

  @Value("#{ T(java.lang.Math).random() * 100.0}") //注入表达式的结果 
  private double randomNumber;

  @Value("#{demoService.another}") //注入其他 Bean的属性 #号
  private String fromAnother;

  @Value("classpath:ch22/el/test.txt") //注入文件资源
  private Resource testFile;

  @Value("http://www.baidu.com")  //注入网址资源
  private Resource testUrl;

  @Value("${book.name}")  //注入 properties 配置文件，可以直接用里面的内容，但是注意是$ ，与之前的#区分开来
  private String bookName;

  @Value("${book.author}")
  private String bookAuthor;

  @Autowired  //通过此注解 在spring配置文件中 查找满足environment 同类型的bean 注入给它  @Resource 是寻找同名的 bean  注入
  private Environment environment;

  @Bean
  public static PropertySourcesPlaceholderConfigurer propertyConfigure() {
    return new PropertySourcesPlaceholderConfigurer();
  }

  public void outputResource() {
    try {
      System.out.println(normal);
      System.out.println(osName);
      System.out.println(randomNumber);
      System.out.println(fromAnother);
      System.out.println(testFile.getInputStream());
      System.out.println(IOUtils.toString(testUrl.getInputStream()));
      System.out.println(bookName);
      System.out.println(bookAuthor);
      System.out.println(environment.getProperty("book.author"));

    } catch (Exception e) {
      e.printStackTrace();
    }
  }


}

-----------------

package ch22.el;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
  public static void main(String[]args){
    AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext(ElConfig.class); //容器加载配置文件 
    ElConfig elConfig=context.getBean(ElConfig.class);
    elConfig.outputResource();
    context.close();
  }

}

```

*PS（在我个人的理解中 @AutoWired 是注入Bean版本的 @Value)*

---



<a name='3'>Bean的初始化和销毁</a>
---

​	在我们实际开发中，经常会遇到在Bean使用之前或之后做些必要的操作，Spring 对Bean的生命周期操作提供了支持。在使用Java配置与注解配置下提供两种方式

 	1. Java配置 ： 使用@Bean 的 initMethod 和 destroyMethod 
 	2. 注解方式： 利用JSR-250的@PostConstruct 和 @PreDestroy

```
<dependency>
  <groupId>javax.annotation</groupId>
  <artifactId>jsr250-api</artifactId>
  <version>1.0</version>
</dependency>
```

```java
package ch23.prepost;

public class BeanWayService {
  public void init(){
    System.out.println("@Bean-init-method");
  }
  public BeanWayService(){
    //super();  原书上有，但是去掉之后也没有影响
    System.out.println("初始化构造函数-BeanWayService");
  }
  public void destroy(){
    System.out.println("@Bean-destroy-method");
  }
}

```

```java
package ch23.prepost;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class JSR250WayService {
  @PostConstruct  //构造函数执行完之后
  public void init(){
    System.out.println("jsr250-init-method");
  }
  public JSR250WayService(){
    //super(); 去掉之后无影响
    System.out.println("初始化构造函数-JSR250构造函数");
  }
  @PreDestroy
  public void destroy(){
    System.out.println("jsr250-destroy-method");
  }
}
```

```java
package ch23.prepost;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan("ch23.prepost")
public class PrePostConfig {
  @Bean(initMethod = "init",destroyMethod = "destroy")
  BeanWayService beanWayService(){
    return new BeanWayService();
  }
  @Bean
  JSR250WayService jsr250WayService(){
    return new JSR250WayService();
  }
}
```

```java
package ch23.prepost;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
  public static void main(String []args){
    AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext(PrePostConfig.class);
    BeanWayService beanWayService=context.getBean(BeanWayService.class);
    JSR250WayService jsr250WayService=context.getBean(JSR250WayService.class);
    context.close();
  }
}
```

---


<a name='4'>Profile</a>
---

​	**profile**为在不同环境下使用不同的配置提供了支持（开发和生产环境肯定是不同的 ）

 1. 通过设定Environment的ActiveProfiles来设定当前context需要使用的配置环境。开发中使用@Profile注解类或者方法，达到不同情况下选择实例化不同的Bean。

 2. 通过设定jvm的Spring.profiles.active参数来配置环境

 3. Web项目设置在Servlet的context parameter中，Servlet3.0以上版本如下

    ```java
    public class WebInit implements WebApplicationInitializer{
        @Override
        public void onStartup(ServletContext container) thorws ServletException{
            container.setInitParameter("spring.profiles.default","dev");
        }
    }
    ```

    用demo演示一下，profile如何在不同的环境下获得不同的Bean

    ```java
    package ch24.profile;
    
    public class DemoBean {
    
      private String content;
    
      public DemoBean(String content){
    //    super();
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
    package ch24.profile;
    
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Profile;
    
    @Configuration
    public class ProfileConfig {
      @Bean
      @Profile("dev") //开发版
      public DemoBean devDemoBean(){
        return new DemoBean("from development profile");
      }
    
      @Bean
      @Profile("prod") //生产版
      public DemoBean prodDemoBean(){
        return new DemoBean("from production profile");
      }
    }
    
    ```

    ```java
    package ch24.profile;
    
    import org.springframework.context.annotation.AnnotationConfigApplicationContext;
    
    public class Main {
      public static void main(String[]args){
        AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext();//获得容器，但是暂无配置
        context.getEnvironment().setActiveProfiles("prod");//设置环境
        context.register(ProfileConfig.class); // 注册配置
        context.refresh();
    
        DemoBean demoBean=context.getBean(DemoBean.class);
        System.out.println(demoBean.getContent());  //from production profile
     
        context.close();
      }
    }
    ```

​	



---
<a name='5'>事件 (Application Event)</a>
---

​	**Spring**的事件为Bean与Bean之间的消息通信提供了支持。当一个Bean处理完一个任务后，希望另一个Bean知道并能做出相应的处理，这时就需要让另一个Bean监听当前Bean发送的事件

​	Spring的事件需要遵循如下流程：

 	1. 自定义事件，集成ApplicationEvent
 	2. 定义事件监听器，实现ApplicationListener
 	3. 使用容器发布事件

```java
package ch25.event;

import org.springframework.context.ApplicationEvent;


public class DemoEvent extends ApplicationEvent {

  private static final long serialVersionID = 1L;
  private String msg;

  public DemoEvent(Object source, String msg) {
    super(source);
    this.msg = msg;
  }

  public String getMsg() {
    return msg;
  }

  public void setMsg(String msg) {
    this.msg = msg;
  }
}
```

```java
package ch25.event;

import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

@Component  
public class DemoListener implements ApplicationListener<DemoEvent> {

  //ApplicationListener<DemoEvent>  表示它监听了 DemoEvent 这个事件
  public void onApplicationEvent(DemoEvent event) {
    String msg = event.getMsg();
    System.out.println("我(bean-demolistener)接收到了bean-demoPublisher发布的消息" + msg);
  }
}
```

```java
package ch25.event;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;

@Component
public class DemoPublisher {

  @Autowired //注入ApplicationContext用来发布事件，publishEvent
  ApplicationContext context;  

  public void publish(String msg) {
    context.publishEvent(new DemoEvent(this, msg));
  }
}
```

```java
package ch25.event;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class EventConfig {

}
```

```java
package ch25.event;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

  public static void main(String[] args) {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(
        EventConfig.class);
    DemoPublisher demoPublisher = context.getBean(DemoPublisher.class);
    demoPublisher.publish("hello application event");
    context.close();
  }
}
```

发布事件的核心代码是用注入的ApplicationContext，context.publishEvent(new DemoEvent(this,msg));   而监听事件 implements ApplicationListener<DemoEvent> 是关键，再用Spring解耦。Bean 之间基于耗时事件处理的通知和处理顺序就迎面而解了。

---

第二章到此结束，出乎意料的没有什么晦涩难懂的，可能是第一章已经把我不懂的一些基础的东西搞定了。还是感谢正在努力的自己。

##### 上一篇：<a href="https://github.com/MrAlan/University/blob/master/SpringBoot%E5%AE%9E%E6%88%98/%E7%AC%AC%E4%B8%80%E7%AB%A0.md">第一章——Spring基础</a>

##### 下一篇：<a href="https://github.com/MrAlan/University/blob/master/SpringBoot%E5%AE%9E%E6%88%98/%E7%AC%AC%E4%B8%89%E7%AB%A0.md">第三章——Spring高级话题</a>

