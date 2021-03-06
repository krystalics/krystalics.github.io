

下面这些tips是我在牛客网AI面试遇到的问题，我把答案集合一下；

---

##### Cookie和Session的区别

由于HTTP协议是无状态的协议，所以服务端需要记录用户的状态时，就需要用某种机制来识具体的用户，这个机制就是Session.

前端用户登录时，后端会产生一个独有的SessionId存储到内存或者Redis中并返回前端。前端管理它的叫做cookie，每次HTTP请求的时候，客户端都会发送相应的Cookie信息到服务端。服务器就能够识别是哪个用户了。

但是从前端到后端的过程中容易被伪造，有大小限制，只能用ASCII 码。而后端的session没有大小限制

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/session.jpg?raw=true">

 

##### 为什么Redis是单线程的

redis 几乎完全是在内存中进行操作的，速度比使用硬盘的数据库要快得多。多线程的上下文的切换，对于一个内存的系统来说，消耗比较大，而且多线程需要各种锁来保证线程安全，对速度有影响。所以因为单线程的Redis够快，所以不需要多线程来并发操作。



至于具体为什么这么快，见这篇博客：

https://blog.csdn.net/xlgen157387/article/details/79470556

外加对mysql和redis速度的定量分析： mysql 每秒钟执行大概几千次，Redis能达到几十万次，二者根本不在一个层面上。

Redis 文章：

<http://database.51cto.com/art/201809/584266.htm#topx> 

##### 怎么将一个ArrayList倒序输出

Collections.reverse(list);  怎么有reverse不将其放进 ArrayList的方法中，而是要另外放在Collections的静态变量中。

像是String也没有reverse方法，而StringBuilder和StringBuffer就有。//如果一个对象没有基本的方法，就去他父类的静态方法中看看，往往有收获，像是 数组 Arrays.sort()  当然数组在java中不是对象。

 

##### MySQL 的慢查找

今天去电信现场初步面试问到的问题，我真是两眼一抹黑啊。听都没听过，感觉和我不是一个技术栈的，，，，继续努力。

MySQL通过慢查询日志定位那些执行效率较低的SQL 语句，用--log-slow-queries[=file_name]选项启动时，mysqld 会写一个包含所有执行时间超过long_query_time 秒的SQL语句的日志文件，通过查看这个日志文件定位效率较低的SQL 。

慢查询主要体现在慢上，通常意义上来讲，只要返回时间大于 >1 sec上的查询都可以称为慢查询。慢查询会导致CPU，内存消耗过高。数据库服务器压力陡然过大，那么大部分情况来讲，肯定是由某些慢查询导致的。

slow_query_log

这个参数设置为ON，可以捕获执行时间超过一定数值的SQL语句。

long_query_time

当SQL语句执行时间超过此数值时，就会被记录到日志中，建议设置为1或者更短。

slow_query_log_file

记录日志的文件名。

log_queries_not_using_indexes

这个参数设置为ON，可以捕获到所有未使用索引的SQL语句，尽管这个SQL语句有可能执行得挺快。



##### Collection和Collections的区别

**Collection是集合接口**。它提供了对集合对象进行基本操作的通用接口方法。Collection接口在Java 类库中有很多具体的实现。Collection接口的意义是为各种具体的集合提供了最大化的统一操作方式。

**Collections 是一个包装类。**它包含有各种有关集合操作的**静态多态方法**。此类**不能实例化**，就像一**个工具类**，服务于Java的Collection框架



##### AOP与IoC 

AOP是面向切面编程：把与主业务无关的部分放到代码外面去做，就像是权限验证（React的高阶组件就是这么个理念），与代理类似：参考知乎的一个回答： <https://www.zhihu.com/question/24863332>   下面是一段静态代理，由我们自己实现

```java
class UserControllerProxy {  //代理模式，写一个新的类来代替原来类的工作，在其基础上修改或添加功能
    private UserController userController;
    
    public void saveUser() {
        checkAuth(); //验证权限
        userController.saveUser();  // 然后再进行Controller的操作
    }
}
```

通俗点来说就是，本来要执行的任务被截断了由AOP来负责。类似于拦截器，过滤器，二者的区别后面再说。

**Spring AOP是基于动态代理** 如果要代理的对象，**实现了某个接口**，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用JDK Proxy去进行代理了（因为JDK代理是创建一个实现了被代理类父类的子类，口语化来说，就是实现它的兄弟类）；这时候Spring AOP会使用**Cglib**，生成一个被代理对象的子类，来作为代理。  这里可以参考： [另一篇文章Java代理](<https://krystalics.github.io/2018/11/06/java%E4%BB%A3%E7%90%86%E4%B8%8EAOP/>)



IoC：控制反转，这个概念不是Spring独有的，而是面向对象语言的一种解耦方式。本来调用一个对象的（非静态）方法需要new出这个对象，但是项目一大起来，对象之间的依赖关系比较复杂，容易出错。所以才出了IoC，把对象的生命周期管理集中了起来交给Spring，有需要用的时候就放出来。

```java
public interface S{
    
}
public class A implements{
    
}

public class B{ //假设B需要调用A的某些方法，并不直接new，而是通过他的父类接口来操作，
    private S s;
    public B(S s){
        this.s=s;  //
    }
}
```



##### 拦截器和过滤器

 Filter可以认为是Servlet的一种“加强版”，它主要用于对用户请求进行预处理，也可以对HttpServletResponse进行后处理，是个典型的处理链使用Filter完整的流程是：Filter对用户请求进行预处理，接着将请求交给Servlet进行预处理并生成响应，最后Filter再对服务器响应进行后处理。

拦截是AOP的一种实现策略，拦截器是动态拦截Action调用的对象。它提供了一种机制可以使开发者可以定义在一个Action执行的前后执行的代码，也可以在一个Action执行前阻止其执行。同时也提供了一种可以提取Action中可重用的部分的方式。拦截器将Action共用的行为独立出来，在Action执行前后执行

**区别：**

- Filter是基于函数回调的，而Interceptor则是基于Java反射的。
- Filter依赖于Servlet容器，而Interceptor不依赖于Servlet容器。
- Filter对几乎所有的请求起作用，而Interceptor只能对action请求起作用。
- Interceptor可以访问Action的上下文，值栈里的对象，而Filter不能。
- 在action的生命周期里，Interceptor可以被多次调用，而Filter只能在容器初始化时调用一次。

##### 二者执行顺序

过滤前-拦截前-action执行-拦截后-过滤后



##### ArrayList和LinkedList

ArrayList是实现了基于动态数组的结构，而LinkedList则是基于实现链表的数据结构。ArrayList的所有数据是在同一个地址上,而LinkedList的每个数据都拥有自己的地址.所以在对数据进行查找的时候，由于LinkedList的每个数据地址不一样，get数据的时候ArrayList的速度会优于LinkedList。 **至于增删改查的操作速度 对比链表和数组就可以。**

ArrayList 在JDK1.8中 默认的初始容量=10，`ArrayList<Integer> it=new ArrayList<>(15); 初始容量改为15` 最大容量为 `Integer.MAX_VALUE - 8 `为什么-8 为最后一次扩容准备，可以扩容到`Integer.MAX_VALUE` , ArrayList 自增长的核心代码如下

```java
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8 //最大长度
transient Object[] elementData; //数据

private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1); //新的容量为原来容量的1.5倍
        if (newCapacity - minCapacity < 0)  //如果新的容量还是小于最小容量，直接定义为minCapacity
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0) //如果容量过大,比预定义的最大容量还打，直接扩到整数最大值
            newCapacity = hugeCapacity(minCapacity); 
       
        elementData = Arrays.copyOf(elementData, newCapacity); //新数组获得原数组的数据
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow  
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```



LinkedList 通过内部 的`Node<E>` 来连接前后元素：

```java
transient Node<E> first;

transient Node<E> last;
```



##### 常用Linux命令

cd  ls  mkdir  touch cp mv  rm...



##### Servlet 

来自知乎回答 <https://www.zhihu.com/question/21416727>

servlet接口定义的是一套处理网络请求的规范，所有实现servlet的类，都需要实现它那五个方法，其中最主要的是两个生命周期方法 init()和destroy()，还有一个处理请求的service()，也就是说，所有实现servlet接口的类，或者说，所有想要处理网络请求的类，都需要回答这三个问题：

- 你初始化时要做什么
- 你销毁时要做什么
- 你接受到请求时要做什么

**你从来不会在servlet中写什么监听8080端口的代码，servlet不会直接和客户端打交道！**

那请求怎么来到servlet呢？答案是servlet容器，比如我们最常用的tomcat，同样，你可以随便谷歌一个servlet的hello world教程，里面肯定会让你把servlet部署到一个容器中，不然你的servlet压根不会起作用。

**tomcat才是与客户端直接打交道的家伙**，他监听了端口，请求过来后，根据url等信息，确定要将请求交给哪个servlet去处理，然后调用那个servlet的service方法，service方法返回一个response对象，tomcat再把这个response返回给客户端。*(通过Socket和ServerSocket类)*



##### 各种编码的情况     [来自osChina的一篇文章](https://my.oschina.net/liting/blog/470021)

我们都知道**ASCII**码只有8位编码，前127位只用来存储英文符号。后来中国接入计算机世界的时候ASCII自然不能满足要求，于是**GB2312**诞生：规定：一个小于127的字符的意义与原来相同，但**两个大于127**的字符连在一起时，就表示一个汉字，前面的一个字节（他称之为高字节）从0xA1用到0xF7，后面一个字节（低字节）从0xA1到0xFE，这样我们就可以组合出大约7000多个简体汉字了。GB2312是对ASCII的中文扩展

但是汉字实在太多了，于是**不再要求低字节一定是127号之后的内码**，**只要第一个字节是大于127就固定表示这是一个汉字的开始，**不管后面跟的是不是扩展字符集里的内容。结果扩展之后的编码方案被称为 GBK 标准，**GBK** 包括了 GB2312 的所有内容，同时又增加了近20000个新的汉字（包括繁体字）和符号。

后来这些编码太杂乱了，显示不同的编码方案都要装不同的软件。所以ISO开始着手解决问题：**UniCode**诞生，它准备搞定所有文化的字母和符号的编码**。直接规定必须用两个字节，也就是16位来统一表示所有的字符，**对于ascii里的那些“半角”字符，UNICODE 包持其原编码不变，只是将其长度由原来的8位扩展为16位，而其他文化和语言的字符则全部重新统一编码。

但是**Unicode并不**和以前的各个编码**兼容**，使得它们之间的**转换必须通过查表来实现**，

互联网的普及，强烈要求出现一种统一的编码方式。UTF-8就是在互联网上使用最广的一种unicode的实现方式。其他实现方式还包括UTF-16和UTF-32，不过在互联网上基本不用。**重复一遍，这里的关系是，UTF-8是Unicode的实现方式之一。**

UTF-8最大的一个特点，就是它是一种变长的编码方式。它可以使用1~4个字节表示一个符号，根据不同的符号而变化字节长度。

UTF-8的编码规则很简单，只有二条：

1. 对于单字节的符号，字节第一位设为0，后面7位为该符号的Unicode码，因此对于英语字母来说UTF-8和ASCII码是相同的
2. 对于**n**字节的符号，第一个字节的**前n位**都**设为1**，**第n+1位为0**，**后面字节的前两位一律设为10**。剩下的位Unicode编码。

| Unicode符号范围（十六进制） | UTF-8编码方式（二进制）             |
| --------------------------- | ----------------------------------- |
| 0000 0000-0000 007F         | 0xxxxxxx                            |
| 0000 0080-0000 07FF         | 110xxxxx 10xxxxxx                   |
| 0000 0800-0000 FFFF         | 1110xxxx 10xxxxxx 10xxxxxx          |
| 0001 0000-0010 FFFF         | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx |



##### 域名劫持 [来自掘金的一篇文章](https://juejin.im/post/59ba146c6fb9a00a4636d8b6)

HTTP发起一个请求：当我们在浏览器请求一个www.baidu.com 的域名时

1. 请求到达运营商的DNS服务器并由其把这个域名解析成对应的IP地址
2. 根据IP地址在互联网上找到对应的服务器，向这个服务器发起一个get/post请求
3. 由这个服务器找到对应的资源原路返回给访问的用户

##### HTTP劫持

分为两种：一种是DNS劫持(域名劫持)，第二种是内容劫持(比较高级基于第一种)

DNS劫持是指在劫持的网络范围内拦截域名解析的请求，分析请求的域名，把审查范围以外的请求放行，否则返回假的IP地址或者什么都不做使请求失去响应，其效果就是对特定的网络不能访问或者访问的是假网址。**本质上就是对DNS解析服务器做手脚，或者使用伪造的DNS解析服务器。**

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/5.png?raw=true">



- 解决办法
  DNS的劫持过程是通过攻击运营商的解析服务器来达到目的。我们可以不用运营商的DNS解析而使用自己的解析服务器或者是提前在自己的App中**将解析好的域名以IP的形式发出去**就可以绕过运营商DNS解析，这样一来也避免了DNS劫持的问题。
- 或者采用HTTPDNS，Https



##### 13.关于数据库的索引，尤其是MySQL

[csdn的一篇文章](https://blog.csdn.net/clh604/article/details/16847481)

索引的作用就是快速的查找资源，代价就是**修改资源的时候要连索引一起改比较麻烦**。索引是由Balance Tree算法构建的，分为B+树（只在节点存储信息），B-树。

如果**WHERE**子句的查询条件里**有不等号**（WHERE coloum!=）或者**函数**（WHERE DAY（column）=）或者在里面使用Like和REGEXP这样的比较操作符时匹配的第一个字符是通配符(如%)。 **不使用索引**，

如果查询条件是LIKE**'%abc’**，MySQL将不使用索引。

LIKE**'abc%‘**，MySQL将使用索引；

在**JOIN**操作中（需要从多个数据表提取数据时），MySQL只有在**主键和外键**的数据类型相同时**才能使用索引**。

**ORDERBY**操作中，MySQL只有在**排序条件不是一个查询条件表达式的情况下才使用索引**。

如果某个数据列里包含许多重复的值，就算为它建立了索引也不会有很好的效果。比如说，如果某个数据列里包含的净是些诸如“0/1”或“Y/N”等值，就没有必要为它创建一个索引。

**复合索引:** Mysql从左到右的使用索引中的字段，一个查询可以只使用索引中的一部份，但只能是最左侧部分。 例如索引是key index (a,b,c). 可以支持a   a,b  a,b,c  3种组合进行查找，但不支持 b,c进行查找 .当最左侧字段是常量引用时，索引就十分有效复合索引可以只使用复合索引中的一部分，但必须是由最左部分开始，且可以存在常量。



##### 保证分布式事务的执行  

[from 掘金](https://juejin.im/post/5b5a0bf9f265da0f6523913b)

首先MySQL的ACID是基本的：

**Atomicity 原子性**：要么不做，要么做全，一旦做不全，就退回重做

**Consistency 一致性**：一个事务执行之前和执行之后数据库的都必须处于一致性状态，即如果事务出错系统中所有变化将回滚，不会对数据进行修改

**Isolation 隔离性**：通常情况下，一个事务所做的修改在最终提交以前，对其他事务是不可见的。分不同的隔离级别。

**Durability 持久性**：一旦事务提交，其所做修改会永久保存在数据库汇总，即使此时系统崩溃也不会造成数据丢失。也分不同级别

事务的**ACID是通过InnoDB日志和锁来保证**。事务的**隔离性是通过数据库锁的机制实现的**，持久性通过**redo log（重做日志）来实现**，**原子性和一致性通过Undo log来实现**。

UndoLog的原理很简单，为了满足事务的原子性，**在操作任何数据之前，首先将数据备份到一个地**方（这个存储数据备份的地方称为UndoLog）。然后进行数据的修改。如果出现了错误或者用户执行了ROLLBACK语句，系统可以利用Undo Log中的备份将数据恢复到事务开始之前的状态。 和Undo Log相反，**RedoLog记录的是新数据的备份**。在事务提交前，只要将RedoLog持久化即可，不需要将数据持久化。当系统崩溃时，虽然数据没有持久化，但是RedoLog已经持久化。系统可以根据RedoLog的内容，将所有数据恢复到最新的状态。



**分布式事务**：指事务的参与者，支持事务的服务器，资源服务器以及事务管理器分别位于不同的分布式系统的不同节点上。简单来说就是，一次大的操作有不同的小操作组成，这些小操作分布在不同的服务器上，且属于不同的应用。

详细的就不说了，看得我脑壳疼。



##### Forward 转发 和 Redirect 重定向 [博客园的文章](https://www.cnblogs.com/selene/p/4518246.html)

Forward是基于服务端，Redirect是基于客户端。

**Forward）**，客户端和浏览器只发出一次请求，Servlet、HTML、JSP或其它信息资源，由第二个信息资源响应该请求，在请求对象request中，保存的对象对于每个信息资源是共享的。


**Redirect）**实际是两次HTTP请求，服务器端在响应第一次请求的时候，让浏览器再向另外一个URL发出请求，从而达到转发的目的。

举个通俗的例子：

**forward**就相当于：“A找B借钱，B说没有，B去找C借，借到借不到都会把消息传递给A”；

**redirect**就相当于："A找B借钱，B说没有，让A去找C借"。

后端的代码如下: redirect直接返回C的URL

```java
public void doGet(HttpServletRequest request,HttpServletResponse response){
    response.sendRedirect("C的URL");
}
```

而forward比较麻烦一些，Web应用程序大多会有一个控制器，由控制器来控制请求应该转发给哪个接口。`javax.servlet.RequestDispatcher`就是请求转发器必须实现的接口，由Web容器为Servlet提供实现该接口的对象，

```java
public void doGet(HttpServletRequest request,HttpServletResponse response){
    RequestDispatcher rd=request.getRequestDispatcher("C的URL")；
    rd.forward(request,response);
}
```

而在SpringMVC中并不需要这么麻烦。只需要 在相应Controller中 `return "forward:/hello" | return "redirect:/hello"` 就可以重定向到hello视图，这需要Themeleaf模板引擎的html做支持或者JSP，如果是前后端分离可能就需要将**hello视图名改为URL**了。



##### 进程创建的步骤

1.申请空白的PCB 进程控制块

2.为新进场分派资源，内存

3.初始化PCB

4.将新进程插入就绪队列  之后轮到它执行才分配cpu

**进程间通信（IPC，InterProcess Communication）**是指在不同进程之间传播或交换信息。IPC的方式通常有管道（包括无名管道和命名管道）、消息队列、信号量、共享存储、Socket、Streams等。其中 Socket和Streams支持不同主机上的两个进程IPC。

线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉，**所以多进程的程序要比多线程的程序健壮，**但在[进程切换](https://www.baidu.com/s?wd=%E8%BF%9B%E7%A8%8B%E5%88%87%E6%8D%A2&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)时，耗费资源较大，效率要差一些。**但对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能用进程。**

线程执行开销小，但不利于资源的管理和保护；而进程正相反。同时，线程适合于在SMP机器上运行，而进程则可以跨机器迁移。























