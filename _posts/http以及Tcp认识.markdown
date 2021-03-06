从用户点击一个URL链接开始解释HTTP的工作原理：假设URL是 `http://www.rbooks.top/mypage?userid=0`

1.浏览器会分析该URL

2.分析完之后浏览器向DNS请求解析 `www.rbooks.top`的IP地址

3.DNS将解析出的IP地址`47.102.119.234`返回给浏览器

4.浏览器与服务器建立TCP连接(默认是80端口)

5.建立TCP连接之后，浏览器开始请求文档 `GET /mypage?userid=0` 

6.服务器给出响应，将数据返回给浏览器

7.释放TCP连接

8.浏览器开始渲染服务器传过来的数据

HTTP的报文是怎么样的呢？真实的http请求如下：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/48.png?raw=true">

状态码分为5类：指明特定的请求是否被满足，如果没有满足，原因是什么：

| 状态码 | 含义       | 例子                             |
| ------ | ---------- | -------------------------------- |
| 1xx    | 通知信息   | 100=服务器正在处理客户请求       |
| 2xx    | 成功       | 200=请求成功                     |
| 3xx    | 重定向     | 301=页面改变了位置               |
| 4xx    | 客户错误   | 403=禁止的页面；404=页面未找到   |
| 5xx    | 服务器错误 | 500=服务器内部错误；503=以后再试 |



相关知识点：

[简述TCP连接的建立与释放](https://zhuanlan.zhihu.com/p/24860403)

TCP是面向连接的运输层协议，它是可靠交付，全双工的，面向字节流的点对点服务。HTTP协议便是基于TCP协议实现的。相比于网络层着力解决主机间的通讯，运输层则是着眼于应用进程之间的通信，所以我们常说的端口，套接字都是运输层上的概念。TCP协议的报文格式如下：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/45.png?raw=true">

seq（序号）：TCP连接字节流中每一个字节都会有一个编号，而本字段的值指的是**本报文段所发送数据部分的第一个字节的序号**

ack（确认号）：表示期望收到的下一个报文段数据部分的第一个字节的编号，编号为ack-1及以前的字节已经收到。

SYN ：当本自担为1时，表示这是一个连接请求或者连接接受报文

ACK：仅当本字段为1时，确认号有效

FIN：用来释放一个连接，当本字段为1时，表示此报文段的发送端数据已经发送完毕，要求释放运输连接。

讲完了基本的内容之后，重点来了。TCP运输连接分为三个阶段：连接建立，数据传送以及连接释放。运输连接管理就是连接建立以及连接释放过程的管控，使其能正常运行，达到这些目的：使通信双方能够通知对方的存在，可以允许通信双方协商一些参数（最大报文段长度，最大窗口大小等等），能够对运输实体资源进行分配（缓存大小等）。TCP连接的建立采用C-S模式：主动发起请求的叫Client，被动等待连接的应用进程叫做Server。

- 第一次握手：客户端的应用进程主栋打开，并向服务端发出请求报文段。其首部中：SYN=1，seq=x
- 第二次握手：服务器应用进程被打开。若同意客户端请求则发回确认报文，其首部中：SYN=1，ACK=1，ack=x+1,seq=y
- 第三次握手：客户端收到确认报文之后，通知上层应用进程连接已经建立，并向服务器发出确认报文，其首部：ACK=1，ack=y+1 。当服务器收到客户端的确认报文之后，也通知其上层应用进程连接已经建立

这个过程用图来表示如下：其中CLOSED：关闭状态，LISTEN：收听状态，SYN-SENT：同步已发送，SYN-RCVD：同步收到，ESTAB-LISHED：连接已建立

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/46.png?raw=true">

通俗的说就像两个人在打招呼：

A：你准备好了吗

B：我准备好了

A：好那我开始了。



释放阶段：

- 第一次挥手：数据传输结束以后，客户端的应用进程发出连接释放报文，并停止发送数据，其首部：FIN=1,seq=u
- 第二次挥手：服务器收到释放报文，发出确认报文。首部：ack=u+1,seq=v。此时本次连接就已经进入了半关闭状态。客户端不再像服务器发送数据。而服务器会继续发送
- 第三次挥手：若服务器已经没有要想客户端发送的数据，其应用进程就会通知服务器释放TCP连接。这个阶段服务器锁发出的最后一个报文的首部应为：FIN=1，ACK=1，seq=w，ack=u+1
- 第四次挥手：客户端收到释放报文后，必须发出确认：ACK=1，seq=u+1 ，ack=w+1 。再经过2MSL（最长报文端寿命）后，本次TCP连接真正结束。通信双方完成了他们的告别。

图示如下：ESTAB-LISHED：连接建立状态，FIN-WAIT-1：终止等待1状态，FIN-WAIT-2：终止等待2状态，CLOSE-WAIT：关闭等待状态，LAST-ACK：最后确认状态，TIME-WAIT：时间等待状态，CLOSED：关闭状态

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/47.png?raw=true">

通俗的说就是：两个人在辩论

A：一段时间之后，我已经讲完了，该你了 。此时A从连接建立状态到了终止等待1状态

B：在听了A的辩论之后，你真的讲完了？此时B开始组织语言，

等待一段时间没有响应之后，B开始讲他的部分 ，此时B处于关闭等待状态，等待它的话说完

B：讲了一段时间之后，我也讲完了，你还有什么补充的吗。此时B已经没有话说了，处于最后确认状态

A：我也没有了，你呢？A处于 时间等待状态

等待一段时间没有回应后，结束辩论



关于TCP为什么要这么设计的一些点：

1.在握手与挥手的过程中，往复的ack和seq有什么含义？

这是通信双方在通信过程的确认手段，确保双方通信的正确性。

2.在结束连接的过程中，为什么在收到服务器端的连接释放报文之后没有立刻结束连接，而是等待了2MSL之后才真正关闭。

这里由两个原因：

(1)，需要保证服务器收到了客户端的最后一条确认报文。假如这条报文丢失，无法做出响应，就造成了服务器不停重传连接释放报文，导致无法正常进入关闭状态。而等待2MSL，可以保证服务器收到最终确认报文；若服务器没有收到，在2MSL之内，客户端会收到服务器的重传报文，来询问(你还有没有要补充的？)，此时客户端会重新传输确认报文，并重置计时器

(2)，存在一种“已失效的连接请求报文”：当客户端发出连接请求报文，而服务器端没有收到，客户端会重新发出一条请求报文，成功建立连接。然而，第一条请求并没有丢失，而是在某个节点逗留时间过长，随后它也到达了服务器，它会让服务器以为客户端又发送了一条请求，从而造成异常。



[以下部分来自这篇文章](https://blog.csdn.net/hyg0811/article/details/102366854)

#### 关于TCP连接的一些问题

进行三次握手的主要作用就是为了确认双方的接收能力和发送能力是否正常，指定自己的初始化序列号为后面的可靠性传送做准备。其实就是**连接服务器指定端口，建立TCP连接，并同步两阶双方的序列号和确认号，交换TCP窗口大小 信息**。

为什么三次握手两次不行吗？

- 第一次握手：客户端发送网络包，服务端收到了。这样**服务端**就能够确认 **客户端的发送能力和服务端的接收能力是正常的**
- 第二次握手：服务端发送网络包，客户端接收。这样**客户端**就能确认 **服务端的发送能力和客户端的接收能力是正常的，**但是这时候服务端不知道客户端的接收能力是否正常以及自己的发送能力，所以需要第三次握手
- 第三次握手，客户端发送网络包，服务端接收。 服务端确认 客户端的接收能力正常

综合三次握手，客户端和服务端都能够得知双方的收发能力正常

|            | 客户端知道的                                           | 服务端知道的                                           |
| ---------- | ------------------------------------------------------ | ------------------------------------------------------ |
| 第一次握手 |                                                        | 客户端的发送能力，服务端的接收能力                     |
| 第二次握手 | 客户端的发送和接收，服务端的发送和接收                 | 客户端的发送能力，服务端的接收能力，                   |
| 第三次握手 | 客户端的发送能力和接收能力，服务端的接收能力和发送能力 | 客户端的发送能力和接收能力，服务端的接收能力和发送能力 |

每握一次收 对方都会知道 ：自己的接收能力和对方的发送能力正常。

**如果采用两次握手：**

客户端发出连接请求，但可能报文丢失未收到确认，就又发了一个请求得到回复之后就完成连接。数据传输完毕之后，释放连接。

这时候客户端一共发出两段报文，其中第一个走丢了可能是某个网络结点长时间滞留导致时间延误，这时候也到达了服务端，服务端就会认为这是一个新的连接，就向客户端发送确认报文之后 连接建立。

这时候客户端已经认为自己完成了任务，自然会忽略服务端的确认请求，导致服务端一直在等待客户端发送数据，浪费资源。

在建立连接的过程中，正在建立的连接会被服务器放到一个队列中，**半连接队列**。如果连接建立完毕自然在**全连接队列**。如果全连接队列满了，就有可能出现丢包的情况

关于SYN-ACK重传次数的问题：服务器发送完SYN-ACK包，如果未收到客户端确认包，服务器会进行首次重传。等待一段时间之后仍未收到确认包，进行第二次重传**。如果重传次数超过系统规定的最大重传次数，系统会将该连接信息从半连接队列中删除**

而且每次重传的间隔时间倍增 ：1s  2s  4s  8s ...



##### 三次握手可以携带数据吗

第三次可以，前两次不行。

如果第一次握手就可以携带数据，那么如果有人恶意攻击服务器，就会在第一次握手时 SYN报文中放入大量数据，攻击者根部不会理会服务器的连接发送能力是否正常，就疯狂的发送SYN报文，这回让服务器花费很多时间和内存空间来接收报文。

第三次可以是因为，**对于客户端来说已经知道了服务器的收发能力正常。**



##### 什么是SYN攻击

这一点就是利用了TCP连接的特性，当客户端没有发送第三次的确认报文时会不断重发SYN-ACK报文，直至超时。SYN攻击就是Client在短时间制造大量不存在的IP地址，利用这些IP向服务器不停的发送SYN包，Server回复确认包时因为IP都是假的自然没有回复，所以Server不断重发直至超时。

这些伪造的SYN包长期占用半连接队列，导致正常的SYN请求因为队列满了被丢弃，从而引起网络拥塞甚至系统瘫痪。SYN攻击是典型的DoS/DDoS攻击

检测SYN攻击很简单，当你看到服务器上有大量半连接状态，特别是源IP地址都是随机的，基本上可以断定是SYN攻击

如何防御?

- 缩短重传超时时间
- 增加最大半连接数
- 过滤网关防护
- SYN cookies技术

