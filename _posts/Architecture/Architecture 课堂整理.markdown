这个文档是整理课堂上我觉得陈明俊老师讲的比较精髓的地方，也会结合ppt，所以会比较杂乱。也为自己期末复习做准备。

---

#### 2018-12-10 

​	接上节课讲的有关微服务和REST之后，我在另一篇文档中记录了一些：[About Rest](https://krystalics.github.io/2018/12/03/About-REST/) ，这节课继续讲了关于SOAP的部分内容和**Web Application Architecture**的内容。

##### What‘s SOAP

​	SOAP 是一种简单的基于 XML 的协议，它使应用程序通过 HTTP 来交换信息。仅仅把HTTP当做通信协议，这是和REST最大的区别。**或者更简单地说：SOAP 是用于访问网络服务的协议**，虽然SOAP可以在各种消息传递系统中使用，并且可以通过各种传输协议提供，但SOAP最初的重点是通过HTTP传输的远程过程调用。也有其他框架（包括CORBA，DCOM和Java RMI）提供与SOAP类似的功能，但SOAP消息完全用XML编写，因此独立于平台和语言。

​	课堂中也没有介绍更具体的东西，上述篇幅是我google了一下，得到的简单结论。老师只是拿了一个 JDK 1.6之后集成了SOAP，通过annotation 来调用SOAP服务。@WebService 和 @WebMethod 。让我们不用写通信代码，这方面的工作都是它帮我们做了，完全是透明的。

简单例子如下：

- 写服务程序，生成随机数

```java
package com.company;

import java.util.Random;
import javax.jws.WebMethod;
import javax.jws.WebService;

@WebService
public class RandService {
  private static final int maxRands=16;
  
  @WebMethod
  public int next1(){
    return new Random().nextInt();
  }
  
  @WebMethod
  public int[]nextN(final int n){
    final int k=(n>maxRands? maxRands:Math.abs(n));
    int []rands=new int[k];
    Random random=new Random();
    for(int i=0;i<k;i++){
      rands[i]=random.nextInt();
    }
    return rands;
  }
  
}

```

- 再写Main程序

```java
package com.company;

import javax.xml.ws.Endpoint;

public class Main {

  public static void main(String[] args) {
    // write your code here
    final String url="http://localhost:8888/rs";
    System.out.println("publishing RandService at endpoint "+url);
    Endpoint.publish(url,new RandService());
  }
}
```

- 运行程序，然后在浏览器访问http://localhost:8888/rs?xsd=1 得到下面的结果。

<img src="https://github.com/krystalics/MyPostPicture/blob/master/48.png?raw=true">

具体后面的就不多说了，估计在后面碰到SOAP的情况也不多。



##### 互联网的微服务模型架构

​	像上面一样，我们将服务写好了。怎么提供给别人使用呢？我们会将它注册到UDDI，相当于一个注册表，记录所有的SOAP服务，类似于包管理器。整个流程如下图：

<img src="https://github.com/krystalics/MyPostPicture/blob/master/49.png?raw=true">

基本上所有的服务都是这种模式，只是表现形式不一样。像Dubbo，以下图片来自Dubbo官网

<img src="https://github.com/krystalics/MyPostPicture/blob/master/50.png?raw=true">



##### Web Application Architeture

​	关于这部分我也听的不是很分明。老师重点讲了伸缩性(Scalability)：可伸缩性(可扩展性)是一种对软件系统计算处理能力的设计指标，高可伸缩性代表一种弹性，在系统扩展成长过程中，软件能够保证旺盛的生命力，通过很少的改动甚至只是硬件设备的添置，就能实现整个系统处理能力的线性增长，实现高吞吐量和低延迟高性能。

​	提倡每一个功能都部署在一个或多个服务器。访问量大的就部署多个，需求量少的就部署一个。这样的好处有很多，下面举出一个。访问不同的域名，就是使用它的功能。

>Handling higher concurrency levels,concurrency means how many users can use your application at the same time without affecting their user experience.

在大部分公司早期，就只有一个服务器，大概率是他们的个人电脑。公司发展，访问量变大，服务器就要更新。再之后成长为数据中心，分布式集群。再之后的发展就是将动态和静态内容分离，将静态内容交个CDN(Content Delivery )。见下图：

<img src="https://github.com/krystalics/MyPostPicture/blob/master/51.png?raw=true">



<img src="https://github.com/krystalics/MyPostPicture/blob/master/52.png?raw=true">



Isolation ： 核心在于将web应用切片成一个个小且独立的服务，放置到独立的服务器上。来拓展伸缩性，服务量大的就配置多个服务器，少的就配置少点。增强伸缩性，一般可以通过增加服务器数量来解决。



##### Content Delivery Network（CDN）Scalability for Static Content 

​	CDN 内容分发网络 ：专门为静态资源（images，JavaScript，CSS，videos ...）搭建的服务。工作原理就像 Http  Proxy。至于什么是代理呢：[请看这里](https://zh.wikipedia.org/wiki/%E4%BB%A3%E7%90%86%E6%9C%8D%E5%8A%A1%E5%99%A8)

下面的图片展示了CDN参与时，用户访问某个网站时的工作流程：

<img src="https://github.com/krystalics/MyPostPicture/blob/master/53.png?raw=true">

图中的顺序 1.表示用户想要访问某网站   2.DNS解析域名  3. 请求到达服务器  4.从CDN加载静态内容。



##### Round—robin DNS

​	Round—robin DNS 是一种[负载分配](https://en.wikipedia.org/wiki/Load_distribution)，[负载平衡](https://en.wikipedia.org/wiki/Load_balancing_(computing))或[容错](https://en.wikipedia.org/wiki/Fault-tolerance)的技术，通过管理[域名系统](https://en.wikipedia.org/wiki/Domain_Name_System)（DNS）响应来解决请求，从而为多个冗余[Internet协议](https://en.wikipedia.org/wiki/Internet_Protocol)服务主机（例如[Web服务器](https://en.wikipedia.org/wiki/Web_server)， [FTP服务器](https://en.wikipedia.org/wiki/FTP_server)）提供服务根据适当的统计模型从客户端计算机。

​	允许我们映射域名到多个IP地址，(每个IP都是一台服务器)。当有多个服务器为同一个域名服务时，如何平衡它们之间的访问量使得资源利用率较高且访问快速。算法有很多种，比如转轮法，就像CPU分配进程时间一样，每个进程分配规定时间，然后调下一个进程。服务器轮流接受请求。也有优先分配请求到请求数量最少的服务器上的算法...有很多，就不列举了。



##### Scalability for a Global Audience

​	当访问量变得很大时(达到几百万往上)，我们可能需要更多数量的数据中心。虽然一个数据中心可以通过增加服务器数量来解决问题，但是对于离这个数据中心比较远的国家或地区，用户请求的响应时间变得很长，用户体验极差。多个数据中心也可以在某个中心遭受洪水，地震等灾害被破坏后数据并不会消失。

​	有了多个数据中心之后，我们访问请求就会被分配到距离我们最近的中心。这个是怎么做到的呢。**GeoDNS 为您服务：**

​	GeoDNS 将根据用户的地理位置和域名解析为IP地址，这个IP里用户近，请求响应快些。

​	**多个数据库服务器和多个数据中心都会有很麻烦的问题需要解决，如数据的一致性如何保证。这些问题非常复杂，这里不介绍。**

##### Edge Cache

​	是用户附近的Http缓存服务器，缓存用户的请求响应，如果在重新访问就会在这里响应，而不用到更深层次的服务器。

下图是 用户在不同地方访问同一个域名的流程。数据中心如果是两个，图的表达意思会更明确；

<img src="https://github.com/krystalics/MyPostPicture/blob/master/54.png?raw=true">

说了那么多，数据中心的结构我们还没好好分析一下。不要认为就是简单的一堆服务器的集群，里面的结构可以很复杂。而且对响应时间和伸缩性的影响会很大。

<img src="https://github.com/krystalics/MyPostPicture/blob/master/55.png?raw=true">

图中详细描述了用户访问网站的整个流程，其中数据中心部分将其内部结构图也展示了出来。不是很清晰，所以会解析一下：

② 是负载均衡器，每个到数据中心的请求都会经过它**，让web的流量均匀分布到③或④上。**

③ Front Cache 1~N 这些缓存可以在数据中心的外部，或者干脆不要这些缓存

④ Front App Server 1~N ，**这一层一般会采用轻量的语言编写，如python ,NodeJs, java中的Spring也是这一层，负责和前端浏览器联系，交互，没有任何业务逻辑的代码。**

⑤ Cache Server，在前端服务器中没有的会先访问缓存服务器，或⑥消息队列

⑦ Web Services Server 1~N ，**这一层是业务逻辑。关于应用的所有逻辑基本在这里。这样可以做到业务变换时代码变换最小。**Java在业务逻辑上很有优势，同时这一层 python , go 等也可以写。**通常这一层是最需要将功能分离成一个个的小模块，独立到服务器上，拓展伸缩性。**

⑧ Data Store Servers ，数据库是最不好做伸缩性的，消耗时间又长，所以才要设置这么多层级和缓存来消耗请求数量，不然大量的请求和流量堆积在数据库就会导致响应变得奇慢无比，系统崩溃。这种情况用术语来讲就是，流量穿透。

⑨ Search Servers 搜索引擎服务器

像这样分工明确的层级和机器，它们的通信如果也是采用HTTP协议速度上就会慢很多，一般大型公司的数据中心内部的通信协议是二进制的协议，速度快。

还有在这样一个系统中增加或者减少服务器的数量并不是简单的数学问题，有个首先要解决的问题就是无状态**stateless**，因为Web Application经常要进行伸缩，所以这个stateless怎么确保就非常关键。保持stateless ,**简单点说就是每一个请求都是独立的，并不会出现这个请求是为了完成某个目的的一部分请求。**

只要保持 Web Application completely staeteless 就可以简单的通过加减服务器的数量完成伸缩。不过这通常很难，这一部分也不多说。

#####  **值得注意的是**

​	**并不是说上图的每一个部分我们都是一定需要的，全都用的话技术上不说，维护的成本将会非常高，只要针对实际情况运用一些就可以了。**



##### 关于架构的一些了解

​	架构并不是某一种语言特有的，也不是某种框架或技术，它围绕着业务模型运转。业务逻辑是架构的核心。下面的图就很好的阐述了这个理念也是对上图的细化。

<img src="https://github.com/krystalics/MyPostPicture/blob/master/56.png?raw=true">





##### Web Services 

​	SOA Service-Oriented Architecture 面向服务架构是这些年一直在提的东西，现在很火的微服务就是从这里衍生出的体系。在这个架构内，把业务需求看成是一种服务，而所有的服务都使用同一种通信协议，使他们可以相互引用。而具体的服务实现细节并不重要，无论是C，Java，Python都可以将它们融合在一起。组成更大的服务。如下图所示

<img src="https://github.com/krystalics/MyPostPicture/blob/master/57.png?raw=true">



暂时先到这里啦。感觉真的对服务的架构有了清晰的认识，不在时以前懵懂的只知道服务器了。

---

参考文章：

[W3CSchool](http://www.w3school.com.cn/soap/soap_intro.asp)

[解道，关于scalable的解释](https://www.jdon.com/scalable.html)