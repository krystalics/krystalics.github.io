##### n个元素进栈，共有多少种出栈顺序？

[参考文章](https://blog.csdn.net/Zyearn/article/details/7758716)   

可以发现：

1个元素进栈就有1种出栈顺序。

2个元素进栈就有2种出栈顺序。 3个元素进栈就有5种出栈顺序，那么拓展到n种是什么情况呢？

f(1)=1  //1

f(2)=2  //12 21

f(3)=5  // 123 132 213 321 231

现在考虑 f(4) ，给4个元素编号为a，b，c，d 分别有1~4号位置。假设1号是栈顶

(1) 假设a元素在1号位置，那么只可能a进栈，马上出栈，此时还剩b，c，d等待操作就是 子问题f(3) 

(2) 如果a在2号位，那么一定有一个元素比a先出栈 ，即有f(1)种可能顺序，除a外还剩下两个元素是顺序未知的，那么就有f(2)种(因为a的位置已知所以不计算它)  总共就有 f(1)*f(2) 出栈顺序

(3) 如果a在3号位，同上  f(2)*f(1)

(4) 如果a在4号位 ，同上 f(3)



结合四种情况 f(4)=f(3)+f(1)*f(2)+f(2)*f(1)+f(3)  为了得到规律我们定义 f(0)=1  情况就变成了下面：

**f(4) = f(0)*f(3) + f(1)*f(2) + f(2) * f(1) + f(3)*f(0)**  由n=4推广到其他情况：

**f(n) = f(0)*f(n-1) + f(1)*f(n-2) + ... + f(n-1)*f(0)**   递推公式出来了，计算可不好计算？

实际上这种出栈顺序的种数的答案就是 **卡特兰数列** ， [wiki百科对于它的定义](https://en.wikipedia.org/wiki/Catalan_number) 

![{\ displaystyle C_ {n} = {\ frac {1} {n + 1}} {2n \ choose n} = {\ frac {(2n)！} {(n + 1)！\，n！}} = \ prod \ limits _ {k = 2} ^ {n} {\ frac {n + k} {k}} \ qquad {\ text {for}} n \ geq 0.}](https://wikimedia.org/api/rest_v1/media/math/render/svg/58374aa2b2e2c016a5b313e2bbd59940a2e1a5f9)

The first Catalan numbers for *n* = 0, 1, 2, 3, ... are  

1，1，2，5，14，42，132，429，1430，4862，16796，58786，208012，742900，2674440，9694845，35357670，129644790，477638700，1767263190，6564120420，24466267020

从这个数列中 很明显看出 f(4)=14  f(3)=5 f(2)=2 f(1)=1 f(0)=1   对于计算出的非整数部分直接将小书去掉， 如 C1=C0 (1+2)/2 =3/2=1  至于证明过程请恕我无能为力！



##### 对于Hash冲突的解决方法

1.拉链法  ： 就是在表头后面接上一个链表，用来装载该位置发生冲突的元素，动态申请空间  在Jdk8之后 HashMap和HashSet都是采用这种方法

2.开放定址法： 有线性探测，二次探查，等。就是在hash表中发生冲突时在冲突位置附近找一个空地址。

3.再散列：发生冲突后，继续散列直到不发生冲突为止。



##### volatile和synchronized 

synchronized是Lock和Condition接口的java内部实现，就是一个内部锁，如果一个方法用synchronized声明，那么对象的锁将保护整个方法。

```java
public synchronized void method(){
    
}
// 等价于
public void method(){
    this.intrinsiLock.lock(); //锁
    try{
        
    }
    finally{this.intrinsicLock.unlock();} //解锁
}
```

同样的它有自己的等待唤醒机制 ：`wait`方法添加一个线程到等待集中，`notifyAll/notify` 方法解除等待线程的阻塞状态，相当于  

```
intrinsicCondition.await();
intrinsicCondition.signalAll();
```

但是内部锁存在一些局限：

- 不能中断一个正在试图获得锁的线程
- 试图获得锁时不能设定超时
- 每个锁仅有单一的条件，这可能是不够的

在代码中应该如何使用这些呢“

- 最好不要使用Lock/Condition也不要使用synchronized关键字，在许多情况下可以使用java.util.concurrent包中的一种机制来处理所有的加锁。
- 如果synchronized适合你的程序，那么就不要使用Lock/Condition
- 如果特别需求Lock/Condition 结构提供的独有特性时才用它



volatile 关键字为实例域的**同步访问提供了一种免锁机制**，如果声明一个域为volatile，那么编译器和虚拟机就知道该域是可能被另一个线程并发更新。注意它**不能提供原子性** ,

`volatile`关键字解决的是内存可见性的问题，会使得所有对`volatile`变量的读写都会直接刷到主存，即保证了变量的可见性。这样就能满足一些对变量可见性有要求而对读取顺序没有要求的需求。

一个应用实例：在下列例子中done的值被一个线程查看确被另一个线程设置，当在查看时会对其加锁，设置线程就可能会进入阻塞。

```java
private boolean done;
public synchronized boolean isDone(){return done;}
public synchronized void setDone(){done=true;}  
```

而使用volatile 就可以解决这个问题

```java
private volatile boolean done;
public boolean isDone(){return done;}
public void setDone(){done=true;}  
```

**volatile和synchronized的区别**

- volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；
-  synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
- volatile仅能使用在变量级别；synchronized则可以使用在变量、方法、和类级别的
- volatile仅能实现变量的修改可见性，不能保证原子性；而synchronized则可以保证变量的修改可见性和原子性
- volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。

**volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化**



##### 前后端交互的时候：如何保证API调用数据时的安全性

[前后端API交互如何保证数据安全性？](https://juejin.im/post/5b149754f265da6e155d4748)

1. 通信使用https
2. 请求签名，防止参数被篡改
3. 身份确认机制，每次请求都要验证是否合法
4. APP中使用ssl pinning 防止抓包操作
5. 队友所有请求和响应都进行加密解密操作



##### 数据库中的视图

视图不包含数据。

视图是位于数据字典中的定义，并具有用于检索其数据的数据库查询。因此视图可以包含多个对象(这里的对象是 数据库中的 table)，行，或属性的数据。在从视图访存数据时，数据库会基于WHERE子句获取必要的记录，并返回数据。创建视图对象时，将装入属性。

视图会根据其所基于的表进行填充。例如，如果视图基于WORKORDER 表并选中了自动选中复选框，那么如果该表有 添加或删除 field，会影响到关联的视图。变更属性时，并非所有变更都会应用到相关联的视图。例如，如果变更field的数据类型，那么该变更会应用到视图。

但是如果变更WORKORDER表的缺省值或者对其添加域，那么此变更不应用于视图。

由于视图将存储已命名的查询，因此可以使用视图来存储常用的复杂查询。然后，可以通过在简单查询中使用视图名称来运行这些查询。



##### 设一棵完全二叉树共有699个结点，则在该二叉树中的叶子结点数为______。  B

A. 349 
B. 350 
C. 255 
D. 351 

解析：首先根据完全二叉树的性质计算出这棵树的层数。 c=log(n)+1 =log(699)+1  整数=10 ，其中log(n)是树的高度，二者之间相差1。

然后1到9层的节点总数=2^9-1 = 511 再 699-511=188 而最后一层有188个节点就说明上一层有94个非叶子节点，那188个节点就是继承自上一层的94个非叶子节点。 第九层本来有2^(9-1)=256个节点 第九层的叶子节点个数= 256-94=162    所以总的叶子节点数=188+162=350 个。



##### 直接广播地址：

**有限广播**　　它不被路由，但会被送到相同[物理网络](http://baike.baidu.com/view/1913655.htm)段上的所有主机，[IP地址](http://baike.baidu.com/view/3930.htm)的**网络字段**和**主机字段**全为1就是地址**255.255.255.255**　　

**直接广播**　　网络广播会被路由，并会发送到专门网络上的每台主机　　IP[地址](http://baike.baidu.com/view/494802.htm)的网络字段定义这个网络，**主机字段**通常全为1，如 192.168.10.**255**



##### 广义表的表头表尾

当广义表LS非空时，称第一个元素为LS的表头，称广义表LS中除去表头后其余元素组成的广义表为LS的表尾：例如  (a,(b)) 的表头是 单元素a  表尾是 广义表 ((b) 注意层次

再例如：  ((a,b),c,d)  的表头是 (a,b)  表尾是  (c,d)



##### 进程间通信的7种方式

[进程间通信IPC](https://www.jianshu.com/p/c1015f5ffa74)

1. 管道(匿名管道pipe)，半双工，数据只能朝一个方向流动；双向通信时需要两个管道
2. 有名管道 FIFO
3. 信号(Signal)
4. 消息队列
5. 共享内存
6. 信号量
7. 套接字



##### 遍历HashMap的方式

因为做题目的时候老是碰到可能使用到的情况，老是不会，这次痛定思痛，先背下来。

```java
HashMap<Integer,Integer> map=new HashMap<>();
Iterator iter=map.entrySet().iterator();
while(iter.hasNext()){
    Map.Entry entry=(Map.Entry) iter.next();
    int key=(Integer)entry.getKey();
    int val=(Integer)entry.getValue();
}
```



##### 多个数的最大公约数

[最大公约数GCD](https://www.jianshu.com/p/25d0ca88a4a1)

应用最广泛的就是辗转相除法，该方法依托于一个原理：

```
gcd(a,b)=gcd(b,a mod b)
```

即a,b的最大公约数就是b,a mod b的最大公约数。

证明如下：

1.充分性

设 g是 a,b的公约数，则a,b可由g来表示  

​	a=xg  b=yg （x,y都是整数）

又，a可以由b表示（任意一个数都可以由另一个数来表示）：

​	a=kb+r （k为整数，r=a%b）  r=a-kb=xg-kyg=(x-ky)g

即 g也是r的约数 ，即g同时是 b 和 r的约数 。这样 g 就是(b,r)即(b,a mod b)的公约数

2.必要性：

设 g 是 (b,a mod b)的公约数，类似于 1 ,同样可以推出g也是(a,b)的公约数。



好了，有了理论基础、下面总结成算法步骤：

1. a%b=r
2. 若 r==0 则 b 即为 GCD
3. 若 r!=0 ，则 a=b , b=r 返回步骤1

```java
public int gcd_recursive(int a,int b){
    if(b==0){
        return a;
    }
    return gcd_recursive(b,a%b);
}

public int gcd_iteration(int a,int b){
    while(b!=0){
		int t=b;
        b=a%b;
        a=b;
    }
    return a;
}
```



[多个数的最大公约数](http://www.voidcn.com/article/p-gkmtgvkr-hd.html)

多个数的辗转相除法：

取最小的数a，然后用剩下的数除以a，**得到的余数和a组成新数组**，然后对新数组进行同样的动作；当出现零的时候，排除零，对其余的数进行相同的动作。直到最后只剩下一个非零数，这个数就是原数组的最大公约数。

例题：求(120,168,328,624,320) 的最大公约数

　　(120,168,328,624,320)

　　=(120,48,88,24,80)(因为168除以120余**48**,328除以120余**88**,……)  然后再选择这个序列中最小的数24，然后重复

　　=(0,0,16,0,8) 

　　=(0,0,0,0,8) 

　　=8

```java
public int multi_gcd(int a[]){
    int count=a.lentgh; //代表着这个序列中 非0数的个数
	while(a.lentgh>1){
    //先找到该序列中的最小数及其 下标
        int min=a[0];
        int index=0;
        for(int i=0;i<a.length;i++){
            if(min>a[i]){
                min=a[i];
                index=i;
            }
        }

        fro(int i=0;i<a.length;i++){
            if(i!=index)
                a[i]=a[i]%a[index];
            if(a[i]==0) remove(a);  
        }
	}
    return a[0]; //因为序列最后就剩下一个数了。
}


public void remove(int []a){
    for(int i=0;i<a.length;i++){
        if(a[i]==0){
            for(int j=i;j<a.length-1;j++){
                a[j]=a[j+1];
            }
        }
    }
}

```



##### TCP连接的建立与关闭过程

> TCP连接的建立与关闭过程可以概括为“三次握手，四次挥手”。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/9.png?raw=true">

**第一次握手**：建立连接时，客户端到服务器，并进入SYN_SENT 状态，等待服务器确认；SYN：同步序列编号(Synchronize Sequence Numbers) 。这里注意：请求建立连接的一般都是客户端程序，而服务器空闲时处于LISTEN状态。  

**第二次握手**：服务器收到syn包，必须确认客户的SYN (ack=j+1) ，同时自己也发送一个SYN包 (syn=k)  即SYN+ACK包，此时服务器进入SYN_RECV状态；

**第三次握手**：客户端收到服务器的SYN+ACK包，想服务器发送确认包 ACK(ack=k+1) ，此包发送完毕，客户端和服务器进入 ESTABLISHED (TCP连接成功)状态，完成三次握手。

具体的状态码如下：该报文段，指明了客户端即将需要连接的 服务器上的 具体端口、初始序号x（保存在包头的Sequence Number字段）



<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/10.png?raw=true">

**第一次挥手**：主动关闭方发送一个FIN进入 FIN_WAIT_1 ， 用来关闭主动方到被动关闭方的数据传送，也就是主动关闭方告诉被动关闭方：我已经不会再给你发数据(在fin包之前发送出去的数据，如果没有收到对应的ack确认报文，主动关闭方依然会重发过这些数据) ，但是，此时主动关闭方还可以接受数据。

**第二次挥手**：被动关闭方收到FIN包后，发送一个ACK给对方，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号）

**第三次挥手**：被动关闭方发送一个FIN，用来关闭被动关闭到主动关闭方的数据传送，也就是告诉主动关闭方，我的数据也发送完了，不会再给你发数据了。

**第四次挥手**：主动关闭方收到FIN后，发送一个ACK给被动关闭方，确认序号为收到序号+1，至此，完成四次挥手。











































