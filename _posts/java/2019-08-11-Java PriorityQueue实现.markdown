最近有一个业务场景需要取得top k的数据，内存有限制。所以特别查了堆排序的实现，原来Java自带了一个小顶堆PriorityQueue，所以我决定先熟悉它的原理，然后使用它完成任务(使用相对来说比较简单)。

参考：

[优先级队列 PriorityQueue 的实现原理](https://www.jianshu.com/p/c577796e537a)

[Java PriorityQueue中二叉堆的与原理](https://www.jianshu.com/p/b308d23f3775)



##### 优先级队列PriorityQueue

java 1.5中引入，是一个基于优先堆的无界队列，这个优先队列中的元素可以按默认的自然排序或者提供Comparator在队列实现实时排序。

需要注意的是它不允许空值，不支持non-comparable(不可比较的对象)，所以没有继承Comparable的类想要排序就需要构造一个Comparator了。和ArraysList一样虽然大小是不限制(long类型的表示极限)的但是可以初始化一个容量，值的注意的是不能往里加null

因为它增加元素的方法中明确抛出了异常(有两个方法可以增加元素add(),offer()，其中add调用的offer方法，都会返回一个布尔类型表示增加成功 )

```java
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1); //增大是向左移一位，扩大两倍
    size = i + 1;
    if (i == 0)
        queue[0] = e;
    else
        siftUp(i, e);
    return true;
}
```

PriorityQueue是非线程安全的，用于多线程环境的是PriorityBlockingQueue。

直接来个例子，就是求topk的，说一下它是如何使用的：下面代码是生成100个随机数中获得10个靠前的数。

```java
public class TopK {
    private PriorityQueue<Integer> queue = new PriorityQueue<Integer>();
    private int K = 10; 

    public void add(int i) {
        if (queue.size() < K) {
            queue.add(i);
        } else {
            // Min heap
            int min = queue.peek();
            if (i > min) {
                queue.poll();
                queue.add(i);
            }
        }
    }

    public void print() {
        while (!queue.isEmpty()) {
            Integer i = queue.poll();
            System.out.println(i);
        }
    }

    public static void main(String[] args) {
        Random r = new Random();
        TopK t = new TopK();

        for (int i = 0; i < 100; i++) {
            t.add(r.nextInt(100));
        }

        t.print();
    }
}
```

大概介绍了一下PriorityQueue，知道是什么东西怎么用之后就可以开始接触原理啦。

大顶堆和小顶堆都是通过完全二叉树实现的：大顶堆，意思是根节点的值比子节点的值要大，小顶堆相反。而完全二叉树的节点位置可以通过公式推导出来，所以这就意味着可以用数组存储这颗树。

- 父节点位置： (curNode-1)/2
- 左子节点位置：curNode*2+1
- 右子节点的位置：curNode*2+2

下面用图演示一下 小顶堆是如何插入元素和删除元素的，至于代码就是JDK中PriorityQueue的代码了、

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/19.png?raw=true">

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/20.png?raw=true">

直接照着jdk中的代码，抠出了一个简单的优先队列，不实现自己构造Comparator的比较

```java

// 这里实现一个简单的PriorityQueue
public class MyPriorityQueue<T> {
    private Object[] queue = new Object[16]; //初始容量为16
    private int size = 0; //表示当前队列的元素个数

    private boolean offer(T t) {
        if (t == null) throw new NullPointerException();
        int i = size++; //获得当前队列的size
        if (i >= queue.length) {
            grow(); //如果size即将超过容量，扩容
        }
        if (i == 0) {
            queue[0] = t;
        } else {
            //在其他位置增加元素需要考虑到整个树的结构
            siftUp(i, t);
        }
        return true;
    }

    @SuppressWarnings("unchecked")
    private void siftUp(int i, T t) {
        Comparable<? super T> key = (Comparable<? super T>) t; //让继承了Comparable的对象可以比较
        while (i > 0) { //直到有节点比新增的元素要小
            int parent = (i - 1) >>> 1; //向右移一位，不带符号
            Object o = queue[parent];
            if (key.compareTo((T) o) >= 0) { //如果t大于该位置的父节点，就说明增加结束
                break;
            }
            queue[i] = o; //如果，新增的元素值小于父节点，将父节点下移
            i = parent; //重新去找父节点的父节点
        }
        queue[i] = key; //将该节点赋值新增的元素
    }

    @SuppressWarnings("unchecked")
    private T poll() { //把堆顶元素即最小的元素取出
        if (size == 0) return null;
        int s = --size;
        T res = (T) queue[0];
        T x = (T) queue[s];
        queue[s] = null;
        if (s != 0) {
            siftDown(0, x); //从0开始迭代,到末尾x
        }
        return res;
    }

    /*
    * 将根节点推出来后，需要重新构建一个小顶堆，选择根节点的时候就看 原根节点的左右节点谁更小了
    * 用最后一个元素代替原根节点，进行结构调整
    * */
    @SuppressWarnings("unchecked")
    private void siftDown(int i, T t) {
        Comparable<? super T> key = (Comparable<? super T>) t; //让继承了Comparable的对象可以比较
        int half = size >>> 1; //因为需要计算子节点，到时候会乘2
        while (i < half) {
            int left = (i << 1) + 1;//左子节点
            Object o = queue[left];
            int right = left + 1;
            if (right < size && ((Comparable<? super T>) o).compareTo((T) queue[right]) > 0) {
                o = queue[left = right]; //如果左节点的值比右节点的大，就将左节点的值和指针都赋值为右节点的，右节点就是新的根节点
            }
            if (key.compareTo((T) o) < 0) {
                break;
            }
            queue[i] = o;  //l就是新的根节点
            i = left;
        }
        queue[i] = key;
    }

    @SuppressWarnings("unchecked")
    private T peek() {
        return size == 0 ? null : (T) queue[0];
    }

    private void grow() {
        int newSize = queue.length << 1;
        queue = Arrays.copyOf(queue, newSize);
    }

    private int size() {
        return size;
    }

    public static void main(String[] args) {
        PriorityQueue<Integer> pq = new PriorityQueue<>();
        MyPriorityQueue<Integer> queue = new MyPriorityQueue<>();
        for (int i = 0; i < 100; i++) {
            queue.offer(100 - i);
        }
        for (int i = 0; i < 100; i++) {
            System.out.println(queue.poll());
        }

    }
}

```

















