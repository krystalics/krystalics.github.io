正确理解Thread Local 的原理与适用场景

本文转发自[**技术世界**](http://www.jasongj.com/)，[原文链接](http://www.jasongj.com/java/threadlocal/)　 

---

##### ThreadLocal解决什么问题

网上有很多不恰当的理解。比如*ThreadLocal为解决多线程程序的并发问题提供了一种新思路；ThreadLocal的目的是为了解决多线程访问资源时的共享问题。*ThreadLocal并**不解决多线程共享变量**的问题。

合理的理解是：ThreadLocal变量它的基本原理是  **同一个 ThreadLocal 所包含的对象**(对`ThreadLocal<String>` 而言，该对象为String类型)**在不同Thread中有不同的副本**。

> ThreadLocal 提供了线程本地的实例。它与普通变量的区别在于：每个使用该变量的线程都会初始化一个完全独立的实例副本(同上，对`ThreadLocal<String>`而言就是String对象的副本) 。ThreadLocal 变量通常被private static修饰，当一个线程结束后，它所使用的所有ThreadLocal 相对的实例副本都可被回收。



总的来说，**ThreadLocal 适用于每个线程需要独立的实例且该实例需要在多个方法中被使用，也即变量在线程隔离而在方法或类间共享的场景**。看到这里就会有同学问了，如果要达到这个目的，在类里面声明一个实例变量不就好了，何必还有专门的ThreadLocal。在这里解释一下。

首先我们要清楚在初始化对象的时候，对象中的实例存储位置：为方法区和堆，**两个都是线程共享的**，只有一些方法内的局部变量是线程私有的，存储在虚拟机栈中。

其中**方法区**(Method Area)：线程共享。存储类的**实例变量，常量，静态变量**，即时编译器JIT编译的代码。所以我们**自己在类中声明的变量在多线程程序中是做不到线程隔离的**

通俗的来说，**ThreadLocal 就是在并发条件下，提供了单线程中实例变量的功能，方便各个方法调用又做到线程隔离。**那么ThreadLocal是如何实现的呢？请看下面的原理解释。

```java
package com.company;

import java.util.concurrent.CountDownLatch;

public class ThreadLocalDemo {

  private static class InnerClass {

    private static ThreadLocal<StringBuilder> counter = ThreadLocal.withInitial(StringBuilder::new); //这是Lambda表达式，最新写法

    public void add(String newStr) {
      StringBuilder str = counter.get();
      counter.set(str.append(newStr));
    }

    public void print() {
      System.out
          .printf("Thread name:%s , ThreadLocal hashcode:%s, Instance hashcode:%s, Value:%s\n",
              Thread.currentThread().getName(),
              counter.hashCode(),
              counter.get().hashCode(),
              counter.get().toString());
    }

    public void set(String words) {
      counter.set(new StringBuilder(words));
      //因为这些方法都是多线程共享的，所以如果分为两个print部分，就会混乱
      // System.out.print("Set, ");
      // print();
      System.out
          .printf("Set,Thread name:%s , ThreadLocal hashcode:%s, Instance hashcode:%s, Value:%s\n",
              Thread.currentThread().getName(),
              counter.hashCode(),
              counter.get().hashCode(),
              counter.get().toString());
    }
  }
  
  public static void main(String[] args) throws InterruptedException {
    int threads = 3;
//CountDownLatch用于阻塞一个线程，等待其它线程先后到达某个条件的时候
//再执行这个线程的后续操作。
    CountDownLatch countDownLatch = new CountDownLatch(threads);
    InnerClass innerClass = new InnerClass();
    for (int i = 1; i <= threads; i++) {
      new Thread(() -> {
        for (int j = 0; j < 4; j++) {
          innerClass.add(String.valueOf(j));
          innerClass.print();
        }
        innerClass.set("Hello world");
        countDownLatch.countDown();
      }, "thread-" + i).start();
    }
    countDownLatch.await(); //阻塞
  }
}
```

那么知道了应用场景和如何使用之后，我们再来探究一下它的底层原理和实现。直接来看JDK采用的方案：

##### ThreadLocal原理

看到上面的代码，我们知道ThreadLocal变量也是类中的实例对象，它的存储空间在堆中，本身也是属于线程共享区域的。所以需要采取一些方案来实现它的目的。

1.**Thread作为键，Map映射ThreadLocal中的泛型实例。可以满足每个Thread有一个独立的备份**，当每个线程访问该ThreadLocal时，就向Map中添加一个映射，而当每个线程结束的时候，清除该映射。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/31.png?raw=true">

但是该方案有几个问题：

- 首先增加和减少映射都需要访问Map，所以需要保证Map的线程安全。像是ConcurrentHashMap，保证并发。但是毕竟应用到锁，性能会差些
- 线程结束时，需要保证它所访问的所有ThreadLocal对应的映射均删除，毕竟ThreadLocal是线程私有的，线程都结束了，它里面的东西自然也应该消失。

由于各种原因，JDK并未采用这种方案。

2.上一个方案是因为要并发访问Map，所以我们尽量避免这个问题。如果**该Map映射在Thread内部，就不存在并发访问的问题了**。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/32.png?raw=true">

该方案没有锁的问题，但是由于每个线程访问某个ThreadLocal变量后，都会在自己的Map内维护该ThreadLocal变量与具体实例的映射，如果不删除这些引用(映射)这些ThreadLocal就不能被回收，同样会造成内存泄露。

看到上面的图和解释，是不是更加蒙蔽了？别急，来看看ThreadLocal 在JDK 8中的实现

##### ThreadLocalMap和内存泄露

第二个方案，Map由ThreadLocal类的静态内部类**ThreadLocalMap**提供。该类的实例维护某个ThreadLocal与具体的实例映射。与HashMap不同的是，ThreadLocalMap的每个Entry都是一个对**键**的**弱引用**，这一点从下列`super(k)`中看出。另外，每个Entry都包含了对**值**的**强引用**

```java
static class Entry extends WeakReference<ThreadLocal<?>>{
    Object vlaue;
    Entry(ThreadLocal<?> k,Object v){
        super(k); //父类 的弱引用 
        value=v;  //关于 value的强引用
    }
}
```

使用弱引用的原因在于，当没有强引用指向ThreadLocal变量时，它可以被回收，避免上文中ThreadLocal不能被回收而造成的内存泄露。当ThreadLocal被回收的同时，对应Entry的键变为了null，使得该Entry无法被移除，造成了另一个方面的内存泄露。

*注：Entry虽然是弱引用，但是对value具体实例是强引用，所以无法避免具体实例相关的内存泄露*

在Thread类中有 `ThreadLocal.ThreadLocalMap threadLocals;`在ThreadLocal类中获得ThreadLocalMap，

对于已经不再被使用且已被回收的 ThreadLocal 对象，它在每个线程内对应的实例由于被线程的 ThreadLocalMap 的 Entry 强引用，无法被回收，可能会造成内存泄漏

针对这个问题，ThreadLocalMap中的set()方法中，通过replaceStateEntry()将所有键为null的Entry的值设置为null，使得该值可以被回收。另外，在rehash()方法中通过expungeStateEntry()将键和值都为null的Entry设置为null，从而使得该null可以被回收。通过这个方式，ThreadLocal防止内存泄露。

```java
private void set(ThreadLocal<?> key, Object value) {
  Entry[] tab = table;
  int len = tab.length;
  int i = key.threadLocalHashCode & (len-1);

  for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
    ThreadLocal<?> k = e.get();
    if (k == key) {
      e.value = value;
      return;
    }
    if (k == null) {
      replaceStaleEntry(key, value, i);
      return;
    }
  }
  tab[i] = new Entry(key, value);
  int sz = ++size;
  if (!cleanSomeSlots(i, sz) && sz >= threshold)
    rehash();
}
```



案例：

对于Java Web引用而言，Session保存了很多信息。很多时候需要通过Session获取信息，有些时候又要修改Session的信息。一方面，需要保证每个线程有自己单独的Session实例，另一方面由于很多地方需要操作Session，存在多个方法共享Session的需求。**如果不使用ThreadLocal，可以在每个线程内部构建一个Session实例，并将该实例在多个方法间传递。**

```java
public class SessionHandler{
    @Data
    public static class Session{
        private String id;
    	private String user;
   		private String status;
    }
    
    public Session createSession() {
    return new Session();
  }

  public String getUser(Session session) {
    return session.getUser();
  }

  public String getStatus(Session session) {
    return session.getStatus();
  }

  public void setStatus(Session session, String status) {
    session.setStatus(status);
  }

  public static void main(String[] args) {
    new Thread(() -> {
      SessionHandler handler = new SessionHandler();
// 每个线程内部 都要手动创建一个 Session，而且和SessionHandler的耦合比较高
// 像下面操作Session，都需要手动传进参数。
      Session session = handler.createSession();
      handler.getStatus(session);
      handler.getUser(session);
      handler.setStatus(session, "close");
      handler.getStatus(session);
    }).start();
  }
}
```

如果使用，ThreadLocal：

```java
public class SessionHandler {
// 只比上一个版本 多这么一行代码，session由ThreadLocal来创建管理
  public static ThreadLocal<Session> session = new ThreadLocal<Session>();

  @Data
  public static class Session {
    private String id;
    private String user;
    private String status;
  }

    
  public void createSession() {
    session.set(new Session());
  }

  public String getUser() {
    return session.get().getUser();
  }

  public String getStatus() {
    return session.get().getStatus();
  }

  public void setStatus(String status) {
    session.get().setStatus(status);
  }

  public static void main(String[] args) {
    new Thread(() -> {
      SessionHandler handler = new SessionHandler();
   //线程内部不用自己创建Session了，和SessionHandler的耦合度也降低了
      handler.getStatus();
      handler.getUser();
      handler.setStatus("close");
      handler.getStatus();
    }).start();
  }
}
```

























