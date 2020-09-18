<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [第七章：Spring Boot 的Web开发](#%E7%AC%AC%E4%B8%83%E7%AB%A0spring-boot-%E7%9A%84web%E5%BC%80%E5%8F%91)
  - [7.1 Spring Boot 的Web开发支持](#71-spring-boot-%E7%9A%84web%E5%BC%80%E5%8F%91%E6%94%AF%E6%8C%81)
  - [7.2 Thymeleaf模板引擎](#72-thymeleaf%E6%A8%A1%E6%9D%BF%E5%BC%95%E6%93%8E)
    - [7.2.1 Thymeleaf的基础知识](#721-thymeleaf%E7%9A%84%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86)
  - [7.2.2与7.2.3跳过](#722%E4%B8%8E723%E8%B7%B3%E8%BF%87)
  - [7.2.4 实战](#724-%E5%AE%9E%E6%88%98)
  - [7.3 Web相关配置](#73-web%E7%9B%B8%E5%85%B3%E9%85%8D%E7%BD%AE)
    - [7.3.1 Spring Boot 提供的自动配置](#731-spring-boot-%E6%8F%90%E4%BE%9B%E7%9A%84%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE)
  - [7.3.2 接管Spring Boot 的Web配置](#732-%E6%8E%A5%E7%AE%A1spring-boot-%E7%9A%84web%E9%85%8D%E7%BD%AE)
  - [7.3.3 注册Servlet，Filter，Listener](#733-%E6%B3%A8%E5%86%8Cservletfilterlistener)
  - [7.4 Tomcat配置](#74-tomcat%E9%85%8D%E7%BD%AE)
    - [7.4.3 替换Tomcat](#743-%E6%9B%BF%E6%8D%A2tomcat)
    - [7.4.4 SSL配置](#744-ssl%E9%85%8D%E7%BD%AE)
  - [**书中的操作好像过时了，所以照着这里来就可以**：springboot 2 HTTP转HTTPS还有用https访问的](#%E4%B9%A6%E4%B8%AD%E7%9A%84%E6%93%8D%E4%BD%9C%E5%A5%BD%E5%83%8F%E8%BF%87%E6%97%B6%E4%BA%86%E6%89%80%E4%BB%A5%E7%85%A7%E7%9D%80%E8%BF%99%E9%87%8C%E6%9D%A5%E5%B0%B1%E5%8F%AF%E4%BB%A5springboot-2-http%E8%BD%AChttps%E8%BF%98%E6%9C%89%E7%94%A8https%E8%AE%BF%E9%97%AE%E7%9A%84)
  - [7.5 Favicon配置：](#75-favicon%E9%85%8D%E7%BD%AE)
  - [7.6 WebSocket](#76-websocket)
    - [7.6.1 什么是WebSocket](#761-%E4%BB%80%E4%B9%88%E6%98%AFwebsocket)
    - [7.6.2 实战](#762-%E5%AE%9E%E6%88%98)
  - [7.7 基于Bootstrap和 AugularJs（Vue） 的现代Web应用](#77-%E5%9F%BA%E4%BA%8Ebootstrap%E5%92%8C-augularjsvue-%E7%9A%84%E7%8E%B0%E4%BB%A3web%E5%BA%94%E7%94%A8)
    - [7.7.1 Bootstrap](#771-bootstrap)
    - [7.7.2 AugularJS](#772-augularjs)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

《Spring Boot 实战》汪云飞*

---

### 第七章：Spring Boot 的Web开发

​	Web 开发是开发中至关重要的一部分，Web开发的核心内容主要包括内嵌Servlet容器和Spring MVC。

#### 7.1 Spring Boot 的Web开发支持

​	提供了spring-boot-starter-web 为Web开发予以支持。其中嵌入了Tomcat以及Spring MVC依赖。具体细节这里就不讲了。

#### 7.2 Thymeleaf模板引擎

​	这是在我之前看这本书的时候就已经用了的、这里有一些对其语法一些归纳可以了解一下，更具体的要去官方文档查看。

##### 7.2.1 Thymeleaf的基础知识

​	Thymeleaf是一个Java的类库，它是一个 xml/xhtml/html5 的模板引擎，可以作为MVC的Web应用的View层。Thymeleaf还提供了额外的模块与Spring MVC集成，下面演示一些用法，其中引入了Bootstrap(样式控制)，Jquery(DOM操作)。

- 引入Thymeleaf：通过 @{} 引用Web资源。而不是使用访问路径。这就相当于 项目目录resources/static/public 前缀。如下面的例子中， jquery-1.10.2.min.js 就是在static 目录下。

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>Bootstrap Template</title>
  <link th:src="@{bootstrap/css/bootstrap.min.css}" rel="stylesheet"/>
  <link th:src="@{bootstrap/css/bootstrap-theme.min.css}" rel="stylesheet"/>
  <!--通过 @{} 引用Web资源。而不是使用访问路径。
  这就相当于 项目目录 resources或static或public 前缀。如下面的例子中，
  jquery-1.11.3.min.js 就是在resources 目录下。-->
</head>
<body>

<h1>hello Thymeleaf学习</h1>
<p>
  访问js中的model中的数据：形式如下
  《script th:inline="javascript"》
      var single=[[${singlePerson}}]];
      console.log(single.name);
  《/script》
  通过 th:inline="javascript" 添加到script标签中，这样js代码即可访问model中的属性
  再通过[[${}]]  获得实际值。
</p>
<script th:src="@{jquery-1.11.3.min.js}"></script>
<script th:src="@{bootstrap/js/bootstrap.min.js}"></script>
</body>
</html>
```

- 访问Model中的数据，就是用 th: ....${} 来做动态处理。

```html
<div class="panel panel-primary"> 
   <div class="panel-heading">
       <h3 class="panel-title">
           访问Model
       </h3>
       
    </div>
    <div class="panel-body">
        <span th:text="${singlePerson.name}"></span>
    </div>
</div>
```

- model中的数据迭代：下面用 th:each做循环，person作为迭代元素来使用。

```html
<div class="panel panel-primary">
  <div class="panel-heading">
    <h3 class="panel-title">列表</h3>
  </div>
  <div class="panel-body">
    <ul class="list-group">
      <li class="list-group-item" th:each="person:${people}">
        <span th:text="${person.name}"></span>
        <span th:text="${person.age}"></span>
      </li>
    </ul>
  </div>
</div>
```

- 数据判断

```html
<div th:if="${not #list.isEmpty(people)}">
    
</div>
```

#### 7.2.2与7.2.3跳过

​	讲的是Thymeleaf与Spring MVC的集成还有Spring Boot的Thymeleaf支持。这里就不写了。

#### 7.2.4 实战

- 新建Spring Boot项目，引入依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf </artifactId>
</dependency>
```

- 实例JavaBean ：此类用来在模板展示数据用，包含name和id属性，这里取前面几章写的DemoObj类

- 将静态资源放入static目录下，如jQuery，和Bootstrap 。
- 数据准备。在之前的DemoController中加入下面内容

```java
@RequestMapping("/bootstrap")
public String bootstrap(Model model){
  DemoObj single=new DemoObj(0,"林家宝");
  List<DemoObj> multi=new ArrayList<>();
  DemoObj obj1=new DemoObj(1,"lin");
  DemoObj obj2=new DemoObj(2,"jia");
  DemoObj obj3=new DemoObj(3,"bao");
  multi.add(obj1);
  multi.add(obj2);
  multi.add(obj3);

  model.addAttribute("singleDemo",single);
  model.addAttribute("multi",multi);
  return "bootstrap_1";//返回 对应html文件名
}
```

- 在 template目录下 创建bootstrap_1.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>Bootstrap Template</title>
  <link th:href="@{/bootstrap/css/bootstrap.min.css}" rel="stylesheet"/>
  <link th:href="@{/bootstrap/css/bootstrap-theme.min.css}" rel="stylesheet"/>
  <!--通过 @{} 引用Web资源。而不是使用访问路径。就相当于项目目录 resources或static或public 前缀
  -->
</head>
<body>

<h1>hello Thymeleaf学习</h1>
<p>
  访问js中的model中的数据：形式如下
  《script th:inline="javascript"》
  var single=[[${singleDemo}]];
  console.log(single.name);
  《/script》
  通过 th:inline="javascript" 添加到script标签中，这样js代码即可访问model中的属性
注意在html中应用这个语法，出错也会导致 themeleaf出错。
</p>

<div class="panel panel-primary">
  <div class="panel-heading">
    <h3 class="panel-title">
      访问Model
    </h3>

  </div>
  <div class="panel-body">
    <span th:text="${singleDemo.getName()}"></span>
  </div>
</div>

<div th:if="${not #lists.isEmpty(multi)}">

  <div class="panel panel-primary">
    <div class="panel-heading">
      <h3 class="panel-title">列表</h3>
    </div>
    <div class="panel-body">
      <ul class="list-group">
        <li class="list-group-item" th:each="demo:${multi}">
          <span th:text="${demo.getId()}"></span>
          <span th:text="${demo.getName()}"></span>
          <!--<button class="btn" th:onclick="'getName('\'+${demo.getName()}+'\');'">获得名字</button>-->
        </li>
      </ul>
    </div>
  </div>

</div>


<script th:src="@{/jquery-1.11.3.min.js}" type="text/javascript"></script>
<script th:src="@{/bootstrap/js/bootstrap.min.js}"></script>

</body>
</html>
```

- 运行并访问localhost:8088/testwar/bootstrap 即可。



#### 7.3 Web相关配置

##### 7.3.1 Spring Boot 提供的自动配置

​	通过查看WebMVCAutoConfiguration 及 WebMVCProperties 的源码可以发现Spring Boot为我们提供了以下自动配置：本来是由我们配置的，如ViewResolver

1. **自动配置的ViewResolver**

   - ContentNegotiatingViewResolver ：这是Spring MVC提供的一个特殊的Vie wResolver。不自己处理View，而是代理给不同的ViewResolver来处理不同的View，所以它的优先级最高。

   - BeanNameViewResolver：在控制器中的一个方法返回值的字符串（view name）会根据BeanNameViewResolver去查找Bean的名称为返回字符串的View来渲染视图。例如：

   ```java
   //定义BeanNameViewResolver的bean
   @Bean 
   public BeanNameViewResolver beanNameViewResolver(){
       return new BeanNameViewResolver();
   }
   
   // 定义一个View的Bean， 名为 jsonView
   @Bean
   public MappingJackson2JsonView jsonView(){
       return new MappingJackson2JsonView();
   }
   
   //在控制器中返回值为字符串 jsonView ，它会Bean名称为jsonView 的视图渲染
   @RequestMapping(value="/json",produces={MediaType.APPLICATION_JSON_VALUE})
   public String json(Model model){
   	DemoObj single=new DemoObj(1,"lin");
       model.addAttribute("single",single);
       return "jsonView"; // 与上面的Bean对应，
   }
   ```

   - InternalResourceViewResolver：这是一个极为常用的ViewResolver。主要通过设置前缀，后缀，以及控制器中方法来返回视图名的字符串，以得到实际页面。


2. **自动配置的静态资源**：在自动配置类的addResourceHandlers 方法中定义了以下静态资源的自动配置。

   1.类路径文件：把类路径下的/static、/public、/resources 和/META-INF/resources 文件加下的静态文件直接映射为/** ，可以通过http://loclhost:8080/** 来访问。

   2.webjar：就是将我们常用的脚本框架封装在jar包中的jar包，如jQuery，bootstrap

  **3.自动配置的Formatter和Converter**，只要我们**定义了Converter，GenericConverter和Formatter接口实现类的Bean，这些Bean就会自动注册到Spring MVC中了。**

  **4.自动配置的HTTPMessageConverters**，如果要增加自定义的HTTPMessageConverter，只需要定义一个自己的HTTPMessageConverter的Bean。然后在此Bean中注册。

```java
//下面是注册代码，  CustomeConverter系列是我们自己定义的。
@Bean
public HttpMessageConverters customeConverter(){
    HttpMessageConverter<?> custome1=new CustomConverter1();
    HttpMessageConverter<?> custome2=new CustomConverter2();
    return new HttpMessageConverters(custome1,custome2);
}
```

5.**静态首页的支持**：在默认类路径classpath:/resources/index.html  , /static/index.html  ,  /public/index.html  , /META-INF/resources/index.html  都可以自动映射为主页。访问根目录时会直接跳到这。

#### 7.3.2 接管Spring Boot 的Web配置

​	如果Spring Boot提供的Spring MVC不符合要求，可以通过配置类(@Configuration) 加上@EnableWebMvc注解来实现自己完全控制MVC配置。

​	当然通常情况下是符合我们需求的。需要额外配置时也可以让配置类继承 WebMvcConfigure 而无须使用注解。如下：既可以保留Spring Boot的原有配置，也可以增加自己的。

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigure{
    @override
    public void addViewControllers(ViewControllerRegistry registry){
        registry.addViewController("/访问路径").setViewName("/在resources / ...html");
    }
}
```



#### 7.3.3 注册Servlet，Filter，Listener

​	当使用嵌入式的Servlet容器(Tomcat，Jetty等)时，我们通过将Servlet，Filter和Listener声明为Bean而达到注册效果。或者注册 ServletRegistrationBean，FilterRegistrationBean和ListenerRegistrationBean 的Bean。

- 直接注册Bean的示例 

```java
@Bean
public XxServlet xxServlet(){
    return XxServlet();
}

@Bean
public YyFilter yyFilter(){
    return YyFilter();
}

@Bean
public ZzListener zzListener(){
    return ZzListener();
}
```

- RegistrationBean的示例

```java
@Bean
public ServletRegistrationBean s(){
    return new ServletRegistrationBean(new XxServlet(),"/xx/*");
}
其他类似。
```



#### 7.4 Tomcat配置

​	虽然说是Tomcat配置，实际上是对Servlet的配置，对Jetty和Undertow都是通用的。

```properties
server.port=8081  #默认是8080 
server.session-timeout=10 #用户会话 session 过期时间，单位为秒
server.context-path=/  # 配置访问路径，默认为 / 
#都是在 application.properties  里面。其他配置，可以百度。这里只做介绍。
```

​	代码配置这里就不演示了。

##### 7.4.3 替换Tomcat

​	Spring Boot默认使用Tomcat作为内嵌Servlet容器。可以换成 Jetty，Undertow 。

 ```xml
<!--换成Jetty只需要将Tomcat的依赖换成Jetty的-->
<dependency>
	<groupId>org.spirngframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
    <!--<artifactId>spring-boot-starter-undertow</artifactId>   也是一样的-->
</dependency>
 ```



##### 7.4.4 SSL配置

​	**secure sockets layer 安全套接层**。是为**网络通信提供安全及数据完整性的一种安全协议**，**SSL在网络传输层对网络连接进行加密。SSL协议位于TCP/IP协议与各种应用层协议之间，为数据通信提供安全支持**。可以分为两层：**SSL记录协议**(SSl  Record Protocol)，它建立在可靠地传输协议(如TCP)之上，为高层协议提供数据封装，压缩，加密等基本功能的支持；**SSL握手协议**(SSL Handshake Protocol) 它建立在SSL记录协议之上，用于在实际数据传输开始前，通信双方进行身份验证，协商加密算法，交换加密秘钥等。

​	在基于B/S的web应用中，是通过Https来实现SSL的。Https是以安全为目标的Http通道。在Http层下加入SSL。Spring Boot 做SSl配置时需要如下操作；  **生成证书**：使用SSl首先需要一个证书，可以是自签名的，也可以是从SSL授权中心获得的。在本例中演示自授权证书的生成。

- 每一个JDK或JRE里都有一个叫keytool的工具，在cmd命令行(前提是JAVA_HOME在Path里)中输入`keytool -genkey -alias tomcat`  (然后按照指示填写内容，**就会在当前目录下生成一个 .keystore文件。**（我个人实践之后发现是在系统的user目录下）。其中会有要你输入两次密码，保证两个密码(一个为Tomcat密码)一致才能顺利启动tomcat

- **Spring Boot 配置SSL**：

```properties
server.ssl.key-store-.keystore
server.ssl.key-store-password=1234
server.ssl.keyStoreType=JKS
server.ssl.keyAlias:tomcat
```

 	在浏览器输入 https://.... 也可以访问了，在这之前是不能的。之后会发现出现下图这个错误

![38.png](https://github.com/MrAlan/MyPostPicture/blob/master/38.png?raw=true)

​	出现这种错误，通常都是配置了不安全的 SSL 版本或者 CipherSuite —— 例如服务器只支持 SSLv3，或者 CipherSuite 只配置了 RC4 系列，使用 Chrome 访问就会得到这个提示。下面一段是我在 [这篇文章中看到的](https://kinsta.com/knowledgebase/err_ssl_version_or_cipher_mismatch/)

>当您访问通过HTTPS运行的网站时，将在浏览器和Web服务器之间执行一系列步骤，以确保证书和SSL / TLS连接有效。其中一些包括TLS握手，针对证书颁发机构检查的证书以及证书的解密。如果由于某种原因浏览器不喜欢它看到的内容（例如配置错误或版本不支持），则浏览器可能会显示以下错误：“ERR_SSL_VERSION_OR_CIPHER_MISMATCH”会阻止您访问该站点。
>
>

​	解决方式通常有四种：[看看这篇文章吧](https://www.trustauth.cn/baike/25469.html)

​	而返回去采用http://来访问会有另一个错误![39.png](https://github.com/MrAlan/MyPostPicture/blob/master/39.png?raw=true)

​	原因是我们定义了ssl 却用http访问是被禁止的。

- **http与https共存：**在程序入口类中加入下面这段代码(spring boot 2.x之后的)

```java
@Bean
public ServletWebServerFactory servletContainer() {
  TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
  tomcat.addAdditionalTomcatConnectors(createHTTPConnector());
  return tomcat;
}

private Connector createHTTPConnector() {
  Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");

  connector.setScheme("http");
  connector.setSecure(false);
  // http 端口
  connector.setPort(8081);
  //https 端口     注意两个端口不能一致
  connector.setRedirectPort(8080);
  return connector;

}
```


#### **书中的操作好像过时了，所以照着这里来就可以**：[springboot 2 HTTP转HTTPS还有用https访问的](https://blog.csdn.net/zsj777/article/details/81431017)



#### 7.5 Favicon配置：

​	默认的是spring boot提供的一片黄叶子favicon ，我们可以关闭它

```properties
spring.mvc.favicon.enabled=false
```

​	然后设置自己的Favicon，只需要将自己的favicon.ico文件放置在类路径根目录，或META-INF/resources/  或 /resources  /static  /public下。

#### 7.6 WebSocket

##### 7.6.1 什么是WebSocket

​	**WebSocket 为浏览器和服务端提供了双工异步通信的功能，**即浏览器可以向服务端发送消息，服务端也可以像浏览器发送消息。通过一个socket来实现双工异步通信能力的，但是直接使用WebSocket(或者SockJS：WebSocket协议的模拟，增加了当浏览器不支持WebSocket的兼容支持)协议开发程序显得特别烦琐，我们会使用它的子协议STOMP , 它是一个更高级的洗衣，基于帧的格式来定义消息，与HTTP的request和response类似。

​	spring Boot 提供的自动配置，以及starter pom 是 spring-boot-starter-websocket

##### 7.6.2 实战

​	新建一个项目，选择Thymeleaf 和 WebSocket依赖。WebSocket分为广播式与点对点式。

**广播式： 服务端有消息，会将信息发送给所有连接了当前endPoint 的浏览器**

- 配置WebSocket 。开启WebSocket支持，并实现 WebSocketMessageBrokerConfigurer  接口，重写方法来配置它。

```java
package com.example.websocket;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;
@Configuration
@EnableWebSocketMessageBroker
// 开启 STOMP 协议 来传输基于代理的消息。这是控制器使用@MessageMapping就像使用@RequestMapping
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

  @Override
  public void registerStompEndpoints(StompEndpointRegistry registry) {
    //注册STOMP协议的节点，并映射指定的URL 和 指定使用 SockJS协议
    registry.addEndpoint("/endpointWisely").withSockJS(); //与 ws.html SockJS节点 里的 一致
  }

  @Override
  public void configureMessageBroker(MessageBrokerRegistry registry) {
    //配置代理消息 (Message Broker)  对于广播式的应该配置一个 /topic 消息代理
    registry.enableSimpleBroker("/topic");
  }
}
```

- 浏览器向服务器发送的消息用一个类WiselyMessage来接受

```java
package com.example.websocket;

public class WiselyMessage {

  private String msg;

  public String getMsg() {
    return msg;
  }
}
```

- 服务端用里一个类WiselyResponse 发送消息

```java
package com.example.websocket;

public class WiselyResponse {

  private String response_msg;

  public WiselyResponse(String response_msg) {
    this.response_msg = response_msg;
  }

  public String getResponseMsg() {
    return response_msg;
  }
}
```

- 演示控制器

```java
package com.example.websocket;

import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Controller;

@Controller
public class WsController {

  @MessageMapping("/welcome")  //映射地址，相当于@RequestMapping在开启了WebSocket时
  @SendTo("/topic/getResponse") //当服务端有消息时，会对订阅了SendTo 中的路径的浏览器发送消息
  public WiselyResponse say(WiselyMessage message) throws Exception {
    /*
     * 这个方法的 返回值 和参数 就是我们定义的 两个类，实际上 从html的js代码中传过来的是 json 字串
     * jackson会将其自动转为 WiselyMessage 对象，但是从实际的运行情况来 装换后虽然称为了 WiselyMessage对象，但是
     * message.getMsg()  得到的值为null
     * */
    Thread.sleep(1000);

    System.out.println(message.getClass() + "----" + message.getMsg() + "----");  //发过来的就是 getMsg == null

    return new WiselyResponse("welcome," + message.getMsg() + " !");
  }
}
```

- 添加脚本将 stomp.min.js ，sock.min.js 以及jQuery放置在 /static 下。

- 演示界面，新建ws.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>Spring Boot + WebSocket 广播式</title>
</head>
<body onload="disconnect()">
<noscript><h2 style="color: antiquewhite">貌似你的浏览器不支持WebSocket</h2></noscript>
<div>
  <div>
    <button id="connect" onclick="connect();">连接</button>
    <button id="disconnect" disabled="disabled" onclick="disconnect();">断开连接</button>
  </div>
  <div id="conversationDiv">
    <label>输入你要发送的信息</label><input type="text" id="message">
    <button id="sendMessage" onclick="sendMessage();">发送</button>
    <p id="response"></p>
  </div>

  <script th:src="@{/sockjs.min.js}"></script>
  <script th:src="@{/stomp.min.js}"></script>
  <script th:src="@{/jquery-1.11.3.min.js}"></script>
  <script>
    /*
    * 若都为 String 而不是 用对象来接受就会简单一些，但是一般都是传输 json 对象。
    * */
    var stompClient=null;

    function setConnected(connected) {
      document.getElementById('connect').disabled=connected;
      document.getElementById('disconnect').disabled=!connected;
      document.getElementById('conversationDiv').style.visibility=connected?'visible':'hidden';
      $('#response').html("连接上了");
    }

    function connect() {
      var socket=new SockJS('/endpointWisely'); //连接SockJS的 endpoint 名称为 endpointWisely
      stompClient=Stomp.over(socket);  //使用 STOMP自协议的WebSocket客户端
      stompClient.connect({},function (frame) {  //连接WebSocket服务端
        setConnected(true);
        console.log('connected'+frame);
        // 通过  stompClient.subscribe 订阅 /topic/getResponse 目标(destination) 发送的消息这个控制器在 @SendTo里定义
        stompClient.subscribe('/topic/getResponse',function (response) {
          showResponse(response.body.toString()); //注意showResponse 这里传的参数 一定要是String类型的
        });
      });
    }

    function disconnect() {
      if(stompClient!=null){
        stompClient.disconnect();
      }
      setConnected(false);
      console.log("Disconnected");
    }

    function sendMessage() {
      var msg=$('#message').val();
      stompClient.send("/welcome",{},JSON.stringify({'message':msg})); //传输一个 json 字符串过去，这里是通过stringify来讲对象转为字符串

    }

    function showResponse(message) {
      var response=$('#response');
      response.html(message);
    }

  </script>
</div>
</body>
</html>
```

[代码传送到github](https://github.com/MrAlan/SpringBoot/websocket)



**点对点式：广播式有自己的应用场景，但是广播式不能解决我们的一个常见场景，即消息由谁发送，由谁接受的问题**

​	本例中演示一个简单的聊天室程序，例子中只有两个用户，互相发消息给彼此。因为需要用户相关的内容，所以需要引入简单的Spring Security相关内容。

- 添加Spring Security 的starter pom 

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
  <groupId>org.thymeleaf.extras</groupId>
  <artifactId>thymeleaf-extras-springsecurity4</artifactId>
</dependency>
```

- Spring Security 的简单配置。

```java
package com.example.websocket.oneline;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

  @Override
  protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    //在内存分配两个用户 ljb 和 wyf 密码都为123  ，角色是 USER
    auth.inMemoryAuthentication()
        .withUser("ljb").password("123").roles("USER")
        .and()
        .withUser("wyf").password("123").roles("USER");
  }

  @Override
  public void configure(WebSecurity web) throws Exception {
    web.ignoring().antMatchers("/bootstrap-3.3.7/**"); // 这个目录下的静态资源 Spring Security不拦截
      //默认路径是 static ,在它下面的子目录，也需要访问的话，就要加到上面
  }

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/", "/login").permitAll() //设置Spring Security 对 / 和 /login 路径不拦截
        .anyRequest().authenticated()
        .and()
        .formLogin()
        .loginPage("/login") // 设置 Spring Security 的登录页面访问路径为 /login
        .defaultSuccessUrl("/chat") //登录成功后转向 /chat 路径
        .permitAll()
        .and()
        .logout()
        .permitAll();
  }
}
```

- 配置WebSocket

```java
package com.example.websocket;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;
@Configuration
@EnableWebSocketMessageBroker
// 开启 STOMP 协议 来传输基于代理的消息。这是控制器使用@MessageMapping就像使用@RequestMapping
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

  @Override
  public void registerStompEndpoints(StompEndpointRegistry registry) {
    //注册STOMP协议的节点，并映射指定的URL 和 指定使用 SockJS协议
    registry.addEndpoint("/endpointWisely").withSockJS(); //与 ws.html SockJS节点 里的 一致
    registry.addEndpoint("/endpointChat").withSockJS(); // 这是点对点的WebSocket注册的  endpoint
  }

  @Override
  public void configureMessageBroker(MessageBrokerRegistry registry) {
    //配置代理消息 (Message Broker)  对于广播式的应该配置一个 /topic 消息代理
    //点对点应增加一个 /queue 消息代理
    registry.enableSimpleBroker("/queue","/topic");
    
  }
}
```

- 写控制器，在之前的WsController里增加代码如下

```java
package com.example.websocket;

import java.security.Principal;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Controller;

@Controller
public class WsController {


  @Autowired
  private SimpMessagingTemplate simpMessagingTemplate; //通过simpMessageTemplate 向浏览器发送消息

  @MessageMapping("/chat")
  public void handleChat(Principal principal, String msg) {
    /*
     * 在 Spring MVC中可以直接在参数中获得principal，里面包含当前用户的信息
     * 下面的代码：如果发送人是 ljb 则将消息发送给 wyf ，反之亦然
     * 通过 simpMessagingTemplate.convertAndSendToUser 向用户发送消息，第一个参数是接收方的用户名，
     * 第二个是浏览器订阅的地址 ，第三个是消息本身
     * */

    if (principal.getName().equals("ljb")) {
      simpMessagingTemplate
          .convertAndSendToUser("wyf", "/queue/notifications",
              principal.getName() + "-send:" + msg);
    } else {
      simpMessagingTemplate.convertAndSendToUser("ljb", "/queue/notifications",
          principal.getName() + "-send:" + msg);
    }
  }

}
```

- 登录界面 login.html  这里引入了 bootstrap 的登录模板

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="www.thymeleaf.org">
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <!-- The above 3 meta tags *must* come first in the head; any other head content must come *after* these tags -->
  <meta name="description" content="">
  <meta name="author" content="">
  <link rel="icon" th:href="@{/bootstrap-3.3.7/docs/favicon.ico}">

  <title>登录页面</title>

  <!-- Bootstrap core CSS -->
  <link th:href="@{/bootstrap-3.3.7/dist/css/bootstrap.min.css}" rel="stylesheet">

  <!-- IE10 viewport hack for Surface/desktop Windows 8 bug -->
  <link th:href="@{/bootstrap-3.3.7/docs/assets/css/ie10-viewport-bug-workaround.css}" rel="stylesheet">

  <!-- Custom styles for this template -->
  <link th:href="@{/bootstrap-3.3.7/docs/examples/signin/signin.css}" rel="stylesheet">

  <!-- Just for debugging purposes. Don't actually copy these 2 lines! -->
  <!--[if lt IE 9]><script th:src="@{/bootstrap-3.3.7/docs/assets/js/ie8-responsive-file-warning.js}"></script><![endif]-->
  <script th:src="@{/bootstrap-3.3.7/docs/assets/js/ie-emulation-modes-warning.js}"></script>

  <!-- HTML5 shim and Respond.js for IE8 support of HTML5 elements and media queries -->
  <!--[if lt IE 9]>
  <script src="https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
  <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
  <![endif]-->
</head>

<body>

<div th:if="${param.error}">
  无效的账号密码
</div>
<div th:if="${param.logout}">
  你已注销
</div>

<div class="container">

  <form class="form-signin">
    <h2 class="form-signin-heading">请登录</h2>
    <label for="inputEmail" class="sr-only">用户名</label>
    <input type="text" id="inputEmail" class="form-control" placeholder="user name" required autofocus>
    <label for="inputPassword" class="sr-only">，密 码</label>
    <input type="password" id="inputPassword" class="form-control" placeholder="Password" required>
    <div class="checkbox">
      <label>
        <input type="checkbox" value="remember-me"> Remember me
      </label>
    </div>
    <button class="btn btn-lg btn-primary btn-block" type="submit">Sign in</button>
  </form>

</div> <!-- /container -->


<!-- IE10 viewport hack for Surface/desktop Windows 8 bug -->
<script th:src="@{/bootstrap-3.3.7/docs/assets/js/ie10-viewport-bug-workaround.js}"></script>
</body>
</html>
```

- 聊天界面 chat.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>Home</title>
  <script th:src="@{/sockjs.min.js}"></script>
  <script th:src="@{/stomp.min.js}"></script>
  <script th:src="@{/jquery-1.11.3.min.js}"></script>
</head>
<body>

<p>
  聊天室
</p>

<form id="wiselyForm">
  <textarea rows="4" cols="60" name="text"></textarea>
  <input type="submit">
</form>

<div id="output">
</div>

<script th:inline="javascript">
  $('#wiselyForm').submit(function (e) {
    e.preventDefault();
    var text = $('#wiselyForm').find('textarea[name="text"]').val();
    sendSpittle(text);
  });

  var sock=new SockJS("/endpointChat");
  var stomp=Stomp.over(sock);
  stomp.connect('guest','guest',function (frame) {
    stomp.subscribe("/user/queue/notifications",handleNotification);
  });

  function handleNotification(message) {
    $('#output').append("<b>Received:"+message.body+"</b><br/>");
  }

  function sendSpittle(text) {
    stomp.send('/chat',{},text);
  }

</script>


</body>
</html>
```

- 增加到viewController上,映射上面两个html

```java
registry.addViewController("/login").setViewName("login");
registry.addViewController("/chat").setViewName("chat");
```

- 运行，预期效果是：两个用户登录系统，可以互发消息。但是一个浏览器的用户会话session是共享的，我们可以在chrome设置两个独立的用户，从而实现用户会话session隔离。如下面这张图

![https://github.com/MrAlan/MyPostPicture/blob/master/40.png?raw=true](https://github.com/MrAlan/MyPostPicture/blob/master/40.png?raw=true)

最终效果图如下：

![41.png](https://github.com/MrAlan/MyPostPicture/blob/master/41.png?raw=true) ![42.png](https://github.com/MrAlan/MyPostPicture/blob/master/42.png?raw=true)

代码同样在 [github传送门](https://github.com/MrAlan/SpringBoot/websocket)



#### 7.7 基于Bootstrap和 AugularJs（Vue） 的现代Web应用

​	现代B/S系统软件有下面几个特色

1. **单页面应用：** single-page application 简称 SPA，指的是一种类似于原生客户端软件的更流畅的用户体验页面。在单页面应用中，所有资源(html,javascript,css)都是按需动态加载到页面上的。并且不需要服务端控制页面的转向。

2. **响应式设计**：Responsive web design 简称 RWD ，指的是不同的设备（电脑，平板，手机）访问相同页面时，得到不同的页面视图，而得到的视图是适应当前屏幕的。当然就算在电脑端我们拖动窗口的大小变化，也能得到响应的视图。

3. **数据导向：**是对于页面导向而言的，页面上的数据根据获得是通过消费后台的REST服务来实现的，而不是通过服务器渲染的动态页面(JSP) 来实现的。一般数据交换使用的格式是JSON



   本节将针对Bootstrap和AugularJS 进行快速入门式的引导。需要深入学习的可以去官网。

##### 7.7.1 Bootstrap

​	**1. 什么是Bootstrap**：官方定义 **Bootstrap是开发响应式和移动优先的Web应用最流行的HTML，CSS，JavaScript 框架**。

##### 7.7.2 AugularJS 

​	AugularJS 官方定义：**AugularJS 是HTML开发本应该的样子，它是用来设计开发Web应用的**

​	AugularJS 使用声明式模板+数据绑定(类似于jsp，thymeleaf)，MVW(Model-View-Whatever)，MVVM(Model-View-ViewModel) , MVC(Model-View-Controller) ，依赖注入和测试。但是这一切的实现只用了 JavaScript。

​	HTML 一般是用来声明静态页面的，但是通常情况下我们希望页面时基于数据动态生成的，这也是我们很多服务端模板引擎出现的原因。而AugularJS只通过前端技术就能得到动态页面。**这和这几年一直很火的前后端分离，前端渲染技术是吻合的，前端开发三剑客，View，Augular，React。**

​	



