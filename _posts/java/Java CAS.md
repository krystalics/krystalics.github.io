参考文章： [Java CAS原理剖析](https://juejin.im/post/5a73cbbff265da4e807783f5)

在Java并发中，我们最初接触的是`synchronized`关键字，但是`synchronized`属于重量级锁，很多时候会引起性能问题，这是候`volatile`也是不错的选择，但是`volatile`不能保证原子性，只能在某些场合下使用。

像`synchronized`这样的独占锁属于**悲观锁**，它是假设某一段时间内一定会发生冲突，所以将该资源锁住。除此之外还有**乐观锁**，它的定义是假设某一段时间内没有冲突，那么我们正好进行操作，如果要是发生冲突了，那我就重试直到成功，乐观锁最常见的就是`CAS`

在Concurrent包下的`ReenterLock`内部的**AQS**，还有各种**Atomic**开头的原子类，内部都应用了CAS，最常见的就是我们在并发编程的时候遇到 i++ 这种情况，传统的方法就是将该方法上锁

```java
public class Test{
    public volatile int i;
    public synchronized void add(){
        i++;
    }
}
```

我们也知道这样做性能可能不是太好，还可以使用`AtomicInteger`，就可以保证`i`原子的`++`操作了。

```java
public class Test{
    public AtomicInteger i;
    public void add(){
        i.getAndIncrement();
    }
}
```

那么接下来就是看看 `getAndIncrement`方法,以及他的调用链

```java
public final int getAndIncrement(){
    return unsafe.getAndAddInt(this,valueOffset,1);
}
```

在unsafe.class中

```java
public final int getAndAddInt(Object var1,long var2,int var4){
    int var5;
    do{
        var5=this.getIntVolatile(var1,var2);
    }while(!this.compareAndSwapInt(var1,var2,var5,var5+var4));
    
    return var5;
}
```

这里的`compareAndSwapInt`函数，也是`CAS`缩写的由来。我们知道前面调用的时候有个`unsafe`，所以我们这里结合`Unsafe`和`AtomicInteger`类来分析；

```java
public class AtomicInteger extends Number implements java.io.Serializable{
    private static final long serialVersionUID=...;
    
    private static final Unsafe unsafe=Unsafe.getUnsafe();
    private static final long valueOffset; //偏移量
    //static块的加载发生于类加载的时候，并且最先初始化。
    static {
        try{
  //这时我们就已经调用unsafe的objectFieldOffset从Atomic类文件中获取value的偏移量
 //那么valueOffset其实是记录value的偏移量。
            valueOffset = unsafe.objectFieldOffset(
                AtomicInteger.class.getDeclaredField("value")
            );
        }catch(Exception ex){
            throw new Error(ex);
        }
    }
    //其实实际存储的值是放在value中
    private volatile int value; 
    //....
}
```

再回到上一个函数`getAndAddInt`，我们看到`var5`获取的是什么，通过调用`unsafe`的`getIntVolatile(var1,var2)`，这个是native方法，具体实现需要到JDK的源码中看，

```java
public native int getIntVolatile(Object var1, long var2);
```

其实就是获取`var1`中`var2`偏移量处的值。`var1`就是`AtomicInteger`，`var2`就是`valueOffset`。实际上就是获取`AtomicInteger`中偏移量为`valueOffset` ,这样我们就从内存里获取到现在`valueOffset`处的值了。

**重点**：在getAndAddInt(Object var1,long var2,int var4)方法中调用了

`compareAndSwapInt(var1,var2,var5,var5+var4)`其实换成可以替换成比较易懂的`compareAndSwapInt(obj,offset,expect,update)`，意思就是如果`obj`内的`value`和`expect`相等，就证明没有其他线程改变过这个变量，那么就更新它为`update`，如果这一步`CAS`没有成功，那么就采用自旋的方式继续进行`CAS`操作，底层原理是在 `JNI`里借助一个CPU指令完成的。所以还是原子操作。

> 自旋是指无限循环，不断重复该动作 
>
> JNI(Java Native Interface，java本地接口)，使得Java虚拟机中的Java程序可以调用本地应用/库，也可以被其他程序调用。本地程序一般是其他语言编写(C++,C或汇编等)，并且被编译为基于本机硬件和操作系统的程序。



##### CAS底层原理

CAS底层使用`JNI`调用C代码实现的，如果有`Hotspot`源码，那么在`Unsafe.cpp`里可以看到实现入下：

```cpp
static JNINativeMethod method_15[]={
    //...一堆代码
    {CC"compareAndSwapInt",CC"("OBJ"J""I""I"")Z", FN_PTR(Unsafe_CompareAndSwapInt)},
    {CC"compareAndSwapLong",CC"("OBJ"J""J""J"")Z", FN_PTR(Unsafe_CompareAndSwapLong)},
    //...一堆代码
}
```

里面调用 `Unsafe_CompareAndSwapInt` 

```cpp
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END
```

p是取出来的对象，addr是p中offset处的地址，最后调用`Atomic::cmpxchg(x,addr,e)`其中参数x是即将更新的值，参数e是原内存的值。代码中可以看到cmpxchg有基于各个平台的实现。

这里原作者选择了`Linux X86`平台下的源码分析。

```c
inline jint     Atomic::cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value) {
  int mp = os::is_MP();
  __asm__ volatile (LOCK_IF_MP(%4) "cmpxchgl %1,(%3)"
                    : "=a" (exchange_value)
                    : "r" (exchange_value), "a" (compare_value), "r" (dest), "r" (mp)
                    : "cc", "memory");
  return exchange_value;
}
```

这是一小段汇编，`__asm__`说明是ASM汇编，`__volatile__`禁止编译器优化。`os::is_MP()`判断当前系统是否为多核系统，如果是就给总线加锁，所以同一芯片上的其他处理器就暂时不能通过总线访问内存，保证了该指令在多处理器环境下的原子性。

最终JDK通过CPU的`cmpxchgl`指令的支持，实现了`AtomicInteger`的`CAS`操作的原子性。



##### CAS的问题

##### 1.ABA问题

CAS需要在操作值的时候检查下值有没有发生变化，**但是对于A->B->A这种情况使用CAS进行检查时会发现它的值并没有发生变化**。常见的解决方案是在变量前面加上版本号，每次更新的时候把版本号加一，由`A-B-A`变成`1A-2B-3A`。目前JDK的atomic包里提供一个类，`AtomicStampedReference`来解决ABA问题，这个类的compareAndSet方法作用是首先**检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等**，则以原子方式将该引用和该标志的值设置为给定的更新值。

##### 2.循环时间长，开销大

我们之前说过，CAS不成功的话，则会原地自旋，长时间不成功，就会给CPU带来非常大的执行开销。**CAS不成功是指，发生了冲突，即多个线程同时访问该资源。**



总结：

1.使用CAS在线程冲突严重的时候会大幅降低程序性能，因为不断冲突下 CAS 一直自旋，消耗CPU性能。**所以CAS比较适合于线程冲突较少的情况使用**。

2.synchronized在jdk1.6之后进行优化，底层实现主要依靠Lock-Free队列：基本思路是自旋后开始阻塞，竞争切换后继续竞争锁，牺牲公平性，但是获得了高吞吐量。在线程冲突较少的时候与CAS有类似性能：而线程冲突严重的情况下，性能远高于CAS、

而从jdk1.6开始，对锁的实现引入了大量的优化，如锁粗化（Lock Coarsening）、锁消除（Lock Elimination）、轻量级锁（Lightweight Locking）、偏向锁（Biased Locking）、适应性自旋（Adaptive Spinning）等技术来减少锁操作的开销。而其中自旋锁的原理，类似于CAS自旋，甚至比CAS自旋更为优化

 具体情况请参考：<https://www.jb51.net/article/86192.htm> 





































'