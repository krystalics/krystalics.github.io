 参考文章    ： [Java内存泄露 ](https://www.ibm.com/developerworks/cn/java/l-JavaMemoryLeak/index.html)  [另一篇文章](https://www.jianshu.com/p/54b5da7c6816)

大家都知道Java有GC机制，开发人员并不需要像C++的研发一样要解决内存分配问题，但是实际上GC并不能保证我们的系统中没有内存泄露。要知道，很多服务端程序的运行时间是按年来计算的，漫长的运行时间让再小的内存泄露都会变得致命。了解了GC的具体管理方式，或许对避免内存泄露有大的帮助。

##### Java是如何管理内存的

**Java的内存管理就是对象的分配和释放问题**。开发人员需要 `new` 为每个对象(除基本数据类型外)申请空间，(有类似于Spring的框架可以管理对象的生命周期，不需要手动new，但是也有这个过程)，所有对象都在堆(Heap)中分配空间。另外，对象的释放是由GC决定和执行的。

总的来说：内存分配是由程序完成的，而内存释放是由GC完成的。

GC为了能正确释放对象，必须监控每个对象的运行状态，包括对象的申请、引用、被引用、赋值等，GC都需要进行监控。

GC释放对象的根本原则是：**该对象不再被引用**      重点是如何对其进行判定

为了更好理解GC的工作原理，我们可以将对象考虑为有向图的顶点、将引用关系考虑为图的有向——从引用者指向被引用者。大多数程序从main进程开始，那么整个程序的对象引用图就是以main进程顶点开始的一棵树。在这棵树中，GC会回收不可达的顶点：如下图

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/1.gif?raw=true">

Java使用有向图的方式进行内存管理，可以消除循环引用的问题：例如有三个对象相互引用，只要它们和根进程不可达，GC就会回收它们。这种方式的优点是**精度高**，但是效率很低；另一种方式是使用计数器，例如COM模型采用计数器方式管理构件，它与有向图相比，精度很低(也很难处理循环引用的问题)，但执行速度很快。



##### 什么是Java中的内存泄露

根据以上论述，**内存泄露就是存在一些被分配对象**：**1.** 这些对象对于根节点是可达的，即在有向图中存在通路可以与其相连；**2.**这些对象在程序中不会再被使用

满足以上两个条件的对象，GC不会回收，所以它们任然占用内存。

举个例子： 下面的o对象看似被手动释放了，但是在Vector中任然保存着该对象的引用

```java
Vector v=new Vector(10);
for(int i=0;i<100;i++){
    Object o=new Object();
    v.add(o);
    o=null;
}
```

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/7.png?raw=true">



##### 那么如何防止内存泄露的发生？

**好的编码习惯**：最基本的建议就是尽早释放无用对象的引用，大多数开发人员使用临时变量的时候都是让引用变量在退出活动域之后自动设置为null，在使用这样的方式时要注意一些复杂的对象图。比如数组，列，树，图等。这些对象之间的相互引用关系较为复杂。建议在确认一个对象无用之后，将其所有引用显示设置为null

**好的测试工具**

在开发中不能完全避免内存泄露，关键要在发现有内存泄露的时候能够用测试工具发现它的位置。这样的工具有很多，这里就不列举了。

##### 注意HashMap，ArrayList这样的集合对象

它们经常会引发内存泄露，尤其是被声明为static时，它们的生命周期就会和引用程序一样长。





























































