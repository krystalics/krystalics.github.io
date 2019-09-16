[java8 使用Map中的computeIfAbsent方法构建本地缓存，提高程序效率 ](https://my.oschina.net/cloudcoder/blog/217775) 

[ConcurrentHashMap.computeIfAbsent引发的bug](https://www.jianshu.com/p/5f6a77baf0eb)

在Java8中，Map接口新增了一个方法 `computeIfAbsent`，通过此方法可以**构建Java本地缓存，降低程序的计算量，程序的复杂度**。

`public V computeIfAbsent(K key,Function<? super K,? extends V> mappingFunction)`

Map不就是这样干的吗？直接put就好了，为啥还要多加一个东西呢？这个问题在最后给出一个答案，

`computeIfAbsent()`方法首先判断缓存Map中是否存在指定的key值，如果存在会自动调用 mappingFunction(key) 计算key的value，对了它和put其中一个不一样的地方就是 它的value是由一个函数计算得出的。 然后将key,value放入缓存中(这点和put没有不一致)

java8会使用thread-safe方式从缓存中获取记录。我们从一个例子来展现出它的作用：



##### Fibonacci数列的计算

使用递归，产生大量重复计算

```java
public int fibonacci(int n){
    if(n<2) return n;
    System.out.println("计算 "+n) //记录n被计算了几次
    return fibonacci(n-2)+fibonacci(n-1);
}
```

我们使用map进行缓存,在考虑并发的情况下会比较复杂

```java
Map<Integer,Integer> cache=new HashMap<>();

public int fibonacci(int n){
    if(n<2) return n;
    Integer res=cache.get(n);
    if(res==null){
        synchronized(cache){ 
 //并发的时候，有其他线程正在put，所以进入锁之后需要再次判断，以免重复运算          
            res=cache.get(n);
            if(res==null){
                System.out.println("计算 "+n) //记录n被计算了几次
                res=fibonacci(n-2)+fibonacci(n-1);
                cache.put(n,res);	
            }
        }
    }
    return res;
}
```

而最新的方法，只需要将原本函数主体加入到 computeIfAbsent中即可

```java
Map<Integer,Integer> cache=new ConcurrentHashMap<>();

public int fibonacci(int n){
    return cache.computeIfAbsent(n,(key)->{
        if(n<2) return n;
    	System.out.println("计算 "+n) //记录n被计算了几次
    	return fibonacci(n-2)+fibonacci(n-1); //返回的值会被自动put进cache
    });
}
```

我们会看到上面是ConcurrentHashMap，这是为了线程安全考虑。单线程下 HashMap也是可以的，而ConcurrentHashMap使用这个方法的时候，需要非常注意一点：

**computeIfAbsent在使用的时候，计算value的过程中一定不能出现对map的修改操作，否则如果修改的key和computeIfAbsent的key分到同一个桶**

put方法和computeIfAbsent方法同时卡在了一个ReservationNode对象上。

查看ConcurrentHashMap的源码可以发现，这种情况在bucket没有初始化的时候会发生，**简单来说computeIfAbsent会在bucket为null的时候初始化一个ReservationNode来占位**，然后等待后面的**计算结果出来，再替换当前的占位对象**，而putVal会**synchorized这个对象**，并根据其hash值的正负来进行更新，遗憾的是ReservationNode的hash如果是-3，在putVal中没有处理过这种情况，**然后就一直for循环处理了**

很明对于单线程的HashMap来说，computeIfAbsent 并不存在以上问题。

















