<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [第四章 Spring MVC](#%E7%AC%AC%E5%9B%9B%E7%AB%A0-spring-mvc)
  - [<a name="gaishu">Spring MVC概述</a>](#a-namegaishuspring-mvc%E6%A6%82%E8%BF%B0a)
  - [<a name="xiangmu">Spring MVC项目快速搭建—Tomcat</a>](#a-namexiangmuspring-mvc%E9%A1%B9%E7%9B%AE%E5%BF%AB%E9%80%9F%E6%90%AD%E5%BB%BAtomcata)
    - [Tomcat结构目录：](#tomcat%E7%BB%93%E6%9E%84%E7%9B%AE%E5%BD%95)
    - [有两种方法来配置虚拟目录：](#%E6%9C%89%E4%B8%A4%E7%A7%8D%E6%96%B9%E6%B3%95%E6%9D%A5%E9%85%8D%E7%BD%AE%E8%99%9A%E6%8B%9F%E7%9B%AE%E5%BD%95)
    - [配置虚拟主机：](#%E9%85%8D%E7%BD%AE%E8%99%9A%E6%8B%9F%E4%B8%BB%E6%9C%BA)
    - [使用内置Tomcat步骤：](#%E4%BD%BF%E7%94%A8%E5%86%85%E7%BD%AEtomcat%E6%AD%A5%E9%AA%A4)
    - [使用外置Tomcat步骤：](#%E4%BD%BF%E7%94%A8%E5%A4%96%E7%BD%AEtomcat%E6%AD%A5%E9%AA%A4)
    - [内外Tomcat都可以：](#%E5%86%85%E5%A4%96tomcat%E9%83%BD%E5%8F%AF%E4%BB%A5)
  - [<a name="changyong">Spring MVC常用注解</a>](#a-namechangyongspring-mvc%E5%B8%B8%E7%94%A8%E6%B3%A8%E8%A7%A3a)
  - [<a name="jiben">Spring MVC基本配置</a>](#a-namejibenspring-mvc%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AEa)
    - [4.4.1 **静态资源映射**](#441-%E9%9D%99%E6%80%81%E8%B5%84%E6%BA%90%E6%98%A0%E5%B0%84)
    - [4.4.2 拦截器配置](#442-%E6%8B%A6%E6%88%AA%E5%99%A8%E9%85%8D%E7%BD%AE)
    - [4.4.3 @ControllerAdvice](#443-controlleradvice)
    - [4.4.4 其他配置](#444-%E5%85%B6%E4%BB%96%E9%85%8D%E7%BD%AE)
  - [<a name="gaoji">Spring MVC高级配置</a>](#a-namegaojispring-mvc%E9%AB%98%E7%BA%A7%E9%85%8D%E7%BD%AEa)
    - [4.5.1 文件上传配置](#451-%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E9%85%8D%E7%BD%AE)
    - [4.5.2 自定义HttpMessageConverter](#452-%E8%87%AA%E5%AE%9A%E4%B9%89httpmessageconverter)
    - [4.5.3 服务器推送技术](#453-%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%8E%A8%E9%80%81%E6%8A%80%E6%9C%AF)
  - [<a name="ceshi">Spring MVC测试</a>](#a-nameceshispring-mvc%E6%B5%8B%E8%AF%95a)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->



《springboot 实战》汪云飞编著*

### 第四章 Spring MVC

---

#### <a name="gaishu">Spring MVC概述</a>

​	说道Spring MVC ，就得先知道 MVC，它和三层架构的关系是什么：很多人都知道这个模型

​	MVC ： Model + View + Controller (数据模型+视图+控制器)

​	三层架构： Presentation tier + Application tier + Data tier (展现层+应用层+数据访问层)

​	看上去 ***Model => Data tier  ,  View => Presentation tier ,  Controller => Application tier*** 。实际上不是这样的。**MVC 只存在三层架构的展现层中**。**M 实际上是数据模型，包含数据的对象。**在 Spring MVC 里有一个专门的类叫 Model ，用来和 V 之间的数据交互，传值；**V 指的是视图页面，**包含JSP，freeMarker，Velocity，Thymeleaf，Tile等 ；**C 当然就是控制器(Spring MVC 中 @Controller 注解的类)** 

​	而三层架构是整个应用的架构，是由Spring 框架负责管理的。一般项目结构中都有 Service 层，DAO层。这两个反馈在应用层和数据访问层。*查了一些文章，很大一部分人认为三层架构是宏观解决方案，MVC是微观解决方案。*

​	还要问一个和重要的问题：为什么需要MVC，三层架构？(以下回答来自知乎：https://www.zhihu.com/question/21851341 的 伍一峰 回答 miNi天中)

>简单来说是为了更好的分开逻辑，从而让代码模块化。模块化的好处是，1.一个模块的修改对其他模块影响不大，2.拓展应用更容易 ，3.不同模块开发可以同时进行

下面一段参考自：[MVC与三层架构](https://juejin.im/post/5929259b44d90400642194f3)

​	MVC架构程序的工作流程是这样的：

>(1) 用户通过View 页面向服务端发出请求，可以是表单请求，超链接请求，AJAX请求等
>
>(2) 服务端Controller 控制器接收到请求后对请求进行解析，找到相应的Model对用户请求进行处理
>
>(3) Model处理后，将处理结果再交给Controller
>
>(4) Controller在接收到处理结果后，根据处理结果找到要作为向客户端发回相应View页面。页面经渲染(数据填充)后，再发送给客户端

![12.png](https://github.com/MrAlan/MyPostPicture/blob/master/12.png?raw=true)





---

#### <a name="xiangmu">Spring MVC项目快速搭建—Tomcat</a>

​	Spring MVC提供了一个***DispatcherServlet***来开发Web应用。在Servlet2.5及以下的时候只要在web.xml 下配置`<servlet>`元素即可。但是这里后面的版本会用到无web.xml的方式，在Spring MVC里实现***WebApplicationInitializer***接口可实现等价于web.xml的配置。

​	这次的demo需要用到Tomcat，这里是链接https://tomcat.apache.org/download-80.cgi ，我安装的版本是8.5的所以就讲我的经历了。选择 Binary Distributions  下的 Core 下面的 32-bit/64-bit Windows Service Installer 安装程序，这样比较简单。(注意jdk的版本要与Tomcat的版本相符)。安装成功之后运行，在浏览器访问 127.0.0.1:8080，结果如下图

![13.png](https://github.com/MrAlan/MyPostPicture/blob/master/13.png?raw=true)

下面一部分内容参考自：https://juejin.im/post/5a75b0be5188254e761781d7

##### Tomcat结构目录：

	>1. bin : 启动和关闭 tomcat的 脚本文件 (bat)
	>2. conf ：配置文件
	>   1. server.xml 该文件用于配置server相关的信息，比如tomcat启动的端口号，配置主机(host)
	>   2. web.xml 文件配置与web应用 (web应用相当于一个web站点)
	>   3. tomcat-user.xml 配置用户名密码和相关权限   (还有其他文件就不多说了)
	>3. lib : 该目录放置运行 tomcat 所需要的 jar包
	>4. logs ：存放日志，当我们需要查看日志的时候，可以查询信息
	>5. webapps：放置我们的web应用
	>6. work ： 该目录用于存放**jsp被访问后生成对应的server文件和 .class 文件**

​	webapps目录的详细说明：(毕竟后面我们的站点是放在这里的：这个目录就是上面说的127.0.0.1:8080)

​	在这个目录中新建一个webtest文件夹，里面再新建一个test.html。访问127.0.0.1:8080/webtest/test.html ，就可以看到test.html里的内容了。

![14.png](https://github.com/MrAlan/MyPostPicture/blob/master/14.png?raw=true)

​	

​	web站点的目录是由规范的，这是由实践工作中总结出来的。webapps目录的结构图如下：

​	![15.png](https://github.com/MrAlan/MyPostPicture/blob/master/15.png?raw=true)

​	***值得注意的是 8.5版本的Tomcat 里 bbs目录换成了 ROOT目录，还有图中的web应用目录是与bbs平级的就好像我们刚才创建的webtest目录。***

​	现有多个html文件，要将其中一个设置为首页，如果没有WEB-INF目录下的web.xml文件时无法实现的。这个是Tomcat定义的的运行规则。

​	所以我们在刚才创建的webtest目录下创建WEB-INF目录，再在里面创建web.xml文件。不知道web.xml怎么写没关系,将 ROOT/WEB-INF/web.xml 复制一下，再加入如下代码。

```xml
<welcome-file-list>
	<welcome-file>test.html</welcome-file>
</welcome-file-list>
```

​	像下面这样：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
  version="3.1"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to Tomcat
  </description>
  
  <welcome-file-list>
	<welcome-file>test.html</welcome-file>
  </welcome-file-list>
  
</web-app>

```

![16.png](https://github.com/MrAlan/MyPostPicture/blob/master/16.png?raw=true)



​	一般情况下 ，上述部分的内容已经够用了。但是如果项目太大，可能磁盘空间不足，把web站点的目录分散到其他磁盘管理就需要配置 **虚拟目录** *默认情况下，只有webapps目录才会被Tomcat管理成一个web站点* 。 把web应用所在目录交给web服务器管理，这个过程称之为虚拟目录的映射。

##### 有两种方法来配置虚拟目录：

方法一：

  1. 在其他盘符中创建一个web站点目录，并在里面创建WEB-INF目录和html文件(就像上文中的webtest 目录，但是是在其他盘中，如D盘，G盘)

  2. 找到Tomcat目录下的 conf/server.xml 文件，添加代码后：

     ```xml
     <Host name="localhost"  appBase="webapps"
                 unpackWARs="true" autoDeploy="true">
         <!-- 下面一行就是添加的代码-->
             <Context path="/webtest1" docBase="D:\webtest1"/>
         	
             <!-- SingleSignOn valve, share authentication between web applications
                  Documentation at: /docs/config/valve.html -->
             <!--
             <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
             -->
     
             <!-- Access log processes all example.
                  Documentation at: /docs/config/valve.html
                  Note: The pattern used is equivalent to using pattern="common" -->
             <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                    prefix="localhost_access_log" suffix=".txt"
                    pattern="%h %l %u %t &quot;%r&quot; %s %b" />
     
           </Host>
     ```

     ​	path 表示的是访问时输入的web项目名，docBase表示的是站点的绝对路径。配置完之后(记得重启Tomcat)

方法二：

​	进入到conf\Catalina\localhost 目录下，创建一个xml文件，如 webtest2.xml（文件名就是站点名）,内容如下

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<Context 
    docBase="E:\webtest2" 
    reloadable="true"> 
</Context> 
```

​	该方法不需要重启Tomcat。



​	如果你嫌输入ip地址很麻烦，也可以修改 C:\Windows\System32\drivers\etc\hosts 文件，配置临时域名(请注意，现在你的web应用还不能被外网访问)

```
127.0.0.1   localhost
127.0.0.1   lin
```



##### 配置虚拟主机：

​	虚拟主机是在主机不够的情况下设置的，一个Tomcat服务器只能运行一个网站，我们如果需要3个域名，就要三台电脑。而我没有三台电脑。

​	1. 在tomcat的server.xml文件添加主机名

```xml
 <Host name="lin" appBase="E:\webtest1">
		<Context path="/webtest1" docBase="E:\webtest1"/>
 </Host>
```

![17.png](https://github.com/MrAlan/MyPostPicture/blob/master/17.png?raw=true)



最后附上 Tomcat 的体系结构图 以及 浏览器访问web资源的流程图

![18.png](https://github.com/MrAlan/MyPostPicture/blob/master/18.png?raw=true)

![19.png](https://github.com/MrAlan/MyPostPicture/blob/master/19.png?raw=true)

---

​	搞定Tomcat后，就可以开始做下一步的演示了。即将Tomcat和IDEA结合起来使用，要说的是Spring boot 内嵌了Tomcat，本来我们是可以不用下载Tomcat的，但是学习嘛，不就是一个个零件慢慢拼凑成最终的模样的吗？而且IDEA也可以配置外置的Tomcat。

​	下面一部分来自：[Spring Boot -Tomcat 容器部署](https://www.jianshu.com/p/a5195a08da3e)

​	一般情况是吧项目打包成war包，然后丢到Tomcat的webapps目录下，启动服务的。SpringBoot会更简单些。

​	环境：IDEA ，Maven3 ，JDK1.8 ，Tomcat8 

##### 使用内置Tomcat步骤：

- 打开IDEA-> new Project -> Spring Initializr -> 填完Group，Article -> 选择Web ->勾选Web->next ->Finish。创建完毕后，会有一个java文件如下：

```java
package com.example.demo;

import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication        //表示这个是应用入口，容器
public class DemoApplication {

    public static void main(String []args){
        SpringApplication.run(DemoApplication.class,args);
    }
}

```

不需要配置Tomcat ，因为在引入SpringBoot的时候，就已经包含了下面Tomcat的依赖。

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
</dependency> 
<!--引入这个依赖后，下面那个就已经自动引入了	-->
```

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <version>1.5.8.RELEASE</version>
    <scope>provided</scope>
</dependency>
```

- 修改tomcat的配置，如port，在main/resources/application.properties中配置

```properties
## 容器端口号，默认8080
server.port=8280
## 上下文路径，默认/    
server.servlet.context-path=/demo
## tomcat编码，默认UTF-8
server.tomcat.uri-encoding=UTF-8
```

- 新建一个 DemoController

```java
@RestController
@RequestMapping("/index")  // 访问的文件名，index
public class DemoController {

    @GetMapping
    public List<Map<String, String>> get(){
        List<Map<String, String>> demos = new ArrayList<>();
        Map<String, String > demo1 = new HashMap<>();
        demo1.put("roach", "小强");
        demo1.put("crow", "乌鸦");
        demos.add(demo1);

        Map<String, String> demo2 = new HashMap<>();
        demo2.put("lion", "狮子");
        demo2.put("tiger", "老虎");
        demos.add(demo2);

        return demos;
    }
}

```

- 然后运行 项目，在浏览器输入 http://localhost:8280/demo/index  就有结果了。

##### 使用外置Tomcat步骤：

 1. 将pom.xml 中的 `<packing>jar<packing>` 改为`<packing>war<packing>`，即修改打包方式以及添加配置如下，

     ```xml
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-tomcat</artifactId>
       <scope>provided</scope> <!--这行可以不要-->
     </dependency>
     ```

     ```xml
     <plugin>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-maven-plugin</artifactId>
       <version>2.1.0.RELEASE</version>
       <configuration>
         <mainClass>SpringmvcApplication</mainClass>
         <layout>ZIP</layout>
       </configuration>
       <executions>
         <execution>
           <goals>
             <goal>repackage</goal>
           </goals>
         </execution>
       </executions>
     </plugin>
     ```

 2. DemoApplication 类 (注解了 @SpringBootApplication 的类) 要继承
    `org.springframework.boot.web.support.SpringBootServletInitializer`,并重写configure方法

    ```java
    @SpringBootApplication
    public class DemoApplication extends SpringBootServletInitializer{
    
        @Override
        protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
            return builder.sources(this.getClass());
        }
    }
    ```

3. 在项目的根目录运行 mvn clean package , 即可打包成功。

4. 打开IDEA的 **Edit configuration**，点击左上角的加号,找到**Tomcat Server 的local**，再输入配置名，以及在 **Application server那栏点击 Configure  ，选择你安装Tomcat的目录**。 还可以输入自己想要的端口 ***(ps : application.properties对外置Tomcat容器无用)***

    ![20.png](https://github.com/MrAlan/MyPostPicture/blob/master/20.png?raw=true)

 5. 再点击图中Deployment，点击右边的加号，选择Artifact，那里有刚才生成的war包，在Application Context那里记得输入 项目的启动地址（记得加"/"）,也可以不输入名字，这样就是端口访问如 localhost:8081/  下面加了名字的是  localhost:8081/springmvc2

    ![21.png](https://github.com/MrAlan/MyPostPicture/blob/master/21.png?raw=true)


 6. 点击应用之后，运行，然后在浏览器上输入 localhost:8081/demo/index




##### 内外Tomcat都可以：

​	这时候就需要加入那个被自动引入的包。这样可以**覆盖**被*spring-boot-starter-web*自动引入的tomcat。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <version>1.5.8.RELEASE</version>
    <scope>provided</scope>
</dependency>
```

​	用内置的Tomcat时:

```java
@SpringBootApplication
public class DemoApplication{

  public static void main(String[]args){
    SpringApplication.run(DemoApplication.class,args);
  }

}
```

​	外置:

```java
@SpringBootApplication
public class DemoApplication extends SpringBootServletInitializer {

  @Override
  protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
    return builder.sources(this.getClass());
  }
}
```

​	基于Maven搭建零配置的Spring MVC 原型项目就不做了，比较麻烦，直接用上述的方法 (IDEA Spring Initializr)构建的Spring mvc 项目，这样更快。

---

#### <a name="changyong">Spring MVC常用注解</a>

  1. @Controller :  被注解的类是 Spring MVC 里的 Controller ，将其声明为Spring 里的一个 Bean。 **Dispatcher Servlet 会自动扫描注解了此注解的类。并将Web请求映射到注解了 @RequestMapping 的方法上**。就像上文中DemoController中的方法一样。在申明普通Bean的时候，使用 @Component 、@Service、@Repository、@Controller 是同等的，因为它们都组合了(@Component) 。但在 Spring MVC 声明控制器Bean的时候，只能使用@Controller。

  2. @RequestMapping ： 是用来**映射 Web 请求（访问路径和参数）、处理类和方法的**。 @RequestMapping 可注解在类或方法上。**注解在方法上的 @RequestMapping  路径会继承注解在类上的路径**， @RequestMapping  支持Servlet 的 request 和response 作为参数，也支持对request 和response的媒体类型进行配置。在Spring 4.3中引入了@RequestMapping在方法层级的变种，来帮助简化常用Http方法的映射，比如@GetMapping  可以读作 GET @RequestMapping

        	1. @GetMapping 
        	2. @PostMapping 
        	3. @PutMapping
        	4. @DeleteMapping
        	5. @PatchMapping

  3. @ResponseBody ：支持将返回值放在 response 体内，而不是返回一个页面，我们在很多基于Ajax 的程序时，可以以此注解返回数据而不是页面。**此注解可放置在 返回值前或方法上**

  4. @RequestBody ： 允许request参数在request 体中，而不是在直接链接地址后面。**此注解放在 参数前**

  5. @PathVariable ： 用来接收路径参数，如  */news/001* 可接收 001 作为参数。**此注解放在参数前**

  6. @RestController ： 是一个组合注解，组合了@Controller 和 @ResponseBody  ，这就意味着当你只需要开发一个和页面交互数据的控制的时候，需要用这个注解。要返回一个页面(如html)时只能用@Controller 注解，因为@ResponseBody 只能是 返回值


代码 发在了 github的另一个库里：https://github.com/MrAlan/SpringBoot/tree/master/demo 

---

#### <a name="jiben">Spring MVC基本配置</a>

​	通常情况下，Spring Boot的自动配置是符合我们的大多数需求的。在你既要保留Spring Boot提供的便利，又要增加自己的额外的配置的时候，可以定义一个配置类并`implements WebMvcConfigure` 

##### 4.4.1 **静态资源映射**

​	SpringBoot 的静态资源(图片，音乐，css,js,静态html...等文件)一般按照它的规则来说可以自由的访问，按先后顺序扫描：**/META-INF/resources/**，**src/main/resources/resources/** ，**src/main/resources/static/** ，**src/main/resources/public/**,   这四个文件中有没有资源。这几个文件夹我们当项目里没有时我们可以自己创建，而且连这四个文件夹的子文件夹都不行。访问时别忘记了加文件格式后缀，如 *.html  ， .png*

​	如果要自己决定在哪个文件夹里扫描，可以在application.properties里配置：`spring.mvc.static-path-pattern=/resources/**`  在 resources/resources  静态资源必须放在该文件夹下要使用网络上的一些资源，比如jquery.js , 要添加  `webjars-locator-core` 依赖(无版本要求)，然后在需要引入资源时 用这个语句就可以了 `"webjars/jquery/jquery.min.js"` 

项目结构图如下：

​	![28.png](https://github.com/MrAlan/MyPostPicture/blob/master/28.png?raw=true)

​	![27.png](https://github.com/MrAlan/MyPostPicture/blob/master/27.png?raw=true)

​	需要**动态页面**时，就要用到**模板引擎**：一般SpringBoot自动配置4种 模板引擎：选择哪种引擎就要看项目更适合哪个了

​	1. FreeMarker    ： 被设计来生成HTMl Web页面，特别是基于MVC模式的应用程序。基于模板生成文本输出的通用工具，使用纯java编写。

​	2. Groovy 

​	3. Thymeleaf     官方推荐：静态html嵌入标签属性，浏览器可以直接打开模板文件，便于前后端联调。但是缺点是：模板必须符合***xml***规范，js脚本必须加入/*<![CDATA[*/标识

​	4. Mustache

​	并不推荐使用JSP，使用引擎时，默认的路径是   `src/main/resources/templates` 。但是模板引擎时常会找不到这个路径，官方给出的答案是：(下面是我理解的版本，[官方文档请到这里](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-spring-mvc-template-engines))

	>因为我们运行程序的方式不同，IntelliJ IDEA 会以不同的方式命名classpath。而且使用Maven或者Gradle 以及它们的打包方式(war  ?  jar  ?)都会对其造成影响，这可能导致Spring Boot无法在类路径中找到模板。

​	官方给出的答案是这样的：

> 1. 可以在IDE中重新排序类路径，以便首先放置模块的类和资源
>
> 2. 可以配置模板前缀用来搜索classpath下 的templates下的每一个目录，如下所示：
>
>    `classpath *：/ templates /`

​	

​	以Thymeleaf为例：讲下配置

1. 引入依赖 , spring-boot-starter-thymeleaf 

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-thymeleaf</artifactId>
   </dependency>
   ```


2. 创建Controller 类
   ```java
   package com.shanshan.start.controller;
   
   import org.springframework.stereotype.Controller;
   import org.springframework.ui.ModelMap;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.ResponseBody;
   
   @Controller
   public class HelloController {
   
       @ResponseBody
       @RequestMapping("/hello")
       public String hello() {
           return "Hello World";
       }
       
       @RequestMapping("/w")
       public String index() {
           Map<String,String> map=new ArrayList<>();
           map.put("hello","world");
           return "world"; 
       }  //这个方法就好像和  world.html连接在了一起，，里面的对象也可以在world.html 里面用
   }
   ```

3. 在templates 目录下加入需要的world.html，与Controller的方法返回值要同名。这里的语法很严格，错了一点都不能正常显示。

   ```html
   <!DOCTYPE html>
   <html xmlns:th="http://www.thymeleaf.org"> <!--这里是链接模板引擎的-->
   <head lang="en">
       <meta charset="UTF-8" />
       <title></title>
   </head>
   <body>
   <h1 th:text="${hello}">World</h1> <!--使用-->
   </body>
   </html>
   ```

   ​	上面通过xmlns：th="http://www.thymeleaf.org"命令控件，将静态页面转换为动态的视图，只要将需要进行动态处理的元素使用***th***前缀即可

4. 在上面三步都完成了之后，基本上就大功告成了，但是有些时候maven引进来的包也可能出错，这样会导致访问出错，把配置对应的maven 2的目录清空后重新构建，再下载所需要的jar包就可。

  ​	对于Controller中的方法也可以定义一个返回 **ModelAndView** 对象，它包含了一个**model**属性和一个**view**属性，是Map的子类。当视图解析器解析它的时候，其中model本身就是Map的实现类的子类，视图解析器将model中的每个元素都通过request.setAttribute(name,value);添加request请求域中，这样就可以在html,jsp等页面中通过EL表达式来获取对应的值。

  ```java
   @RequestMapping("/w")
   public ModelAndView index() {
       List<User> list=new ArrayList<>();
       list.add(new User(1,"lin"));
       list.add(new User(2,"jia"));
       list.add(new User(3,"bao"));
       
       ModelAndView modelAndview=new ModelAndView("world"); //添加的是view名，即动态页面的文件名
       modelAndview.addObject("userlist",list);
       return modelAndview; 
   } 
  ```

  然后根据上面的内容写 html

  ```html
  <!DOCTYPE html>
  <html xmlns:th="http://www.thymeleaf.org"> <!--这里是链接模板引擎的-->
  <head lang="en">
      <meta charset="UTF-8" />
      <title></title>
  </head>
  <body>
  	<tr th:each="user : ${userList}">
          <td th:text="${user.getId()}">用户ID</td>
          <td th:text="${user.getName()}">用户名</td>
      </tr>
  </body>
  </html>
  ```



##### 4.4.2 拦截器配置

​	拦截器 **Interceptor** 实现对**每一个请求处理后进行相关的业务处理**，类似于Servlet的Filter 。可让普通的Bean实现 HandlerInterceptor 接口来实现自定义拦截器。实现之前要让另一个类实现WebMvcConfigurer的addInterceptor()方法  来注册拦截器。  下面写一个简单的得知处理时间的拦截器。

```java
package com.example.springmvc;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

@Component
public class DemoInterceptor implements HandlerInterceptor {

  @Override
  public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
      throws Exception {
    long startTime=System.currentTimeMillis();
    request.setAttribute("startTime",startTime);
    return  true;
  }

  @Override
  public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
      ModelAndView modelAndView) throws Exception {
    long startTime=(Long) request.getAttribute("startTime");
    request.removeAttribute("startTime");
    long endTime=System.currentTimeMillis();
    System.out.println("本次处理请求的时间为"+(endTime-startTime)+"ms");
    request.setAttribute("HandlingTime",endTime-startTime);
  }
}

```

```java
package com.example.springmvc;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

  @Bean   //定义拦截器的 Bean
  public DemoInterceptor demoInterceptor(){
    return new DemoInterceptor();
  }

  @Override
  public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(demoInterceptor());  //利用 Bean 注册拦截器，
  }
}
```



##### 4.4.3 @ControllerAdvice

​	通过@ControllerAdvice ，我们可以**将对于控制器的全局配置放置在同一个位置**(全局配置指**整个程序**，同一位置指的是通过这个注解，**所有的可以配置全局的注解都可以放在同一个*类*中统一管理**)，比如下面三个：

 	1. @ExceptionHandler ： 用于全局处理控制器的异常
 	2. @InitBinder ： 用来设置WebDataBinder ，WebDataBinder 用来自动绑定前台的请求参数到Model中
 	3. @ModelAttribute ： 本来的作用是绑定键值对到Model里，此处是让全局的@RequestMapping 都能获得在此处设置的键值对。

```java
package com.example.springmvc;

import org.springframework.ui.Model;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.servlet.ModelAndView;

@ControllerAdvice  // 声明一个控制器建言，@ControllerAdvice 组合了 @Component 注解。所以自动注册为Spring的Bean
public class DemoControllerAdvice {

  @ExceptionHandler(value = Exception.class) //定义全局处理，value属性是过滤拦截的条件，在这里，很明显要拦截程序中所有的exception，并不只下面的方法
  public ModelAndView exception(Exception exception, WebRequest request){
    ModelAndView modelAndView=new ModelAndView("error");//定义要转到的 view 名
    modelAndView.addObject("errorMessage",exception.getMessage());
    return modelAndView;
  }//每当程序中 发出了异常，，这个方法就会截获，从而跳转到  error.html

  @ModelAttribute  //将键值对添加到全局，所有注解了 @RequestMapping 的方法都可以获得此 键值对
  public void addAttributes(Model model){
    model.addAttribute("msg","额外报错信息");
  }

  @InitBinder  //定制 WebDataBinder
  public void initBinder(WebDataBinder webDataBinder){
    webDataBinder.setDisallowedFields("id");
  }

}
```

```java
package com.example.springmvc;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class AdviceController {
  @RequestMapping("/advice")
  public String getSomething(@ModelAttribute("msg")String msg,DemoObj demoObj){
    // 方法参数中的 msg来源于 DemoControllerAdvice 里的 addAttributes 方法，它被@ModelAttribute注解了。而且本身也注解了@RequestMapping
    throw new IllegalArgumentException("非常抱歉，参数有误/"+"来自@ModelAttribute:"+msg);
    //发出异常后，，被 DemoControllerAdvice.exception方法拦截，并转入error.html页面，，因为它注解了 @ExceptionHandler
    //拦截 程序中所有的 异常
  }
  //加了 demoobj  它的请求路径就变成了，/advice?id=3&name="lin"
}
```



##### 4.4.4 其他配置

​	像配置页面转向简单来写会是下面这样， 将viewname.html 映射

```java
@RequestMapping("/pathname")
public String hello(){
    return "viewname";
}

```

​	没有业务逻辑，只是简单的页面转向就写了三行代码，在涉及大量页面跳转时如果都这么写，项目的结构会很乱。可以在配置中重写 WebMvcConfigurer.addViewControllers  简化 。但是 addViewController 方法 默认前缀地址是 templates/  ，这个要注意。

```java
public void addViewControllers(ViewControllerRegistry registry){
    registry.addViewController("/urlname").setViewName("/viewname"); //前后两个viewname最好一致  ，这里做个演示，就把路径改为了/urlname
    //如果资源在 static就改成这样             .setViewName(../static/viewname);
}

```

​	注意：如果资源在 template子目录下,要把 资源路径写清楚，比如在 ***templates/views/view.html***  就要写成  ***views.view***  , 即 `setViewName("view/views")`   

  

---

#### <a name="gaoji">Spring MVC高级配置</a>

##### 4.5.1 文件上传配置

​	文件上传是一个项目里经常要用的功能，Spring MVC 通过配置 MultipartResolver来上传文件。在Spring 的控制器中，通过MultipartFile file 来接收文件，MultipartFile [ ] Resolver 接收多个文件上传。如果文件过大就会报错，具体的内容可以google。

​	`Request method 'GET' not supported` 结果遇到了这个错误，估计原因是用了 viewController .addViewController("").setViewName()  这个和 post 方法的流程冲突了。应该自己写个返回页面的代码，如下：

```java
package com.example.springmvc.file;

import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;

@Controller
public class UploadController {

  @RequestMapping("/uploadFile")  // 静态页面访问
  public String upload(){
    return "uploadFile";
  }


  @RequestMapping(value = "/uploadFile",method = RequestMethod.POST)  //当页面出现post请求时，调用
  @ResponseBody
  public String upload(MultipartFile file){
 /*
 采用绝对路径，也可以是项目的相对路径如果是 "/temp/cp/"  写入路径 在   /tmp/tomcat.273391201583741210.8080/work/Tomcat/localhost/ROOT/tmp/cp/
 当我们那个路径下没有 temp/cp文件夹，就会发生异常
*/
      
    try{
      byte[] bytes = file.getBytes();
      Path path = Paths.get("E:/" + file.getOriginalFilename());
      Files.write(path, bytes);
      return "上传成功";
    }catch (Exception e){
      return "error : "+e.getMessage();
    }
  }
}
```

再写uploadFile.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
<body>
  <div class="upload">
      <!-- 这里的action 很重要，要填项目路径，运行中虽然会默认前缀是localhost:8080/projectName, 但是表单提交后返回的是 根据 action里的内容，，所以如果只填 /uploadFile  那么提交后返回的路径就是
localhost:8088/uploadFile  会报错，404
-->
    <form action="/uploadFile" enctype="multipart/form-data" method="post">
      <!--<input type="file"/><br/>      name=“file” 不能漏-->
       <input type="file" name="file"/>
      <input type="submit" value="上传文件"/>
    </form>
  </div>

</body>
</html>
```

​	到这又遇到了问题，MultipartFile 找不到 我们上传的文件，返回了error ,null 。  查了很多资料后发现是 上面的`<input type="file" >` ，name="file" 漏了，补上就可以了。

​	还有想要从HTML的<a>标签里访问我们项目中的其他html，如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>静态测试</title>
</head>
<body>
  this is a index html

  <a href=uploadFile>去传输文件界面</a>
  <a href=anno/demo>去anno/demo</a>
  <a href=anno>去anno</a>
<!--  这里用的 href是我们在浏览器上的的访问路径（除去默认的localhost:8088/ 这是我定义的项目路径），并不是按照项目中的文件结构来写这些资源，包括js，css 等其他资源。起码在内置Tomcat运行是这样的。-->
</body>
</html>
```

​	在这之后又遇到了问题，那就是如果在端口后加上项目名  如 localhost:8088/testwar 之后，上传就会发生错误，如下面两张图所示：

​	![](https://github.com/MrAlan/MyPostPicture/blob/master/29.png?raw=true)![](https://github.com/MrAlan/MyPostPicture/blob/master/30.png?raw=true)

​	提交之后返回的路径把中间的项目名给漏了。找了几篇文章觉得post之后需要重定向是比较接近答案的。终于在[这篇文章](https://juejin.im/post/5b139828e51d450685491e7c) 中看到一个表单的样本，里面的**action**和我写的很不一样，我是直接 **action="/uploadFile"**。查了html 里 form 的action 语法之后：知道是我action里的url填错了，多加了 个 ***/***

![](https://github.com/MrAlan/MyPostPicture/blob/master/31.png?raw=true)

```html
<form action="http://localhost:8088/testwar/uploadFile" enctype="multipart/form-data" method="post"> 或者
<form action="uploadFile" enctype="multipart/form-data" method="post"> 都可以，
但是 <form action="/uploadFile" enctype="multipart/form-data" method="post"> 不行。

```

##### 4.5.2 自定义HttpMessageConverter 

​	HttpMessageConverter 是用来处理 request 和 response 里的数据的。Spring 为我们内置了大量的 HttpMessageConverter 。本节将演示自定义的 HttpMessageConverter ，并注册这个HTTPMessageConverter 到Spring MVC中。

 1. 自定义 HTTPMessageConverter

    ```java
    package com.example.springmvc.ch45.message;
    
    import com.example.springmvc.DemoObj;
    import java.io.IOException;
    import java.nio.charset.Charset;
    import org.springframework.http.HttpInputMessage;
    import org.springframework.http.HttpOutputMessage;
    import org.springframework.http.MediaType;
    import org.springframework.http.converter.AbstractHttpMessageConverter;
    import org.springframework.http.converter.HttpMessageNotReadableException;
    import org.springframework.http.converter.HttpMessageNotWritableException;
    import org.springframework.util.StreamUtils;
    
    public class MyMessageConverter extends AbstractHttpMessageConverter<DemoObj> {
    
      public MyMessageConverter(){
        /*
        * 新建一个我们自定义的媒体类型，application/x-wisely  在 ConverterController 那里用到，
        * 和这里对应连接
        * */
        super(new MediaType("application","x-wisely", Charset.forName("UTF-8")));
      }
    
      /*
      * 表明这个 HTTPMessageConverter 只处理DemoObj 这个类
      * */
      @Override
      protected boolean supports(Class<?> aClass) {
        return DemoObj.class.isAssignableFrom(aClass);
      }
    
      /*
      * 重写 readInternal 方法 ，处理请求的数据，代码表明我们处理由 '-' 隔开的数据，并转成DemoObj的对象
      * 在这个例子中，拦截了 converter.html中 的 ajax 的post请求传过来的 数据，
      * */
      @Override
      protected DemoObj readInternal(Class<? extends DemoObj> aClass, HttpInputMessage httpInputMessage)
          throws IOException, HttpMessageNotReadableException {
        String temp= StreamUtils.copyToString(httpInputMessage.getBody(),Charset.forName("UTF-8"));
        String []tempArr=temp.split("-");
        return new DemoObj(new Long(tempArr[0]),tempArr[1]);
      }
    
      /*
      * 重写 writeInternal 处理如何输出数据到response ，此例中，我们在原样输出在前面加上 "hello"
      * 在这个例子中  拦截了 ConverterController.convert()方法 返回到客户端的 response  ，进行了处理
      * */
      @Override
      protected void writeInternal(DemoObj demoObj, HttpOutputMessage httpOutputMessage)
          throws IOException, HttpMessageNotWritableException {
          String out ="hello"+demoObj.getId()+"-"+demoObj.getName();
          httpOutputMessage.getBody().write(out.getBytes());
      }
    }
    
    ```

    2.在 MyMvcConfig 里的 addViewController映射访问页面

    ```java
    registry.addViewController("/converter").setViewName("converter");
    // converter.html 就放在templates目录下，所以路径是 converter
    ```

    3.配置 HTTPMessageConverter 的Bean，有两种方法：（MyMvcConfig 实现了 WebMvcConfigure）

    4.在 MyMvcConfig 重写configureMessageConverters ： 会覆盖掉 Spring MVC 默认注册的多个 HTTPMessageConverter

    5. 在 MyMvcConfig 里面重写 extendMessageConverts: 仅添加一个自定义的 HttpMessageConverter ，不会覆盖默认的 HTTPMessageConverter

    ```java
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
      converters.add(converter());
    }
    
    @Bean
    public MyMessageConverter converter(){
      return new MyMessageConverter();
    }
    ```

    4. 演示用的控制器

    ```java
    package com.example.springmvc.ch45.message;
    
    import com.example.springmvc.DemoObj;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.ResponseBody;
    
    @Controller
    public class ConverterController {
      /*
      * 这里接受 converter.html里的 ajax的post请求，数据在传过来之前，
      * 被MyMessageConverter.readInternal 方法处理成了DemoObj对象，
      * 传到下面被注解的 demoObj中
      *
      * 这里返回的 response 内容则会被拦截MyMessageConverter 里的 writerInternal 处理
      * */
    
      @RequestMapping(value = "/convert",produces = {"application/x-wisely"})
      public @ResponseBody DemoObj convert(@RequestBody DemoObj demoObj){
        return demoObj;
      }
    }
    
    ```

    5. 写 converter.html

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <title>HttpMessageConverter Demo</title>
    </head>
    <body>
      <div id="resp"></div>
      <input type="button" onclick="req();" value="请求"/>
      <script src="jquery-1.11.3.min.js" type="text/javascript"></script>
        <!--根据浏览器访问的路径 写 src的路径，实际上我把jquery-1.11.3.min.js 放在了 resources/resources 目录下, -->
      <script>
      function req() {
          /*
         通过这个 url 访问 我们写的控制器/convert ，这个请求被 MyMessageConverter拦截，用readInternal 处理发送的data 将其转换为DemoObj对象，传到ConverterController的convert方法中，那里有个用@RequestBody 注解的DemoObj参数。
         之后 convert 再返回一个 demoObj对象，由 MyMessageConverter.writerInternal 方法处理成
         "hello" + obj.getId()+"-"+obj.getName(); 成功返回之后 调用下面success 的 function ，将返回的数据传入  id为resp 的div 中。完成整个流程。
         */
        $.ajax({
          url:"convert", 
          data:"1-wangyunfei",  
          type:"POST",
          contentType:"application/x-wisely",
          success:function (data) {
            $("#resp").html(data);
          }
        });
      }
      </script>
    </body>
    </html
    ```

    运行后访问 localhost:8088/converter



##### 4.5.3 服务器推送技术

​	服务端推送技术在我们日常开发中较为常用，可能**早期很多人的解决方案是用Ajax向服务器轮询消息**，使浏览器尽可能第一时间获得服务端的消息，因为这种方式的轮询频率不好控制，所以大大增加了服务端的压力。

​	本节多有的服务端推送方案都是基于：当客户端向服务端发送请求，服务端会抓住这个请求不放，等有数据更新的时候才返回给客户端，当客户端收到消息后，再向服务端发送请求，周而复始。这种方式的好处是减少了服务端的请求数量，大大减轻了服务器的压力。

​	除了服务端推送技术外，还有一个双向通信技术——WebSocket。在本书后面部分演示

​	本节将提供基于SSE(Server Send Event 服务端发送事件)的服务器推送和基于Servlet3.0+的异步方法特性，其中第一种方式需要新式浏览器的支持，第二种可以跨浏览器。

一.SSE	

1.控制演示器

```java
package com.example.springmvc.sse;

import java.util.Random;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class SseController {


  /*
  *  媒体类型为 text/event-stream  这是服务端对SSE的支持，
  *  本例中每2秒钟向浏览器推送随机消息,下面value=push ，是访问路径，通过sse.html 里的js中
  *  访问，通过SSE的客户端访问这里 ， 使其可以不断的发送信息到客户端，
  *  如果 输入的网址是 /push 只返回一个 信息  ，并不能不断的返回。
  * */

  @RequestMapping(value = "/push",produces = "text/event-stream")
  public @ResponseBody String push(){
    Random r=new Random();
    try{
      Thread.sleep(2000);
    }catch (Exception e){
      e.printStackTrace();
    }
    return "data: Testing 1,2,3"+r.nextInt()+"\n\n";
  }

}
```

2.演示界面，新建sse.html 

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>SSE demo</title>
</head>
<body>
  <div id="msgFromPush"></div>
  <script type="text/javascript" src=jquery-1.11.3.min.js></script>
  <script>
    /*
    * EventSource 对象是新式浏览器(Chrome ,firefox )等才有的，EventSource是SSE的客户端。
    * 下面添加了SSE客户端监听，在此获得 服务端推送的消息
    * */
    if(!!window.EventSource){
      var source=new EventSource('push'); //连接 服务端的 push 方法，使得 服务端可以不断向客户端发送信息
      s='';
      source.addEventListener('message',function (evt) {
        s+=evt.data+"<br/>";
        $("#msgFromPush").html(s); // 将服务端推送的消息内容 写进 div中，使页面内容发生变化
      });

      source.addEventListener('open',function (evt) {
        console.log("连接打开");
      });

      source.addEventListener('error',function (e) {
        if (e.readyState==EventSource.CLOSED){
          console.log("连接关闭");
        }else {
          console.log(e.readyState);
        }
      })
    }else {
      console.log("你的浏览器不支持 sse ");
    }
  </script>
</body>
</html>
```

​	在浏览器输入  localhost:8088/sse 就可



二. Servlet3.0+异步方法处理

1.写好定时任务作为发送的服务

```java
package com.example.springmvc.sse;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;
import org.springframework.web.context.request.async.DeferredResult;

@Service
public class PushService {

  /*
   * Servlet 3中的异步支持为 在另一个线程中处理HTTP请求提供了可能性。
   * 当有一个长时间运行的任务时，这是特别有趣的，
   * 因为当另一个线程处理这个请求时，容器线程被释放，并且可以继续为其他请求服务。
   * spring 提供 DeferredResult 来实现
   * */

  /*
   * DeferredResult的超时处理，采用委托机制，也就是在实例DeferredResult时给予一个超时时长（毫秒），
   * 同时在onTimeout中委托（传入）一个新的处理线程（我们可以认为是超时线程）；
   * 当超时时间到来，DeferredResult启动超时线程，超时线程处理业务，
   * 封装返回数据，给DeferredResult赋值（正确返回的或错误返回的）
   * */

  private DeferredResult<String> deferredResult;

  /*
   * 产生DeferredResult 给控制器用，通过 @Scheduled 注解定时 更新 产生DeferredResult
   * */
  public DeferredResult<String> getAsyncUpdate() {
    deferredResult = new DeferredResult<String>();
    return deferredResult;
  }

  @Scheduled(fixedDelay = 2000)  //上一个完成后，等两秒执行 。这是个定时任务
  public void refresh() {
    if (deferredResult != null) {
      deferredResult.setResult(Long.toString(System.currentTimeMillis()));
    }
  }
}
```

2.写Async 的控制器

```java
package com.example.springmvc.sse;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.context.request.async.DeferredResult;

@Controller
public class AsyncController {

  /*
   * Servlet3.0+ 为异步提供的接口是 DeferredResult和 Callable ，这里的服务是DeferredResult
   * Callable和DeferredResult做的是同样的事情——释放容器线程，
   * 在另一个线程上异步运行长时间的任务。不同的是谁管理执行任务的线程。
   * DeferredResult 是自己管理线程：创建一个线程并将结果set到DeferredResult是由我们自己来做的。
   * Callable 这里没用到，就不说了。
   * */
  @Autowired
  PushService pushService;

  @RequestMapping("/defer")
  @ResponseBody
  public DeferredResult<String> deferredCall() {
    return pushService.getAsyncUpdate();
  }
}

```

3.再写完async.html在templates/sse问件夹下。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Servlet Async support</title>
</head>
<body>
  <script src=jquery-1.11.3.min.js></script>
  <script>
    /*
    * 这里使用的是 Ajax 请求，没有浏览器兼容性的问题
    * $.get()是简写 $.ajax()相当于是
    * $.ajax({
    *   url: "defer", 这是 相对于 async.html的url 相对地址，
    *   success:function(data){data是 访问成功之后服务端返回的信息
    *     console.log(data)
    *     deferred();
    *   }
    * })
    * */
    deferred();   // 这个页面打开后就向后台发出请求
    function deferred() {
      $.get('defer',function (data) { //向路径 /defer 的页面发送请求
        console.log(data);//在控制台输出服务端推送的内容
        deferred(); //一次请求完成后，再向后台发出请求
      })
    }
  </script>
</body>
</html>
```

4.在MyMvcConfig.addViewController里增加

`registry.addViewController("/async").setViewName("sse/async");`

5.运行后 在浏览器输入 ，localhost:8088/async

---

#### <a name="ceshi">Spring MVC测试</a>

​	本节中，我们要进行一些和Spring MVC 相关的测试，主要涉及控制器的测试。为了测试Web项目，通常不启动项目，我们需要一些Servlet 相关的模拟对象，比如 **MockMVC ，MockHttpServletRequest ，MockHTTPServletResponse ，MockHTTPSession** 等。在 Spring 里我们使用 **@WebAppConfiguration** 指定 加载 **ApplicationContext** 是一个 **WebApplicationContext**。

​	其实我们启动项目，试着按照我们的思路运行也算是一种测试。可是在现实开发中，我们是先知道需求才进行开发的，这个时候可以引入 **测试驱动开发 (Test Driven Development，TDD )，**我们按照需求先写一个自己预期结果的测试用例，可能一开始是个失败的测试用例，但是不断的重构和编码，做种让测试通过，这样才能保证软件的质量和可控性。

​	在下面我们借助 JUnit， Spring TestContext framework ，分别演示对普通页面转向型控制器和RestController 进行测试。

Demo：

 1. 测试需要引入依赖

    ```xml
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>5.1.2.RELEASE</version>
    </dependency>
    
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
      <!--scope  属性表示 引入的包的 生存周期，这里是test，也就是说发布时，我们将不包含这些包-->
    </dependency>
    ```

2. 创建演示需要的服务

   ```java
   package com.example.springmvc.test;
   
   import org.springframework.stereotype.Service;
   
   @Service
   public class DemoService {
     public String saySomething(){
       return "hello?";
     }
   }
   ```

3. 在src/test/java 下 创建测试用例

   ```java
   package com.example.springmvc;
   
   import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.forwardedUrl;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.model;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.view;
   
   import com.example.springmvc.test.DemoService;
   import org.junit.Before;
   import org.junit.Test;
   import org.junit.runner.RunWith;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.mock.web.MockHttpServletRequest;
   import org.springframework.mock.web.MockHttpSession;
   import org.springframework.test.context.ContextConfiguration;
   import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
   import org.springframework.test.context.web.WebAppConfiguration;
   import org.springframework.test.web.servlet.MockMvc;
   import org.springframework.test.web.servlet.setup.MockMvcBuilders;
   import org.springframework.web.context.WebApplicationContext;
   
   /*
    * 声明加载ApplicationContext是一个 WebApplicationContext 。它的属性指定的是Web资源的位置,默认是src/main/webapp这里改为.../resources
    * */
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration(classes = {MyMvcConfig.class})
   @WebAppConfiguration("src/main/resources")
   public class TestControllerIntegrationTests {
   
     private MockMvc mockMvc; //模拟 MVC对象，初始化是下面Before 注解的内容，
   
     @Autowired
     private DemoService demoService; //在测试中注入的Bean
   
     @Autowired
     WebApplicationContext wac; //注入 WebApplicationContext
   
     @Autowired
     MockHttpSession session; //可注入 模拟的http session ，此处只做演示，没有使用
   
     @Autowired
     MockHttpServletRequest request; //可注入 模拟的http request ，此处只做演示，没有使用
   
     @Before  //测试开始前的工作，，这里是初始化 mockMvc对象
     public void setup() {
       this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
     }
   
     @Test
     public void testNormalController() throws Exception {
   
       mockMvc.perform(get("/normal"))  //模拟向 /normal 进行 get请求
           .andExpect(status().isOk())  //预期返回状态为 200
           .andExpect(view().name("testPage")) //预期的页面为 page
           .andExpect(forwardedUrl("/testPage"))  // 预期页面真正路径为
           .andExpect(model().attribute("msg", demoService.saySomething()));
       //预期model里的内容时 demoService.saySomething()的返回值hello
     }
   
     @Test
     public void testRestController() throws Exception {
       mockMvc.perform(get("/testRest"))
           .andExpect(status().isOk())
           .andExpect(content().contentType("text/plain;charset=UtF-8")) //预期返回的类型
           .andExpect(content().string(demoService.saySomething())); //预期返回的内容 
     }
   }
   ```

4. 编写普通控制类

   ```java
   package com.example.springmvc.test;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Controller;
   import org.springframework.ui.Model;
   import org.springframework.web.bind.annotation.RequestMapping;
   
   @Controller
   public class NormalController {
     @Autowired
     DemoService demoService;
   
     @RequestMapping("/normal")
     public String testPage(Model model){
       model.addAttribute("msg",demoService.saySomething()) ;
       return "testPage";
     }
   }
   ```

5. 在static目录下编写 testPage.html 

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
     <meta charset="UTF-8">
     <title>Test Page</title>
   </head>
   <body>
     <pre>
       Welcome to Spring MVC world!
     </pre>
   </body>
   </html>
   ```

6. 编写 RestController 控制器

   ```java
   package com.example.springmvc.test;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.ResponseBody;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class MyRestController {
     @Autowired 
     DemoService demoService;
     
     @RequestMapping(value = "/testRest" , produces = "text/plain;charset=UTF-8")
     public @ResponseBody String testRest(){
       return demoService.saySomething();
     }
   }
   ```

   ​	个人猜测：这里声明下  现在项目中的 src/main/resources 目录在运行后，逻辑目录为  /WEB-INF/classes   实际上在项目中为target 目录，里面的结构应该与 /web-inf/ 一致	 。以及测试中的页面是由thymeleaf 控制的，所以 testPage 放在 templates 目录下。

   ​	.andExpect(forwardedUrl(null))   预期页面真正路径为 null 这是因为 我的项目中没有/web-inf/classes/这样的结构
