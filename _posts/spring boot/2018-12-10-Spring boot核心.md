*《Spring Boot实战》汪云飞著*

---

### 第六章：Spring Boot 核心

#### 6.1 基本配置

##### 6.1.1 入口类和@SpringBootApplication

​	Spring Boot 通常有个名为 *[projectname]Application* 的入口类，入口类中有个main方法，用@SpringBootApplication注解，作为Spring Boot 项目的启动。

​	@SpringBootApplicatio是组合注解，源码如下：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documentd
@Inherited
@Configuration
@EnableAutoConfiguration //让Spring boot 根据类路径中的jar包依赖为当前项目进行自动配置
@ComponentScan
public @interface SpringBootApplication{
    Class<?> []exclude() default {}; 
    String []excludeName() default {};
}
/*
Spring Boot 会自动扫描@SpringBootApplication 所在类的同级包以及下级包中的Bean，若为JPA项目还可以扫描标注了@Entity 的实体类。 建议将入口类放置在java包下。
*/
```

##### 6.1.2 关闭特定的自动配置

​	`@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})` 在exclude属性中配置。

##### 6.1.3 修改Banner 

1. 在 Spring Boot 启动的时候会有一个默认的图案：一个由线段组成的Spring 单词。
2. 我们在 src/main/resources 下新建一个banner.txt
3. 通过 http://patorjk.com/software/taag 网站生成字符，如敲入 Great ，将生成的字符复制进banner.txt中
4. 启动项目后，图案就会变成下面这个这样：

```
  ________                      __   
 /  _____/______   ____ _____ _/  |_ 
/   \  __\_  __ \_/ __ \\__  \\   __\
\    \_\  \  | \/\  ___/ / __ \|  |  
 \______  /__|    \___  >____  /__|  
        \/            \/     \/      
```

##### 6.1.4 Spring Boot 的配置文件	

​		Spring Boot 使用一个全局的配置文件——application.properties 或 application.yml 在 src/main/resources 目录下或者是类路径的/config下。

##### 6.1.5 starter pom

​	Spring Boot 为我们提供了简化企业级开发绝大多数场景的 starter pom 。下面是官方提供的 starter pom 列表。这里列举出一部分。

| 名称                             | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| spring-boot-starter              | Spring Boot 核心starter，包含自动配置，日志，yaml配置文件的支持 |
| spring-boot-starter-actuator     | 准生产特性，用来监控和管理应用                               |
| spring-boot-starter-remote-shell | 提供基于ssh协议的监控和管理                                  |
| spring-boot-starter-aop          | 使用 spring-aop 和AspectJ 支持面向切面编程                   |
| spring-boot-starter-freemaker    | 对freemarker 模板引擎的支持                                  |
| spring-boot-starter-Tomcat       | spring boot 默认的Servlet容器Tomcat                          |
| .....                            | ......                                                       |

​	还有第三方的 starter pom

| 名称         | 地址                                                         |
| ------------ | ------------------------------------------------------------ |
| Handlerbars  | https://github.com/allegro/handlerbars-spring-boot-starter   |
| Apache Camel | https://github.com/apache/camel/tree/master/components/camel-spring-boot |
| ....         | ....                                                         |



##### 6.1.6 使用xml配置

​	虽然提倡无xml配置，但是有些特定的要求需要用xml配置。

```java
@ImportResource({"classpath:some-context.xml","classpath:another-context.xml"})
```



#### 6.2 外部配置

##### 6.2.1 命令行参数配置	

​	Spring Boot 可以基于jar包运行，用    `java -jar xx.jar` 命令可以打包成jar 。

​	可以通过`java -jar xx.jar --server.port=9090` 更改Tomcat端口号

##### 6.2.2 使用properties文件的属性

​	不仅可以用来配置，还可以当做存储文件使用。可以用@Value 来注入 在之前的章节里有介绍，但是都用@Value就很麻烦。SpringBoot提供了**基于类型安全的配置方式** 通过@ConfigurationProperties 将properties的属性和一个Bean的属性关联。

- 在application.properties增加属性如下。也可以新建一个properties文件

```properties
book.author=wjf
book.name=spring boot in action
```

- 引入配置依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency>
```

- 类型安全的Bean

```java
package com.example.springbootredis.ch1;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "book")
public class Temp {
  private String name;
  private String author;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public String getAuthor() {
    return author;
  }

  public void setAuthor(String author) {
    this.author = author;
  }
}
```

- 测试

```java
package com.example.springbootredis;

import com.example.springbootredis.ch1.temp;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class SpringbootredisApplication {

  @Autowired
  private Temp tem;

  @RequestMapping("/")
  public String index() {
    return "book name is " + tem.getName() + " and author is " + tem.getAuthor();
  }


  public static void main(String[] args) {
    SpringApplication.run(SpringbootredisApplication.class, args);
  }
}
```



#### 6.3 日志配置

​	Spring Boot 支持 Java Util Logging ，Log4J，Log4J2和 Logback作为日志框架。默认情况下使用 Logback 。

​	配置日志文件：  `loggin.file=D:/mylog/log.log` 

​	配置日志级别，格式为logging.level.包名=级别: `logging.level.org.springframework.web=DEBUG`

#### 6.4 Profile配置

​	Profile是Spring用来针对不同环境的。全局profile配置使用 application-{profile}.properties (如：application-prod.properties) 。下面做个简单的演示，prod 开8088端口，dev开9090端口。

​	application-prod.properties :  `server.port=8088`

​	application-dev.properties: `server.port=9090`

由application.properties 控制。`spring.profiles.active=dev` 或者 `spring-profiles.active=prod`来确定使用哪个配置文件 

#### 6.5 Spring Boot运行原理

​	虽然有Spring Boot 为我们自动配置，但是我们清楚了它的原理之后就可以更加流畅的使用它了。而且还可以在本节之后自定义一个 starter pom 。它的自动配置源码在 spring-boot-autoconfigure-2.1.0.x.jar 内。

​	可以通过在application.properties中加入`debug=true` 来确定项目运行时有哪些自动配置启用了，哪些没有。 后面的内容有些繁琐就不记录了。直接开始写starter pom吧。

- 新建一个的Maven项目。artifactID就叫 spring-boot-starter-hello 并修改pom.xml如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>spring-boot-starter-hello</artifactId>
  <version>1.0-SNAPSHOT</version>

  <dependencies>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-autoconfigure</artifactId>
      <version>2.1.0.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-configuration-processor</artifactId>
      <version>2.1.0.RELEASE</version>
      <optional>true</optional>
    </dependency>
    
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

</project>
```

- 属性配置

```java
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "hello")
public class HelloServiceProperties {
  /*
  * 这里的配置与 类型安全的属性获取。在application.properties 中通过hello.msg 来设置。
  * 若不设置 默认hello.msg = world 下面的final 代码。
  * */
  private static final String MSG="world";
  private String msg=MSG;

  public String getMsg() {
    return msg;
  }

  public void setMsg(String msg) {
    this.msg = msg;
  }
}
```

- 判断依据类

```java
public class HelloService {
  // 根据此类的存在 创建这个类的 Bean
  private String msg;
  public String sayHello(){
    return "hello"+msg;
  }

  public String getMsg() {
    return msg;
  }

  public void setMsg(String msg) {
    this.msg = msg;
  }
}
```

- 自动配置类

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


/*
* 根据HelloServiceProperties提供的参数，并通过@ConditionalOnClass 判断 HelloService 
* 这个类是否在类路径中，matchIfMissing  : 当容器中没有这个Bean时，自动配置Bean
* */
@Configuration
@EnableConfigurationProperties(HelloServiceProperties.class) 
@ConditionalOnClass(HelloService.class)
@ConditionalOnProperty(prefix = "hello",value = "enabled",matchIfMissing = true)
public class HelloServiceAutoConfiguration {
  @Autowired
  private HelloServiceProperties helloServiceProperties;
  
  @Bean
  @ConditionalOnMissingBean(HelloService.class)
  public HelloService helloService(){
    HelloService helloService=new HelloService();
    helloService.setMsg(helloServiceProperties.getMsg());
    return helloService;
  }
}
```

- 注册配置。若想要自动配置生效，需要注册自动配置类。在src/main/resources 下新建META-INF/spring.factories 。里面内容如下，若有多个配置用逗号隔开。项目中如果没有resources目录，不能直接新建，要通过 File -> new -> Source Folder 。在Folder Name 那栏填上 src/main/resources 。

```factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=HelloServiceAutoConfiguration
```

​	至此，就搞定了所有程序。打包成jar 包发到Maven私服上，或者通过 **mvn install** 安装到本地库。

​	新建一个 spring boot 程序。并引入我们安装到本地库中的 spring-boot-starter-hello 即可

```xml
<dependency>
	<groupId>com.example</groupId>
    <artifactId>spring-boot-starter-hello</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

​	

