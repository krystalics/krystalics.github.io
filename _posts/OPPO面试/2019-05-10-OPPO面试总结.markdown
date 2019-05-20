虽然短短20分钟，但是问了很多方面的问题，有的答上来了，有的没听过，有的不深入。。。总之感觉凉了。但是给我的启发还是大的，并没有问什么框架的东西，首先问了我对jdk中的哪个包比较熟悉(IO，网络包，集合类等)，然后就四处出击，*感觉根本没看我的简历*。直接问我JVM，堆的具体划分，GC算法，线程池，数据库，MySQL多个表join查询时的索引，聚簇索引，还有隔离级别等等，下面将对面试过程中的问题进行分析。

---

##### 1.hashmap的数据结构，扩容的情况

底层结构是散列表。当不同的键值产生的hashcode相同时，就会将其放在一个桶(bucket，一个横行称为一个桶)中，这也是哈希表应付冲突的办法之一 ： 拉链法

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/13.png?raw=true">

**寻址方式：**HashMap将Key的哈希值对数组长度取模，结果作为该Entry在数组的index。在计算机中**取模的代价远高于位运算**，因此HashMap要求数组的长度必须为2的N次方（它默认的长度=16，而且扩容也都是两倍）。此时将Key的哈希值对2^N -1进行运算，其效果等同于取模。

HashMap并不要求用户在指定HashMap容量时必须传入2的N次方整数，而是通过其他方法对传入的容量进行校正，至2的N次方倍。

当HashMap的size超过Capacity*loadFactor时，需要对HashMap进行扩容。具体方法是创建一个新的，长度为原来两倍的数组，保证Capacity仍为2的N次方，同时需要将原来的数据重新插入到新的HashMap中 (rehash过程)



##### 2.ConcurrentHashMap与hashtable的区别(二者都是线程同步)

本段转发自[**技术世界**](http://www.jasongj.com/)，[原文链接](http://www.jasongj.com/java/concurrenthashmap/)　<http://www.jasongj.com/java/concurrenthashmap/>

​	Hashtable是对整个对象加锁，当Hashtable的大小增加到一定的时候，性能急剧下降，因为迭代时需要被锁定很长时间。

​	**jdk1.7**的时候ConcurrentHashMap引入Segment，，把一个大的HashMap拆分成N个小的HashMap

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/14.png?raw=true">

**寻址方式**：在读写某个Key时，先取该Key的hash值，并将该哈希值的高N位对Segment个数取模从而得到Key应该属于哪个Segment，接着如同操作HashMap一样操作这个Segment。

**同步方式**：Segment继承了[ReentrantLock](https://blog.csdn.net/lipeng_bigdata/article/details/52154637)，很方便对每个Segment上锁。

**对于读操作**，获取Key所在的Segment时，需要保证可见性(Valotile：只能修饰变量，可以将被其修饰的变量声明CPU缓存中的值无效，只能到主存中去找，保证该变量更改时可以被其他线程知道)

**对于写操作**，并不要求同时获取所有Segment的锁，如果那样就相当于锁住了整个Map。它会先获取Key-Value对所在的Segment的锁，获取成功之后就可以像操作一个普通HashMap一样操作该Segment，并保证Segment的安全性。

同时，由于其他Segment的锁并未被获取，因此理论上可以支持ConcurrencyLevel(等于Segment的个数)个线程安全的并发读写。

获取锁时，并不直接使用lock来获取，因为该方法获取锁失败时会挂起（参考[可重入锁](http://www.jasongj.com/java/multi_thread/#%E9%87%8D%E5%85%A5%E9%94%81)），它使用了自旋锁，如果tryLock获取锁失败，说明锁被其他线程占用，此时通过循环再次以tryLock的方式申请锁。如果在循环过程中该Key所对应的链表头被修改，则重置retry次数。如果retry次数超过一定值，则使用lock方法申请锁。这里使用自旋锁是因为自旋锁的效率比较高，但是它消耗CPU资源比较多，因此在自旋次数超过阈值时切换为互斥锁。

**size操作**：put，remove和get操作只需要关心一个Segment，而size操作需要**遍历所有的Segment才能算出整个Map的大小**。简单方案是先锁住整个Map，计算完之后再解锁(毕竟可能计算size的时候，有其他线程对它进行增加或删除)。但是如果锁住的话，其他线程无法对它进行操作，不利于并行。

为了更好支持并发操作，ConcurrentHashMap会在**不上锁的前提逐个Segment计算三次size**，如果某相邻两次计算获取的所有Segment的更新次数(每个Segment都与HashMap一样通过modCount跟踪自己的修改次数，Segment每修改一次其modCount+1)相等，说明两次计算过程中并无更新操作，则这两次计算出的总size相等，则直接作为最终结果返回。**如果这三次计算过程中Map有更新，则对所有Segment加锁重新计算Size**



##### 基于JDK1.8 CAS(Compare and Swap)的ConcurrentHashMap

Java 7位实现并行访问，引入了Segment这一结构，实现了分段锁，理论上最大并发度与Segment个数相同。Java 8为了进一步提高并发性，摒弃了分段锁的方案，而是**直接使用一个大的数组**。同时为了提高哈希碰撞下的寻址性能，Java 8在链表长度超过一定阈值(默认是8)时将链表(寻址时间复杂度为O(N))转为红黑树(寻址时间复杂度为O(log(N)))。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/15.png?raw=true">

**寻址方式**：Java 8的ConcurrentHashMap同样是通过Key的哈希值与数组长度取模来确定Key在数组中的索引。因为引入了红黑树，所以即使哈希冲突比较严重，寻址效率也足够高，并没有在哈希值的计算上做过多的设计。只是将Key的hashCode值与其高16位作异或并保证最高位为0(保证结果为正整数)

```java
static final int HASH_BITS = 0x7fffffff
static final int spread(int h){
    return (h^(h>>>16))&HASH_BITS;
}
```

**同步方式**：

**对于put操作**，如果Key对应的数组元素为null，则通过[CAS操作](<https://juejin.im/post/5a73cbbff265da4e807783f5>)将其设置为当前值。如果Key对应的数组元素(即链表表头或者树的根元素)不为null，则**对该元素使用synchronized关键字申请锁**，然后进行操作。如果put操作使得当前链表长度超过一定阈值，则该链表转换为树，用以提高寻址效率。

**对于读操作**，由于数组被volatile关键字修饰，因此不用担心数组可见性的问题。同时每个元素是一个Node实例(Java 7中每个元素是HashEntry)，它的Key值和hash值都有final修饰，不可变更，无须关心他们被修改之后的可见性。

```java
static class Node<K,V> implements Map.Entry<K,V>{
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;
}
```

对于Key对应的数组元素的可见性，有Unsafe.getObjectVolatile()保证

```java
static final <K,V> Node<K,V> tabAt(Node<K,V> []tab,int i){
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);

}
```

**size操作：**put方法和remove方法都会通过addCount方法维护Map的size，size方法通过sumCount获取size



##### 3.线程的生命周期，sleep()和wait()的区别，java线程和操作系统的线程之间的区别

[Java中sleep()与wait()](https://blog.csdn.net/u012050154/article/details/50903326)

[Java与线程](https://www.imooc.com/article/257138) 该文章说是转载联系作者，但是打开原文链接却是404。 

[java线程是用户级的吗](https://www.zhihu.com/question/23096638)

生命周期：new(新建)，Runnable(可运行)，Waiting(等待)，Timed Waiting(计时等待)，Blocked(阻塞)，Terminated(终止)。

当用**new**操作创建一个线程时，处于**new**状态：这时，线程处于刚刚创建的状态，程序还没有开始运行线程中的代码，在线程运行之前还有一些基础工作要做。

一旦调用**start**方法，线程就处于**Runnable**状态：一个可运行的线程可能在运行也可能没有运行，这取决于操作系统给线程提供的时间。

当线程处于**Blocked**或者**Waiting**状态时，它暂时不活动：不允许任何代码且消耗最少资源，直到线程调度器重新激活它。如何到达这两种状态呢？

- 当一个线程试图获取一个内部的对象锁(不是`java.util.concurrent`库中的锁)，而该锁被其他线程持有，则该线程进入**Blocked**状态。当本线程持有锁的时候，就是非阻塞状态
- 当一个线程等待另一个线程通知调度器一个条件时，它进入**Waiting**状态。在调用Object.wait()或者Thread.join()方法，或者是等待 `java.util.concurrent`库中Lock或Condition时，就会处于这个状态。实际上，被阻塞与等待状态有很大不同
- 有几个方法有一个**超时参数**，调用它们导致线程进入**Timed Waiting**状态。这个状态一直持续到超时满期或者接收到适当的通知。

**只有两种情况**：因为run()方法**正常退出而自然死亡**，另一个是因为一个**没有捕获的异常**从而终止了run()方法意外死亡。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/37.png?raw=true">



sleep()：是正在执行的线程主动让出CPU，并不释放资源，过了指定时间之后继续执行。**可以在任何地方使用**

wait()：让当前线程退出同步资源锁，以便其他正在等待该资源的线程能够获取，只有调用了notify()方法才能够让它恢复过来，去参与竞争同步资源锁。**只能在同步块或同步方法中使用**



**java与线程：**

线程是CPU调度的基本单位，Thread类与大部分Java API都有显著区别，它的所有关键方法都是Native的(Native是用于和其他语言交互的，例如C++，使用这个关键字意味着这个方法和程序运行的平台有关，比如操作系统的内核，调度算法等等) 。大约分为**三种线程**：

**内核线程(Kernel-Level Thread,KLT)**

很明显这是由操作系统内核直接支持的线程。由内核来完成线程切换，内核通过操纵调度器Sheduler对线程进行调度，并负责将线程的任务映射到各个CPU上。每个内核线程都可以视为内核的一个分身，这样OS就有能力同时处理多件事情，支持多线程的内核就叫多线程内核(Multi-Threads Kernel) 

我们的Java程序一般不会直接去使用KLT，转而使用KLT的接口——轻量进程(Light Weight Process,LWP)即我们通常意义上的线程。由于每个LWP都有一个KLT支持，因此只有先支持KLT才能有LWP。**这种1:1的关系称为一对一的线程模型。**

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/16.jpg?raw=true">

也由于是基于KLT实现的，所以各种线程操作，如创建，构析及同步都需要进行**系统调用**。而系统调用的代价相对较高，需要在**用户态和内核态中来回切换**。其次，每个LWP都需要有一个KLT支持，因此LWP要消耗一定内核资源(比如KLT的栈空间)，因此一个系统支持的**LWP的数量是有限的**。



**用户线程**由于创建，切换，调度各种细节都需要考虑，实现起来极其困难。已经被Java，Ruby等语言放弃了。



**用户线程混合LWP**

用户线程完全建立在用户空间(在内存中除了操作系统所处的内核空间外的空间叫做用户空间)。因此用户线程的创建，切换，析构等操作依然廉价并且可以支持大规模的用户线程并发。操作系统提供支持的LWP则作为用户线程和内核线程之间的桥梁。这样可以使用内核提供的线程调度功能及CPU映射，并且用户线程的系统调用要通过LWP来完成，大大降低了整个进程被完全阻塞的风险。

在这种混合模式中，用户线程与LWP的数量比不确定，即为N:M的关系。许多Unix系统如Solaris，HP-UX等都提供了N:M的线程模型实现。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/17.jpg?raw=true">



**Java线程**

操作系统支持怎样的线程模型，在很大程度上决定了JVM的线程是怎样映射的，这一点在不同的平台上没有办法达成一致，**JVM中也没有规范Java线程需要使用哪个线程模型来实现。**线程模型只对线程的并发规模和操作成本产生影响，对Java程序的编码和运行过程都是透明的。

对于Sun公司的JDK来说，它的Windows版与Linux版都是使用1:1的线程模型实现。一条Java线程映射一条LWP，因为Windows和Linux系统提供的线程模型就是1对1的。而Solaris平台中，由于操作系统的线程特性可以同时支持一对一以及多对多，所以Solaris的JDK对应的提供了两个平台专有的参数来明确指定使用哪种线程模型。



##### 4.Hibernate的缓存

[Hibernate缓存机制](https://www.jianshu.com/p/4ebf4b3b8e96)  [Hibernate的缓存](https://juejin.im/post/5b39c8ccf265da59a4020374)

Hibernate缓存包括两大类：Hibernate一级缓存（存放在栈中）和二级缓存（存放在堆中）。

一级缓存又称为Session缓存：Session内置不能被卸载，Session的缓存是事务范围的缓存(Session的生命周期对应一个数据库事务或者一个应用事务)。一级缓存中，持久化类的每个实例都有唯一的OID。

当获取Session对象时，就可以使用一级缓存了。 *每个SpringBoot的版本不一样，代码也会发生变化*

```java
@PersistenceContext
private EntityManager entityManager;
---
HibernateEntityManager hem=(HibernateEntityManager)entityManager;
Session session=hem.getSession();
```

缓存状态分为三种：

**瞬时状态**：创建一个POJO，还未将对象数据保存到数据库中时，Session中也没有当前POJO实例。

```java
Account account=new Account();
account.setPhone("123");  //这时实例并没有保存
```

**持久状态**：POJO对象被添加到Session缓存中，数据库中也有对应数据

```java
public int saveStatus(Account account){
    account.save(); //保存到数据库中
    Session session=....getSession();
	int id=session.save(account);
    account.setPhone("23213"); //修改被持久化的POJO
    return id; //返回对象id
}

```

注意上述代码并没有保存到数据库中的代码，这里会执行两条SQL语句，一条添加SQL，另一条修改SQL （Hibernate被持久化的POJO对象在被重新赋值时会触发更新操作）

**脱管状态**：在缓存中已经被持久化的POJO对象，对它执行了evict方法，将POJO对象从缓存中剥离出来，但是数据库中还有数据。



**二级缓存又称SessionFactory的缓存**：由于SessionFactory对象的生命周期和引用程序整个过程对应，因此Hibernate二级缓存是进程范围或者集群范围的缓存，有可能出现并发问题，因此需要采用适当的并发访问策略，该策略为被缓存的数据提供了事务隔离级别。 二级缓存是可选的，可以不配置它，默认情况下是关闭的。配置需要一定的流程，这里就不详细讲了。

获取代码如下：*每个SpringBoot的版本不一样，代码也会发生变化*

```java
@Autowired
private EntityManagerFactory entityManagerFactory;
---
SessionFactory sf=entityManagerFactory.unwrap(SessionFactory.class);
```



适用于：

- 很少被修改的数据
- 不是很很重要的数据，允许偶尔出现并发的数据
- 不会被并发访问的数据
- 常量数据



Hibernate的查询分为两类：1.得到当个对象，2.得到一个结果集

单个对象：

get()和load()方法的区别在于对二级缓存的使用上。load()方法会使用二级缓存，get()方法在一级缓存没有找到的情况下会直接查询数据库连接，不会去二级缓存中找。

```java
Employee em1=session.load(Employee.Class,9);
Employee em2=session.get(Employee.Class,9);
```

结果集对象：

```java
Query query = session.createQuery("from Customers");
List<Customers> list=query.list();
Iterator<Customers> iter=query.iterate();
```

`list()`：无论有没有缓存，都会执行一条SQL语句，从数据库中获取记录

```java
// SELECT *FROM EMP
List<Employee> list1 = session.createQuery("from Employee").list(); 
for (Employee e : list1) {
    System.out.println(e);
}
// SELECT *FROM EMP
List<Employee> list2 = session.createQuery("from Employee").list();
for (Employee e : list2) {
    System.out.println(e);
}
```

`iterate()`：执行N+1条SQL查询。第一个查询只返回所有记录的主键，并且每当迭代返回迭代器时，就执行包含WHERE子句的单独SQL查询。如果记录存在于缓存中，则执行第一个查询，不执行后面N个，直接从缓存中获取记录。

```java
// SELECT EMP_ID FROM EMP
Iterator<Employee> iterator1 = session.createQuery("from Employee").iterate(); 
while(iterator1.hasNext()) {
    // SELECT * FROM EMP WHERE EMP_ID=?
    System.out.println(iterator1.next()); 
}

// SELECT EMP_ID FROM EMP
Iterator<Employee> iterator2 = session.createQuery("from Employee").iterate(); 
while (iterator2.hasNext()) {
    // From cache, no SQL
    System.out.println(iterator2.next()); 
}
```



Hibernate根据ID访问数据对象的时候，首先从Session一级缓存中查找；查不到，如果配置了二级缓存就从二级缓存中查。如果都查不到，在查询数据库，把结果按照ID放入到缓存删除，更新，增加数据的时候，同时更新缓存。



##### 5.JVM的结构划分

[Java新生代，老年代..](https://gblog.sherlocky.com/java-xin-sheng-dai-lao-nian-dai/)

[JVM内存结构](https://www.cnblogs.com/ityouknow/p/5610232.html)

[JVM内存结构和Java内存模型](https://zhuanlan.zhihu.com/p/38348646)

[Java对象在JVM中的内存分配](https://www.jianshu.com/p/2f295b9f4cd4)

JVM的内存结构大致如下： 从一个更高的维度看JVM和系统调用之间的关系

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/19.png?raw=true">

- 堆(Heap)：线程共享。所有对象实例以及数组都要在堆上分配。回收器主要管理的对象
- 方法区(Method Area)：线程共享。存储类信息，常量，静态变量，即时编译器JIT编译的代码
- 虚拟机栈(JVM Stack)：线程私有。存储局部变量表，操作栈，动态链接，方法出口，对象指针
- 本地方法栈(Native Method Stack)：线程私有。为JVM使用的Native方法服务。如Java使用C++编写接口服务时，代码在此区运行
- 程序计数器(Program Counter Register)：线程私有。也叫PC寄存器，它可以看作是当前线程所执行的字节码的行号指示器。指向下一条要执行的指令。

下图是具体的内存分布和控制参数：图片来自网络

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/18.png?raw=true">

**控制参数：**

- -Xms 设置堆的最小空间大小
- -Xmx 设置堆的最大空间大小
- -XX:NewSize 设置新生代最小空间大小
- -XX:MaxNewSize 设置新生代最大空间大小
- -XX:PermSize 设置永久代最小空间大小
- -XX:MaxPermSize 设置永久代最大空间大小
- -Xss 设置每个线程的堆栈大小
- 老年代空间大小=堆空间大小-年轻代空间大小

**堆**

对于大多数应用来说，Java堆是Java虚拟机所创建的内存中最大的一块。被所有线程共享，且在虚拟机启动的时候就创建，堆的作用是存放对象实例和数组。

它还是垃圾收集器管理的主要区域，因此很多时候也被称作GC堆。如果从内存回收的角度看，由于现代的**收集器都是采用分代收集算法**，所以又将其细分：分为**新生代**和**老年代**。新生代又可以分为**Eden**空间，**From Survivor** 空间 (s0)，**To Survivor** 空间 (s1)。默认情况下按照 8:1:1的比例分配空间大小。

根据JVM规范，堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可，就像我们的磁盘空间一样。在实现时既可以是固定大小，也可以是可扩展的，不过当前的主流虚拟机都是按照可扩展来实现的(通过-Xmx,-Xms来控制)  。如果在堆中没有足够的内存来完成实例初始化，并且堆也无法再扩展了，将会抛出`OutOfMemoryError`异常。

**所有新生成的对象首先都是放在新生代**。而且应用程序只使用**eden**和其中一个**Survivor**，当初级垃圾回收时，Minor GC挂起程序，**将eden和Survivor中的存活对象**复制到另一个**非活动的Survivor**中，然后一次性清除eden和Survivor，**将原来的非活动Survivor标记成活动Survivor**。将**指定次数回收后仍然存在的对象移动到老年代中**，初级回收之后得到一个空的可用的Eden。



**方法区**

和堆一样，是各个线程共享的内存区域。虽然JVM规范把方法区描述为堆的一个逻辑部分，但它也被叫做Non-heap 非堆，目的应该是与堆区分开来。

对于习惯在HotSpot虚拟机上开发和部署的开发者来说，很多人也把方法区叫做**永久代(Permanent Generation)**，本质上两者并不等价，仅仅是因为HotSpot虚拟机的设计团队选择把GC分代收集扩展至方法区，或者说使用永久代来实现方法区而已。

**持久代在Java 8中被彻底删除，取代它的叫做 元空间**

在**这个区域还可以不实现垃圾回收**。但是并不是说只要数据进入方法区就真的如永久代的名字一样永久存在，这个区域的**内存回收目标主要是常量池的回收和对类型的卸载**，一般来说这个区域的回收效果并不是很好，尤其是类型的卸载条件相当苛刻，但是这部分的回收确实有必要。



**程序计数器**

由于JVM的多线程是通过线程轮流切换并分配CPU执行时间的方式实现的，在任何确定时刻一个CPU(相当于多核处理器的一个内核)只会执行一条线程中的指令。因此为了线程切换后能够恢复到正确执行的位置，每条线程都需要一个独立的程序计数器，各条线程之间的计数器互不影响，独立存储。 

如果线程正在执行Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的位置；如果是Native方法，计数器值为空(Undefined) 次内存区域是JVM规范中唯一一个没有规定任何`OutOfMemoryError`情况的区域。



说完理论部分，让我们来看一个例子：

```java
public class Student {

    private String name;
    private static Birthday birthday = new Birthday();

    public Student(String name) {
        this.name = name;
    }
    
    class Birthday {
        private int year = 2010;
        private int month = 10;
        private int day = 1;
    }

    public static void main(String[] args) {
        Student s = new Student("zhangsan");
        int age = 10;
        System.out.println(age);
    }
}

```

特别说明下：**JVM并不会知道一个类是否为内部类，在Java编译器将Java源码编译到Class文件的过程中，将内部类做了“解糖” ，给其添加了一些必要的转换之后提升为和顶层类一样的形式。**所以下图中的birthday是在堆中的一块内存。

以Student类执行main方法的最后一行来分析该对象在内存中的分配情况

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/33.png?raw=true">

图不是很准确，但是能大致说明问题。

- 局部变量：在虚拟机栈中、基本类型的值直接存储在栈中，如age=10，如果是一个对象，只存储对象的引用如s=ref
- 实例变量：存放在堆中，如Student类中的实例 name=ref
- 静态变量：存放在方法区中，如Student.class中的，birthday=ref



##### 6.MySQL隔离级别，join查询时采用的索引，聚簇索引

MySQL隔离级别: 参考自高性能MySQL 第三版

在SQL标准中定义了四种隔离级别，每一种级别都规定了一个事务汇总所做的修改，哪些在事务内和事务间是可见的，哪些是不可见的。较低级别的隔离通常可以执行更高的并发，系统开销也更低。

- Read Uncommitted 未提交读：事务中的修改即使为提交，其他事务也都是可见的。**其他事务可以读取未提交的数据，这也叫做脏读**。这个级别会导致很多问题，而且性能也不必其他级别好太多，所以一般不用这个级别
- Read  Committed 提交读：大多数数据库的默认隔离级别Read Committed，但是MySQL不是。Read Committed 满足前面隔离性的简单定义：一个事务开始时，只能看见已经提交的事务所做的修改。或者说所做的任何修改在未提交时对其他事务不可见。**也叫做 不可重复读(nonrepeatable read) ，因为两次执行同样的查询可能得到不同结果**
- Repeatable Read 可重复读：它解决了脏读问题和不可重复读问题，**该级别保证了在同一事务中多次读取同样记录的结果是一致的**。但是理论上，可重复读级别还是无法解决幻读问题。所谓幻读：当某个事务在读取某个范围内的记录时，另一个事务又在该范围内插入新的记录，当之前的事务再次读取该范围的记录时，会产生幻行 。InnoDB和XtraDB存储引擎通过多版本并发控制(MVCC,Multiversion Concurrency Control)解决了幻读问题，它是MySQL的默认事务隔离级别
- Serializable 可串行化：是最高隔离级别，他通过强制事务串行，避免了前面说的幻读问题。在每一行记录上都加上锁，所以会导致大量超时和锁争用的问题。实际上，很少会用到这个级别，只有在非常需要确保数据一致性而且可以接受没有并发的情况下，才可以考虑该级别。

[对聚簇索引和非聚簇索引的认识](https://blog.csdn.net/alexdamiao/article/details/51934917)

聚簇索引是对磁盘上实际数据重新组织以按指定的一个或多个列的值的排序算法。特点是存储数据的顺序和索引顺序一致。一般情况下主键会默认创建聚簇索引，且一张表只允许存在一个聚簇索引。 索引本身的数据结构是B树，

聚簇索引的叶子就是数据节点，而**非聚簇索引的叶子节点只存放数据的指针，而不是数据本身**。MySQL中不同数据存储引擎对聚簇索引的支持不同，MyISAM是采用非聚簇索引。



MyISAM引擎的存储方式

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/20.png?raw=true">

MyISAM是按列值与行号来组织索引的。它的叶子节点中保存的实际上是指向存放数据的物理块的指针。从MyISAM存储的物理文件我们看得出，MyISAM引擎的索引文件(.myi)和数据文件(.myd)是相互独立的。

它的二级索引：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/21.png?raw=true">

而InnoDB按聚簇索引的形式存储数据，所以它的数据布局有着很大的不同。它存储数据的结果大致如下。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/22.png?raw=true">

注：聚簇索引中的每个叶子节点包含主键值，事务ID，回滚指针(rollback pointer 用于事务和MVCC)，除了主键外的其他 列。

InnoDB的二级索引与主键索引有很大的不同。InnoDB的二级索引的叶子包含主键值，而不是行指针(row pointers) ，这减小了移动数据或者数据页面分裂时维护二级索引的开销，因为InnoDB不需要更新索引的行指针。其结构大致如下：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/23.png?raw=true">



InnoDB的二级索引的叶子节点存放的是**查询的Key字段加主键值**。因此，通过二级索引查询首先查到主键值，然后InnoDB再根据查到的主键值通过主键索引找到相应的数据块。

而MyISAM的二级索引叶子节点存放的还是列值与行号的组合，叶子节点中保存的是数据的物理地址。所以可以看出**MyISAM的主键索引和二级索引没有任何区别，**主键索引仅仅只是一个叫做Primary的唯一，非空的索引，且MyISAM引擎中可以不设主键。



例子

```sql
select * from table1 where name='Bob'; 
```

首先通过二级索引树找到 Bob 的对应主键15，然后再去主键索引(聚簇索引)中找id=15的数据。

根据数据显示：

1.聚簇索引的查询效率比非聚簇索引的查询效率更高。所以尽量使用主键进行查找。

2.主键定义的长度越小，二级索引的大小就越小，这样每个磁盘块存储的索引数据越多，查询效率越高



##### 7.线程池

[jdk1.8下的线程池](https://juejin.im/post/5bac78ce6fb9a05cee1ded19)

[Java中线程池，你真的会用吗？](https://www.hollischuang.com/archives/2888)

[Java线程池的实现原理](https://mp.weixin.qq.com/s/-89-CcDnSLBYy3THmcLEdQ)

程序的运行，本质上是对系统资源(CPU，内存，磁盘，网络等等)的使用。如何高效的使用这些资源呢？这里说的**线程池就是对CPU利用的优化手段。**

>先来了解一个名词：池化技术。
>
>简单点说，就是提前保存大量的资源，以备不时之需。在机器资源有限的情况下使用池化技术可以大大提高资源利用率，提高性能。
>
>典型的有：线程池，连接池，内存池，对象池等



一般我们在并发编程中，使用到线程都是要**手动创建**->**执行任务**->**销毁线程**  这个过程会造成很大的性能开销。如果该**线程执行完任务之后又去执行另一个任务而不是销毁**就可以大大节约开销。这就是通常意义的线程池。

**线程池：预先创建好多个线程，放在池中，这样可以在需要使用线程时直接获取，避免多次重复创建和销毁带来的开销。**具体到Java中就是： 通过BlockingQueue阻塞队列，存放Runnable对象，检查线程队列中的线程，存在线程空闲且未被销毁，继续使用此线程执行任务。

在Java中创建线程池：

```java
import java.util.concurrent.*;

public class App{
    public static void main(String[]args) throws Exception{
        ExecutorService es=new ThreadPoolExecutor(10, //核心线程数
                                                  10,  //最大线程数量
       /*非核心线程可以空闲的时间，和下面的TimeUnit组成,这里的意思是60秒*/
                                                  60L,
                                                  TimeUnit.SECONDS,
   /*阻塞队列容量为 10*/          new ArrayBlockingQueue<>(10));
      
        //添加任务
        es.execute(new Runnable(){
            @Override
            public void run(){
                System.println("abce");
            }
        });
        es.shutdown();
    }
}
```

JDK提供给外部的接口也很简单，直接调用`ThreadPoolExecutor`构造一个就好了。也可以通过Executors静态工厂构建，但是一般不建议用这个，因为Executors创建的线程池存在OOM风险。其主要原因是 Executors的任务队列底层是`LinkedBlockingQueue`实现的，

Java中的`BlockingQueue`主要有两种实现，分别是`ArrayBlockingQueue`和`LinkedBlockingQueue`。而`LinkedBlockingQueue`不设置容量的话，默认容量是`Integer.MAX_VALUE`，而采用`Executors`创建线程池的时候并未设置容量，于是我们可以不断的往里面加任务，最终可能导致OOM。

可以看到，开发者想要在代码中使用线程池还是比较简单的。下面让我们进入源码解读一下这个 jdk1.8 下的核心构造器

```java
 public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
 /*这里传入指定容量的阻塞队列*/    BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
     
     //acc 获取调用上下文
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
     //核心线程数量，类比于正式员工数量
        this.corePoolSize = corePoolSize;
     //最大线程数量，核心线程+临时线程
        this.maximumPoolSize = maximumPoolSize;
     //多余任务等待队列，当线程已经达到最大数量任务还没有做完，就进入这个等待队列
        this.workQueue = workQueue;
     //非核心线程的空闲时间，即外包人员等待工作的时间，如果超出这个时间，就将其释放
        this.keepAliveTime = unit.toNanos(keepAliveTime);
     //创建线程的工厂，在这个地方可以统一处理创建的线程属性
     //毕竟每个公司对员工的要求不一样
        this.threadFactory = threadFactory;
     //线程池拒绝策略，当任务太多，人也不够，需求池也排满了。
     //默认是不处理，抛出异常告诉任务提交者，这边忙不过来了
        this.handler = handler;
    }
```

如果把线程池比作公司，公司有正式员工处理正常业务，如果工作量大，会雇佣外包人员。闲时就会释放外包人员来减少公司管理开销。一个公司因为成本关系，雇佣的人员始终是有最大的数量限制。如果已经到达了最大线程数，任务还是处理不过来，就走**需求池 排任务**

在线程池中添加一个任务：execute()

```java
//Integer.SIZE=32 就是int在java中的字节数
private static final int COUNT_BITS = Integer.SIZE - 3;  // =29
private static final int CAPACITY   = (1 << COUNT_BITS) - 1; //容量
//定义线程状态如下  
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
//原子计算，还是获得线程状态ctl
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    
    int c = ctl.get();
    // 如果线程池的当前线程数数量小于核心线程数，则创建新的线程执行任务
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true)) //true表示添加的是核心线程
            return;
        c = ctl.get();
    }
    //判断 线程池是否正在运行并且 任务队列是否允许插入，
    //插入成功后再次验证新的线程状态下，线程池是否可以运行
    //如果不在运行，移除插入任务，然后抛出拒绝策略。
    //如果在运行，且线程池没有线程了，启用一个线程
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    //添加非核心线程 如果失败，直接拒绝。
    else if (!addWorker(command, false))//false参数表示 添加非核心线程
        reject(command);
}
```

从上述代码中可以看出**addWorker**主要负责创建新的线程并执行任务，代码比较复杂。就不展示出来了。 

- 首先判断是否能够添加工作线程，这个直接判断线程池的状态值 大于等于SHUTDOWN，就不提交任务了，直接返回
- 然后做自旋，更新创建线程数量。通过判定当前创建的线程是否为核心线程，如果是并且当前线程数小于corePoolSize，则跳出循环，开始创建新的线程。

然后：

- 获取线程池的主锁：线程池的**工作线程通过Worker类实现**，通过ReentrantLock保证安全
- 添加线程到workers中(即线程池中)
- 启动新的线程

而workers的声明如下：可以看出**线程池底层的存储结构是一个HashSet**

```java
private final HashSet<Worker> workers = new HashSet<Worker>();
```



**worker处理队列任务**

1.首先判断是否为第一次执行任务，或者从队列中可以获取到任务

2.获取到任务后，执行beforeExecute 做任务前处理

3.执行任务

4.执行afterExecute 做任务后处理



**线程池总结**：线程池的本质是一个HashSet，多余的任务会放在阻塞队列中。只有当阻塞队列满了之后，才会触发非核心线程的创建，而当线程池中的线程数量达到它的最大容量，而且队列也满了的话，就会执行拒绝测录。另外还提供了两个钩子(beforeExecute,afterExecute)给我们，让我们可以在执行任务前后做一些处理。

它的关键技术是：锁(lock,cas)，阻塞队列，hashSet(资源池)

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/29.png?raw=true">



##### 8.GC算法

[JVM垃圾收集与GC算法](https://www.jianshu.com/p/43c1b262d36b)

[Java GC算法 垃圾收集器](http://www.importnew.com/23752.html)

[GC机制](https://juejin.im/post/5a15be736fb9a044fc4464d6)

Sun公司的JVM采用的是**根搜索算法**来判定哪些对象要被回收；

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/24.png?raw=true">

基本思想是：从一个叫GC Roots的根节点触发，向下搜索，如果一个对象不能够到达GC Roots的话，就说明该对象不再被引用，可以被回收。如上图的Object5,6,7，它们虽然相互引用，但其实已经没有作用了。

GC Roots包括：

- 虚拟机栈中引用的对象：如下面在方法中声明的对象，它的引用就存储在虚拟机栈中

```java
public void method(){
    Object obj=new Object();
}
```

- 方法区中类静态属性实体引用的对象：如下面直接在类中的static属性对象

```java
private static Object object; 
```

- 方法区中常量引用的对象

- 本地方法栈中JNI引用的对象

补充概念：在JDK1.2之后引入了四个引用类型：

- 强引用：new出来的对象都是强引用，只要引用还在，GC无论如何都不会回收，即使抛出`OutOfMemoryError`异常
- 软引用：只有当JVM内存不足时才回收，用来描述一些有用但是不是必须的对象
- 弱引用：只要发生GC，立马回收，描述非必须对象，强度比软引用更弱。
- 虚引用：JVM完全忽略虚引用，它唯一的作用就是做一些跟踪记录，辅助finalize函数的使用。

最后总结一下：什么样的类需要被回收

> 1. 该类的所有实例都已经被回收
> 2. 加载该类的ClassLoad已经被回收
> 3. 该类的反射类java.lang.Class对象没有被引用



**现代的GC都是分代收集：在新生代和老年代执行不同的收集算法。**

分为两种：minor GC，Full GC(也称为Major GC)，**Minor GC 是发生在Eden中**的垃圾收集动作，采用的是**复制算法**。Eden几乎是所有对象出生的地方，即Java对象申请的内存以及存放都是在这个地方。Java中的大部分对象通常不需长久存活，具有朝生夕灭的性质。

当一个对象被判定"死亡"的时候，GC就有责任来回收掉这部分对象的内存空间。Eden是GC收集垃圾的频繁区域。当对象在Eden出生后，在经过一次Minor GC后，如果对象还存活，并且被另一块Survivor区域所容纳，则使用复制算法将这些任然还存活的对象复制到另一块Survivor区域中，然后清理所使用过的Eden和Survivor区域，并且将这些对象的年龄设置为1，以后对象在Survivor区熬过一次Minor GC，就将其年龄+1，当对象的年龄达到某个值时(默认是15岁) 这些对象会搬进老年代。

但是对于较大的对象(即需要分配一块较大的连续内存空间)则是直接进入到老年代。

**复制算法：采用的方式是从根节点进行扫描，将存活的对象移动非活动的Survivor中**。当存活对象较少时，复制算法比较高效。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/25.png?raw=true">



**Full GC(也叫Major GC)是发生在老年代**的垃圾收集动作，采用**标记-清除算法**或者**标记-整理算法**

现实生活中，老年代的人通常会比新生代的人“早死”，堆内存中的老年代不同于这个现象。老年代里的对象几乎都是在Survivor区域中熬过来的，它们并不容易死掉。因此，Full GC发生的次数并不会有Minor GC那么频繁，并且一次Full GC要比进行一次Minor GC的时间要长。

另外，**标记-清除算法**收集垃圾的时候会产生许多内存碎片(即不连续的内存空间)，此后需要为较大的对象分配内存时，若无法找到足够的连续空间时，就会提前触发一次GC的收集动作。

**标记-清除算法：采用从根集合开始扫描，将存活对象进行标记，标记完毕之后，再扫描整个空间中未被标记的对象，并进行清除。**

如图：蓝色是有被引用的部分，褐色是没有被引用的部分。在Marking（标记）阶段需要全盘扫描，

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/26.png?raw=true">

清除之后，出现很多内存碎片：在空间中存活的对象较多时，效率较高。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/27.png?raw=true">

**标记-压缩算法**也叫**标记-整理算法**：与算法标记-清除算法类似，都是先对存活的对象进行标记。但是**清除之后会把活的对象向左端空闲空间移动，然后再更新引用它们的指针**。弥补了标记-清除算法的缺点，但是成本也增加了，适用于老年代(旧生代)

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/28.png?raw=true">



##### 垃圾收集器

在JVM中，GC是由垃圾回收器来执行，所以在实际应用场景中，我们需要选择合适的垃圾收集器。

**1.串行收集器(Serial GC)**

最古老，最稳定以及效率高的收集器。在新生代复制算法，老年代标记-压缩；垃圾回收过程会暂停服务。

**2.ParNew GC**

基本**和Serial GC一样，单本质区别是加入了多线程机制**，提高了效率，这样它就可以被用于服务端上，同时它可以与CMS GC配合，所以一般用于server端。**新生代并行，老年代串行。**

**3.Parallel Scavenge GC**

在整个扫描和**复制**过程采用**多线程**的方式进行，适用于多CPU，对暂停时间要求较短的应用，是server级别的默认GC方式。和ParNew类似，但是更加关注吞吐量。

**4.CMS (Concurrent Mark Sweep)收集器**

该收集器的目标是解决Serial GC停顿的问题，**以达到最短回收时间。**常见的B/S架构的应用就适合这种收集器，因为具有高并发，高响应的特点，**CMS基于标记-清除算法。**它的运作比较复杂：

1.初始标记(CMS initial mark): 暂停服务，标记一下GC Roots能直接关联到的对象，速度很快

2.并发标记(CMS concurrent mark)：进行GC Roots Tracing过程

3.重新标记(CMS remark) ：暂停服务，为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，

4.并发清除(CMS concurrent sweep)

**5.G1收集器**

相比CMS收集器有改进，首先**基于标记-压缩算法/标记-整理算法**，不会产生内存碎片，其次可以较精确的控制停顿。和ParNew类似，当新生代的空间占用达到一定比例，开始收集。

**6.Serial Old收集器**

Serial Old是Serial收集器的老年代版本，同样**使用单线程执行收集，使用标记-整理算法**。主要适用Client模式下的虚拟机。

**7.Parallel Old 收集器**

是Parallel Scavenge 收集器的老年代版本，使用多线程和“标记-整理”算法。

下图是年轻代和老年代的垃圾收集器，其中的连线表示可以组合使用。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/34.png?raw=true">



##### 9.DB连接池

[数据库连接池技术详解](https://juejin.im/post/5b7944c6e51d4538c86cf195)

**数据库连接池技术，即使用来分配，管理，释放数据库连接的**。实际上在简单的数据库应用中，对于数据库的访问不是很频繁，创建一个连接用完之后关闭它，这样做的开销并不是很大。但是如果是一个比较复杂的系统，频繁的建立，关闭连接会极大的降低系统的性能。

连接复用：通过建立一个数据库连接池以及一套连接使用管理策略，使得一个数据库连接可以得到高效，安全的复用，避免了数据库连接频繁建立，关闭的开销。

对于共享资源，有一个很著名的设计模式：资源池。该模式正是为了解决资源频繁分配，释放所造成的问题。把该模式应用到数据库连接管理领域，就是建立一个数据库连接池。

具体原理：在内部对象池中维护一定数量的数据库连接，并对外暴露数据库连接获取和返回方法。`getConnection() 获取连接` `releaseConnection() 返回连接` 。每次有访问的时候，数据库连接池就会给用户分配一个数据库连接，当用户用完了连接之后，连接池再将连接回收。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/30.png?raw=true">

数据库连接池的优势很明显：

- 资源重用
- 更快的系统响应速度
- 新的资源分配方式
- 统一的连接管理，避免数据库连接泄露



**自己实现一个简单的数据库连接池**

```java

public class MyPool {

    private final int init_count = 3; //初始化链接数目
    private final int max_count = 6; //最大连接数量
    private int current_count = 0; //当前正在使用的连接数
   
    //连接池，用来存放初始化链接
    private LinkedList<Connection> pool = new LinkedList<Connection>();

    //构造函数，初始化一定数量的连接放入连接池
    public MyPool() {
        for (int i=0;i<init_count;i++){
            //记录当前正在使用的连接数
            current_count++;
            Connection connection = createConnection();
            pool.addLast(connection);
        }
    }

    //创建新的连接
    public Connection createConnection() {
        try {
            Class.forName("com.mysql.jdbc.Driver");
            Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/keyan","root","root");
            return connection;
        }catch (Exception e){
            System.out.println("数据库链接异常");
            throw new RuntimeException();
        }
    }

    //获取链接
    public Connection getConnection() {
        if (pool.size() > 0){
            //removeFirst删除第一个并且返回
            return pool.removeFirst();
        }
        //如果池中没有连接，并且正在使用的连接要小于最大连接数 
        if (current_count < max_count){
            //记录当前使用的连接数
            current_count++;
            //创建链接
            return createConnection();
        }
        
        throw new RuntimeException("当前链接已经达到最大连接数");
    }

    //释放链接，
    public void releaseConnection(Connection connection){
        //如果当前池中的连接数小于初始数量，将该连接收回
        if (pool.size() < init_count){
            pool.addLast(connection);
            current_count--;
        }else {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```





##### 10.内存限制下，找出2GB文件中最大的100个数字和重复次数最多的100个字符串。

[经典面试问题: Top K 之 ---- 海量数据找出现次数最多或，不重复的。](https://juejin.im/post/5aa0ee9f518825557c010bc0)











