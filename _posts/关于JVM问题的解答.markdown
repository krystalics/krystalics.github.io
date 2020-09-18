以下问题来自： [推荐史上最容易学的 JVM 教程，面试不再慌！](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247490720&idx=1&sn=b9a32a5c055ca3f4182d21be8f3b73e9&chksm=eb539996dc241080064617892de565718873503af21fa9bafe79aa7a3e87435a5ece6724aff1&mpshare=1&scene=23&srcid=&sharer_sharetime=1567944987139&sharer_shareid=9dafa29195228a32c015a56c9eb5b72d#rd) 

#### 内存模型以及分区，每个区放什么

参考 [CyC java并发](https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20%E5%B9%B6%E5%8F%91.md) 

**内存模型是Java程序在各种平台都能达到一致访问效果的关键**，我们都知道CPU的速度比内存要快很多，所以会有缓存和寄存器。而Java的内存模型就是搭建在这个基础之上，分为工作内存和主内存

工作内存：就是线程自己的私有内存，一般都是高速缓存或者寄存器中 (这会涉及到后面的并发机制)，而主内存就是存放变量的地方，其实就是堆啦。

那么按照上面的陈述，多个线程时会使用多个缓存，那么缓存中的数据和主存中的数据很可能不是一致的，所以需要一个机制来解决这个问题。**Java内存模型定义了8个操作来完成主存和工作内存的交互**

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/60.png?raw=true">

- read：将变量的值从主存中读取到工作内存
- load：得到read的值，放入工作内存的变量副本中
- use：将工作内存中变量的值传给CPU
- assign：将CPU的值传给工作内存，并赋值相应变量
- store：将工作内存中的值 传递到 主存中
- write：得到store的值，写到相应的变量中
- lock：作用于主存变量
- unlock：和上面一样

##### 这样的内存模型有三大特性：1.原子性，2.可见性，3.有序性

##### 1.原子性：

​	Java内存模型保证了read,load,use,assign,store,write,lock,unlock这些分散的操作都是原子的，例如对于一个int类型的变量进行assign赋值操作，这个操作是原子的，一步完成。但是如果是double，long等没有被volatile修饰的64位数据 ，读写操作都将分为两次32位的操作来进行。即**load,store,read,write 操作可以不具备原子性**。

这点很重要，比如一个int变量在1000个线程中自增，最终得到的数据并不会是1000，而是一个小于1000的数。这点可以从内存模型中得到答案，**当一个线程对它进行修改时（还没写入主存），另一个线程时从主存中获得该变量的当前值并不是从 之前的线程中获得值，自然导致两个线程的修改结果只修改了一次。**

而且即使修改一次变量，这个步骤也是分为：1.把变量值 从主存中取出 **read** 并 **load**，2.从工作内存中取出该变量值到CPU中执行操作 **use** 3.将值从CPU放回工作内存 **assign**，再到主存 **store** , **write**  。

##### 2.可见性

很明多个缓存中的线程对彼此并不可见，要保证可见性就必须让线程在主存中见面——即**将变量的值修改完store,write到主存中，并且是在其他线程读取该变量之前**。

- volatile：值的注意的是，只能保证可见性，并不能保证原子性
- synchronized，对一个变量unlock操作时，必须把变量的值同步会主存
- final，被final关键字修饰的字段在构造器初始完成时，并且没有发生this逃逸(即没有其他线程在该对象初始化一半时，访问该对象) ，其他线程就可以看见final字段的值

##### 3.有序性

这再单线程的程序中是毫无疑问的，都是从头到尾执行下来。但是并发的情况下，操作都是无序的，因为我们不知道哪个线程先执行，而且还会发生**指令重排序**(上面的volatile可以避免它) ，重排序不会影响单线程的执行，并发时才是致命的问题，因为很多时候逻辑上是没有错误的，这种问题是很难被发现的。

**volatile 通过添加内存屏障的方式来禁止指令重排，即重排序不能把后面的指令放到内存屏障之前。**

当然直接使用synchronized同步的话，同一时刻只有一个线程在执行程序，就不会有这个问题了。



除了我们的强制使用volatile和同步，**JVM其实自定义了一套 执行顺序**，用于优化。也叫happen-before 原则 ，很多人会直接翻译成 在什么之前发生， [java 8大happen-before原则超全面详解](https://www.jianshu.com/p/1508eedba54d) 这篇文章中给出不同的见解，先说是哪8大：

- 单线程 happen-before 原则：在同一线程中，书写在前面的操作 happen-before 后面的操作，这里是对后面的代码可见（即使前面的操作在之后发生，后面的代码也可以得知它的变化），而不是说前面的操作一定发生在后面的操作之前
- 锁 ：同一个锁的unlock 操作在happen-before 该锁的lock操作
- volatile：对一个volatile变量的写操作 happen-before 对此变量的其他读写操作
- happen-before 的传递性原理：A happen-before B，B happen-before C，则A happen-before C
- 线程启动：同一个线程的start方法 happen-before 此线程的其他方法
- 线程中断：对线程interrupt方法调用 happen-before 被中断线程的检测到中断发送代码
- 线程终结：线程的所有操作都 happen-before 线程的终止检测
- 对象创建 ： 一个对象的初始化完成 happen-before 它的finalize方法调用



“生效可见于” 可能更适合 happen-before ，意思就是这行 **代码的生效可以被其他的代码知道**，并没有书写顺序的前后，因为指令重排之后发现 写在后面的代码可以比前面的代码先执行。

```java
int a=3; //1
int b=a+1; //2

int c=3;//3
int d=4;//4
```

我们都知道 **2的赋值中会使用到 a变量**，那么Java 单线程的happen-before原则 就会保证 b 的值是4，即使指令重排序也不会影响这个结果。因为1在2的前面，所以单线程时就不用担心执行顺序会发生变化；这种有引用关系的 可以保证顺序，**但是3 和4 没有引用关系，它们直接谁先发生都不影响结果，指令重排序就可能发生，让d先于c赋值**。

上述代码在并发的情况下可能赋值顺序就完全不一样了，所以需要volatile禁止重排序

```java
volatile int a;
a=1;//1
b=a;//2
```

b依赖a，a是volatile变量 设置了内存屏障，对a的操作都是有屏障的，其他对a的操作都是有序的，并发时也是一样的

但假设两个操作是分在两个不同的方法中，没有书写上的顺序，

```java
volatile int x;
int b;
int c;
// 线程1 执行方法A
public void methodA(){
    b=4;//1
    x=3;//2 这两个无引用，所以顺序无所谓，只不过1的生效，2无论是否先于1执行都可见
}
// 线程2 执行方法B  
public void methodB(){
    c=x;//3
    c=b;//4  两个都是对c赋值，所以顺序很关键，3先于4发生 单线程happen-before原则
}
```

对**x的操作是有内存屏障的，所以就看两个线程的调用顺序**，又因为happen-before的传递性

- 假如是 1线程先于2线程 **就会有 2先于3 发生** ，**又因为3先于4，所以2先于4**，又1对2可见，所以对 4也是可见的。c的值自然只能是4
- 假如 2线程先于1线程  **就会有3先于2 发生**，又3先于4发生 但是 **1和4由于是在两个线程，并不符合单线程的happen-before，也没有volatile，所以顺序随机，c可能等于4，也可能等于0**



##### 上面说了很多，究竟什么是指令重排序呢？？？？？

[《Java 并发编程实战》之java内存模型](https://www.jianshu.com/p/47f999a7c280) 

```java
public class A {
   public int a;
   public boolean b = false;

   public void methodA(){
       a = 3;
       b = true;  
       a = a + 1;
   }
} 
```

在执行methodA的时候，发现 a变量操作了两次，如果没有指令重排序

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/61.png?raw=true">

图中CPU和缓存要进行9次通信，缓存和内存通信7次。假设cpu和缓存通信用时1ms，缓存系统和内存通信一次用时10ms，那么总用时 1×9+7×10=79ms。 指令重排之后，过程如下

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/62.png?raw=true">

指令重排之后 1×6+6×10=66ms。

但是在上文中说了，指令重排序会带来很多问题，所以要警惕这个。



#### 一个对象从创建到销毁都是怎么在这些部分里存活和转移的

出生在年轻代中的Eden，gc时把和Survivor区中存活的对象 都移动到另一个Survivor区域中，经过一定次数gc还活着的对象，就会移动到老年代中。



##### 内存的哪些部分会参与GC的回收，回收策略是什么

**根搜索算法**：死掉的对象会被gc，而判断活着的依据 就是从GC Roots的根节点触发，向下搜索如果一个对象不能到达GC Roots，就会被回收。

而GC Roots 包括：

- 栈中引用的对象，如在方法中声明的

```java
public void method(){
    Object obj=new Object(); //obj 就是一个GC Root
}
```

- 方法区中 静态属性实体引用的对象

```java
private staitc Object object; //object 也是一个GC Root
```

- 方法区中 常量引用的对象

```java
private final Object....;
```

- 本地栈中JNI 引用的对象



这就牵扯出 jdk1.2之后的4个引用类型：

- 强引用：new出来的对象都是强引用，只要引用还在，GC不会回收这部分，即使抛出OOM异常
- 软引用：当JVM内存不足时，即使它被GC Root引用，还是会GC掉它，用来描述一些有用但不是必须的对象
- 弱引用：只要GC 就回收，不管有没有被GC Root引用
- 虚引用：JVM 完全忽略，只是用来记录一些东西的



回收策略是：现代的GC都是 分代的，年轻代和老年代执行不同的策略

年轻代：复制算法

老年代：标记清楚，或者 标记压缩

JVM集成了很多垃圾收集器，都是封装好的。

##### Java的内存模型是怎么设计的，为什么要这么设计





##### 结合内存模型的设计谈谈volatile关键字的作用

