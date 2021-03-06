本文参考文章：

[深入分析Java I/O的工作机制](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html) 

[深入分析Java中的中文编码问题](https://www.ibm.com/developerworks/cn/java/j-lo-chinesecoding/)

[Java Nio浅析](https://tech.meituan.com/2016/11/04/nio.html)

实习的工作涉及到了文件的IO，我对这块并不是太熟悉，虽然也不要求我了解这方面的东西。但实际上很久之前，我对NIO和网络以及其他一些比较有难度的包就比较感兴趣，困于没有需求。这次NIO是在周会上说要学，当做工作来做，在这里要强推一波[帆软](http://www.fanruan.com/company) （虽然没什么流量）

---

在了解java 的io包之前，先来了解I/O本身。所有的I/O都分为两个阶段：等待和就绪，例子：read()，分为等待系统可读和真正的读；write()也是等待网卡可以写以及真正的写。等待就绪的阻塞时不使用CPU的，是在空等；真正读写的阻塞是使用CPU的，这个过程很快（因为CPU就很快）属于memory copy，带宽通常在1GB/s以上。一些细节，在下文中给出。

##### 几种常见的I/O模型如下：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/11.png?raw=true">

上图中给出了几个概念：阻塞，非阻塞，同步，异步。java.io的I/O模型是同步阻塞的，java.nio的I/O模型则是同步非阻塞的。

其实在专门查过之前，我对这几个概念也只是有一个模糊的认知，真要让我解释可能也可以说个大概，但是很多东西都不确定。所以专门去查了定义：其中[JAVA NIO 不是同步非阻塞I/O吗，为什么说JAVA NIO提供了基于Selector的异步网络I/O？ - 祖春雷的回答 - 知乎](https://www.zhihu.com/question/27991975/answer/71988550) 回答比较准确：

>  同步异步指的是真正IO操作（数据内核态到用户态的拷贝）是否需要进程参与，而Java nio提供了异步处理，这个异步处理是指编程模型上的异步。基于reactor模式的事件驱动，事件处理器的注册和处理器的执行是异步的。

应用程序的一次IO分成两个阶段（上面说过）

- 第一阶段是IO请求：指的是应用程序通知操作系统我有io的意向，给我准备数据或者空间。这个阶段就是上文中的等待阶段
- 第二阶段是IO操作：指的是操作系统将IO请求的所需数据或者空间已经准备好，可以进行真正的数据拷贝（数据在用户态内核态之间的拷贝）。也叫就绪阶段

**阻塞非阻塞发生在等待阶段（即第一阶段）**，当操作系统并没有将io请求所需的数据或者空间准备好，根据io请求的线程是否等待来区分阻塞与非阻塞。如果线程等待一直等就是阻塞，如果不等就是非阻塞。而非阻塞的情况下如何知道资源已经准备好了呢？**1.轮询 2.通知**

而上面的引用中说的“进程参与数据拷贝”，就是**进程占据CPU执行数据拷贝指令**。**进程向操作系统发出IO请求，内核线程将数据准备好放在内核空间**。**同步**的场景是这样的：**进程去内核空间将数据取到用户态空间进行处理**；而**异步**的场景是：**内核线程自动将数据送到用户态空间，然后进程处理数据。**

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/17.png?raw=true">

举个例子：一家餐馆打电话到啤酒厂订购20箱啤酒(发出IO请求)，啤酒厂派一个员工去仓库取货(数据准备阶段)，员工进入仓库寻找啤酒对应的存放地点(等待磁盘运转)，该员工将啤酒送到餐馆门口。如果餐馆派出一个员工来接受这20箱啤酒，就是同步；如果啤酒厂员工直接把啤酒送进餐馆就是异步。

抽象的过程讲完了，我们聊点代码吧：**异步中用户定义的操作是在实际操作之后调用的。比如你定义了一个操作要显示从SOCKET中读入的数据，那么当读操作完成以后，你的操作才会被调用。**即调用该方法时会先执行读操作，再执行显示的操作。同步的话是**先执行显示的操作发现没有数据** ，要么就等待要么就离开，直到数据读入后的某个时间点再执行显示操作。

这里要指出：**用户态和内核态的划分是在类Unix系统中的划分。内核态是控制计算机的硬件资源，并提供上层应用程序的环境；用户态是上层应用程序活动的空间，应用程序的执行必须依托于内核提供的资源**。系统调用则是为了让上层应用程序能够访问内核空间的资源提供的接口。

在处理IO的时候，阻塞和非阻塞都是同步IO，只有使用了特殊API的才是异步IO，毕竟需要调用内核线程所以和具体的操作系统很有关系。具体情况如下图：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/12.png?raw=true">

而Java 7的 NIO2（也称java AIO）提供了一整套Async API，在Windows下基于原生IOCP的 Proactor模式。再看上图中还有什么陌生的概念，mmm一看还有几个，首先是IO multiplexing多路复用，以及其括号中的select，poll，epoll是什么意思呢？

在**执行非阻塞IO的时候**，只有当系统通知那个描述符(如Socket)可读，采取执行read操作并保证每次read都能读到有效数据而不做单纯的返回-1或者EAGAIN的无用功。

**操作系统通过 select/poll/epoll 之类的系统调用函数来使用，这些函数都可以同时监视多个描述符的读写就绪状况。**这样，**多个描述符的I/O操作都能在一个线程内并发交替地顺序完成，这就是程序中的I/O多路复用。**简单来说，就是多个IO可以在一个线程内操作。

整个非阻塞过程只有调用上面三个系统调用的时候才会阻塞，收发客户消息时不会阻塞的，整个进程或者线程被充分利用(相比于阻塞的情况)，这也叫事件驱动（多路复用的一种实现）——也叫reactor模式。

展示select的过程：遍历用户空间中的数据，直到找到与连接对应的数据。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/13.png?raw=true">

那么在多连接的情况下，每来一个连接就需要遍历一遍用户控件，会导致程序性能大大降低。所以人们改进了IO多路复用模型，发明了一个叫epoll的方法：会将需要IO的连接记录下来，让他们回去等通知，形成一张注册表。每当之前的连接过来就会寻找与之对应的信息。然后进行IO操作，可以避免遍历空间这样的消耗。



事件驱动有一个致命的弱点，在很多情况下只适用于单线程。很明显不适应现代多核CPU，而且使用单线程就不能有阻塞，因为一旦阻塞就不能对其他连接服务。*而使用多线程做异步I/O，马上会面临回调地狱，至于为什么说是地狱，下文中有描述回调的情况*

针对上面的说的弱点，人们做出了改进：

- 利用多进程，每个进程单线程去利用多核CPU。导致新的问题是：进程间状态共享和通信。不过相比提升的性能来说，算是小问题
- 发明了协程

##### **协程的概念是相对多进程或者多线程来说的，它是协作式的用户态线程。**

- 与之相对的，**线程和进程是以抢占式执行的**，意思是系统帮我们自动快速切换线程和进程来让我们感觉同步运行，**这个切换是由系统自动完成**
- **协作式执行是指想要切换线程你必须要用户手动来切换**。

无需系统自动切换（系统自动切换会浪费很多资源），而协程是我们用户手动切换，而且在同一个栈中执行，速度非常快而且节省资源。

与之对应的协程也有自己的缺点：它只能有一个进程，一个线程。一旦发生IO阻塞，这个程序就会卡住。。。所以我们要使用协程之前，必须保证我们所有的**IO都必须是非阻塞的**，或异步IO。

所有的读和写，网络请求接口都要设置成非阻塞式的，当系统内核把这些玩意儿执行完毕以后，再通过回调函数，通知用户处理。在用户空间上看，我们一直保持在一个线程，一个进程中，因此速率极高。这也是NodeJs并发量为什么那么变态的原因，它因为js只要一条线程，所以所有IO都是非阻塞异步的，通过回调来告诉用户。

上文中说，通过回调函数来告诉用户；问题就在着了

- 我们进行IO操作的目的是因为我们想获得某些数据然后再开始进行操作，所以我们不得不在回调函数中操作
- 在回调函数中获得我们想要的数据，依靠这段数据又想要获取另一段数据（参考Sql的子查询），只能继续回调
- 如果，回调层次过多（例如：10000层），人都会疯掉

所以协程真正的目的并不是高并发，而是为了解决这种无限的回调地狱。就像使用同步方法一样去写异步调用。这点可以类比Js中的`async , await`，代码如下

```js
async function method(){
    const res = await _login(data); //向服务器发送登录数据
}
function _login(data){
   return $.ajax({
        method:"post",
        url:...
    })
}
```

本来使用ajax是要有一个回调函数的。

```js
$.ajax({
    method:"post",
    url:...,
    success:function(res){
    	//do something
	}
})
```

说了那么多关于IO的东西，让我们进入Java的IO世界吧。

---

我们知道Java的IO操作在java.io包下，大致可以分为

- 基于字节：InputStream，OutputStream，
- 基于字符：Writer，Reader

而文章中还附加了两个

- 基于磁盘的：File
- 基于网络的：Socket

前面的划分是根据**传输数据的数据格式**，后面的划分是**根据传输数据的方式**。文章中总结了一句我觉得很经典

> I/O的核心问题要么是数据格式影响I/O操作，要么是传输方式影响I/O操作

关于InputStream和OutputStream的层次结构这里就不展示了，由于我之前也接触过IO还算是有点熟悉，也知道它们是可以相互组合使用来操作数据，这里给出文章中的一个例子：

```java
OutputStream out=new BufferedOutputStream(new ObjectOutputStream(new FileOutputStream("fileName")));
```

了解IO的同学都知道，IO是基于流的——关于流，*我的认识是数据就像水一样流淌。这里有一点，水一样流淌，意味着数据传输是持续的，并不是一块一块的*。这些操作数据的类，就像是一个个水管，控制着水的流向和速度以及大小。

我们知道在磁盘中存储的是二进制，或者说是字节并没有字符这么一说，而Java的字符流传输就是对字节流的一种包装，所有的I/O都是操作的字节！包装成字符流只是为了让我们人类更好的使用和理解罢了。

上面提到了字符流是对字节流的包装，那么真正接触到磁盘，要存储数据的时候字符流传输的还是字节，那么从我们传进去字符再到磁盘中的字节，中间肯定经过了一层转换。字符到字节的转换时很耗费计算资源的，如果是中文的话就更是如此，甚至还可能引发乱码问题。

接下来我们岔开主线，讨论一下编码。提起编码，相信大家脑海中都会浮现ASCII，UTF-8之类的，也大概知道编码是为了让我们人类能够和机器交互。各个编码的区别第二篇文章里有详细的讲述我这里就不赘述了。

我们直接进入它的核心部分：Java I/O操作中的编码层（即字符和字节的转换层）直接盗图啦

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/1.png?raw=true">

Reader是I/O中读字符的父类，而InputStream是读字节的父类。将二者关联起来的就是图中的InputStreamReader（连名字都起得很有关联性）。它负责在I/O过程中处理读取字节到字符的转换，而具体的转换则通过StreamDecoder实现。因为读是从本地磁盘中读，读的只能是字节码。所以读这边的转换时从 字节到字符。

StreamDecoder解码过程必须由用户指定Charset编码格式（如UTF-8），将字节转为对应的编码的字符，如果没有指定字符集，它将默认使用本地环境中的默认字符集（中文环境中就是GBK啦）

同样的写的情况与读的类似。写字符的父类是Writer，写字节的父类是OutputStream。通过OutputStream转换字符和字节

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/2.png?raw=true">

因为写的可能是字符，而磁盘中的只能是字节。所以写过程中的转换时单向的字符到字节。一个例子如下：

```java
String file = "C:\\Users\\12857\\Desktop\\iotest.txt";
String charset="UTF-8";

//将字符转换成字节，写进磁盘
FileOutputStream outputStream=new FileOutputStream(file);
OutputStreamWriter writer=new OutputStreamWriter(outputStream,charset);
writer.write("这是要保存的中文字符。。。。。。。。。。。。。");
writer.close();

//读取字节 转成我们看得懂的字符
FileInputStream inputStream=new FileInputStream(file);
InputStreamReader reader=new InputStreamReader(inputStream,charset);
StringBuilder builder=new StringBuilder();
char[] buf=new char[64]; //一次读64个字符
while((reader.read(buf))!=-1){
    builder.append(buf);
}
reader.close();
System.out.println(builder.toString());
```

一般指定编码都是 UTF-8，用本地环境如果放到其他环境部署很可能出现乱码。

本地磁盘的数据传输过程已经了解了，接下来要看看Java web中涉及的编码了。网络中传输的数据都是以字节为单位的，所有**数据都必须能够被序列化为字节**，而Java中的数据被序列化必须继承Serializable接口——很常见的就是 SpringBoot中的entity模块中的table类，需要继承Serializable，因为很多时候需要传输它们。

用户从浏览器发起一个HTTP请求，需要编码的地方有：URL，Cookie，Parameter。服务器接收到HTTP请求后要解析HTTP协议，其中URI，Cookie和POST表单参数需要解码。服务器还需要读取数据库中的数据，本地文件或者网络中其他服务器的文件，这些数据都存在编码问题，当Servlet处理完所有请求的数据后，需要将这些数据再编码通过Socket发送到用户请求的浏览器中，经过浏览器界面后解析成文本数据。

了解了Web中需要编码的地方之后，可以顺便知道一下遇到乱码应该怎么处理：要知道乱码的核心都是 **char到byte或者byte到char转换中编码和解码的字符集不一致** 。但是由于一次操作可能经过多次编解码，很难查到是哪个环节出了问题

这里列举我在开发中遇到的问题：网站中涉及到数据库的数据，中文全都是问号，英文显示正常。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/3.png?raw=true">

后来查到数据库中的编码是 latin1，和后台以及前端的编码UTF-8并不一致。

好了讲了一大串编码问题之后(我这只是节选原文部分)，让我们回归IO。

上文中举了一个字符和字节相互转换读写文件的例子，但是我们回忆一下我们使用IO的各个类的时候有这么麻烦吗？下面一个例子：

```java
StringBuilder builder=new StringBuilder();
char[] buf=new char[1024];
File file = new File("C:\\Users\\12857\\Desktop\\iotest.txt");
FileReader f=new FileReader(file);
while(f.read(buf)>0){
    builder.append(buf);
}
System.out.println(builder.toString());
```

可以看到例子中并没有InputStreamReader的转换。实际上这是因为FileReader继承了它，通过StringDecoder解码成char。使用了默认字符集（我的本地的是GBK）

我们花了很大的篇幅在讲数据的编解码，还有一个关键的问题是数据的存储过程，**如何将数据持久化到物理磁盘呢？**

---

我们知道数据在磁盘中最小的单位是文件，上层应用程序只能通过文件来操作磁盘上的数据，所以文件也是操作系统和**磁盘驱动器**交互的最小单元。结合Java来看，很有意思的是Java中的File并不代表一个真实存在的文件对象，当我们`new File("C:\\Users\\12857\\Desktop\\iotest.txt")` 的时候它返回一个和这个路径相关联的虚拟对象，这可能是一个真实的文件也可能是一个目录。

为什么要将文件和目录都抽象成是File呢？特别来个Directory好像也很方便呀，这是因为大多数情况下，我们并不关心这个文件是否真的存在，就算没有也可以创建一个。我们真正关心的是该怎样对该文件进行操作。

只有真正要进行读取的时候我们才要检查该文件是不是真的在磁盘中存在。例如FileInputStream类是操作文件的接口，在创建它的时候会创建一个FileDescriptor对象（该对象是真正代表一个存在的文件对象的描述）。当我们在操作一个文件对象的时候，通过FileDescriptor对象获取真正操作的底层操作系统关联的文件描述，比如FileDescriptor.sync()（实际上是调用native的方法，通过C++操作）可以将操作系统缓存中的数据强制刷新到物理磁盘。

从磁盘中读取一段文本，图示如下：（手动画图，略显粗糙）

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/4.png?raw=true">

总结一下：当我们传入一个文件路径，将会根据这个路径创建一个File对象来标识这个文件，然后会根据这个File对象创建真正读取文件的操作对象，以及一个关联真实存在的磁盘文件的文件描述符FileDescriptor，通过这个对象可以直接控制该磁盘文件。读字符文本需要通过StreamDecoder将字节byte编码成char格式，读取过程是由操作系统决定的。

---

##### Java Socket的工作机制

看完了本地磁盘的工作原理，那么就要了解网络的工作机制了。其实Socket只是一个概念，并没有具体到一个实体，它是一种抽象：用来描述计算机之间完成通信的一种功能。它就像是两个城市之间的交通工具（但并没有具体到是火车还是高铁）。

Socket也有很多种（类比交通工具），大多数情况下我们用的都是基于TCP/IP的流套接字。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/5.png?raw=true">

学过计算机网络的同学们看这个图是不是很熟悉，它是TCP/IP网络模型的简单具象，只不过在上层应用和TCP之间加了Socket，由此可知两台主机之间的通信必须通过Socket来连接，就好像两个城市的交流必须要有交通工具一样。

我们知道网络中是通过ip获得远程主机的地址，tcp或udp获得主机上通信的端口，这样就可以通过一个**Socket实例唯一代表一个主机上一个应用程序的通信链路**

当客户端要与服务器通信，客户端首先要创建一个Socket实例，操作系统将为这个实例分配一个没有被使用的本机端口，并创建一个包含本机地址和服务器地址以及端口号的套接字数据结构。在创建Socket实例的构造函数正确返回之前，将要进行TCP三次握手协议，TCP握手协议完成之后，Socket实例将创建完成，否则抛出IOException。

与之对应的服务器将创建一个ServerSocket实例，只需要指定的端口号没有被占用，一般该实例都会创建成功。同时操作系统也会为ServerSocket实例创建一个底层数据结构，这个数据结构中包含指定监听的所有地址。之后调用accept()方法，进入阻塞状态，等待客户端的请求。

当有一个新的请求到来时，将为这个连接创建一个新的套接字数据结构，该套接字数据的信息包含的地址和端口信息，正是请求源地址和端口。这个新的数据结构将会关联到ServerSocket实例的一个未完成的连接数据结构列表中，注意这时服务端与之对应的Socket实例并没有完成创建，而要等到与客户端的三次握手完成后，这个服务端的Socket实例才会返回，并将这个Socket实例对应的数据结构从未完成列表中移到已完成列表。所以ServerSocket所关联的列表中每个数据结构都代表与一个客户端建立的TCP连接

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/6.png?raw=true">

上图是我根据文章描述，画的一幅图，不知道理解是否有误。

好了Socket实例已经创建了，我们的目的是通过Socket传输数据。接下来将描述

##### 数据传输

当连接已经建立成功，服务端和客户端都会拥有一个Socket实例，每个Socket实例都有一个InputStream和OutputStream，通过这两个对象来交换数据。同时我们也知道网络I/O都是以字节流传输的，当Socket对象创建时，操作系统将会为InputStream和OutputStream分别分配一定大小的缓存区，数据的写入和读取都是通过这个缓存区完成的。

写入端将数据写到OutputStream对应的SendQ队列中，当队列满了，数据将发送到另一端InputStream的RecvQ队列中，如果这时RecvQ满了，那么OutputStream的write方法将会阻塞直到RecvQ队列有足够的空间容纳SendQ发送的数据。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/7.png?raw=true">

值的注意的是，这个缓存区的大小以及写入端的速度和读取端的速度非常影响这个连接的数据传输效率，由于可能发生阻塞，所以网络I/O与磁盘I/O在数据的写入和读取还要有一个协调的过程，如果两边同时传送数据时可能造成死锁。

也是由于IO的种种缺陷，才催生出了NIO乃至AIO。

说了那么多，来个经典的BIO的服务程序模型。了解一下，BIO在Web应用中是如何工作的，下面是伪代码，不能正常工作；

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(10, 20, 50, TimeUnit.SECONDS,new ArrayBlockingQueue<>(10)); //线程池
ServerSocket ss=new ServerSocket();
ss.bind(8088);

while(!Thread.currentThread.isInturrpted()){//直到新连接到来一直循环
    Socket socket=ss.accept();
    executor.submit(new ConnectIOonHandle(socket));//为新连接创建新线程
}

class ConnectIOonHandle extends Thread{
    private Socket socket;
    public ConnectIOonHandle(Socket socket){
        this.socket=socket;
    }
    public void run(){
        while(!Thread.currentThread.isInturrpted()&&socket.isClosed()){
            String something=socket.read(); //读取数据
            if(){
                //...
            }
        }
    }
}
```

BIO一般会有线程池为每一个新来的连接创建一个新的线程，但是线程吃的是内存资源，如果线程过多机器就会宕机，过少就没有意义。所以缺陷比较大，需要平衡。适合并发数量小的情况。一般1000连接以下。

BIO模型中，需要多线程是因为I/O操作的时候，没有办法知道到底能不能写，能不能读，只能在哪里等待。除非整个系统就一个连接，否则新的连接来了，不创建线程还能怎么办。











