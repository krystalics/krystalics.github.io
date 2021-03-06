#### Java.nio详解：

我们把之前会出现阻塞的I/O称之为Block-I/O ——简称BIO。一旦线程阻塞将会失去CPU的使用权，这再大规模访问量和有性能要求的情况下是不能接受的。虽然可以采用并发来保证不会对其他连接的响应终止，而且也采用线程池来尽量减少线程数，减小线程开销。

可是随着业务的发展，尤其是移动互联网的发展，一个大型的网站可能需要同时保证几百万的连接数，我们不可能在服务器中拥有如此多的资源，即使使用线程池。

##### 为什么要使用NIO？

NIO的创建目的是为了让Java程序员可以实现高速I/O而无需编写自定义的native代码，NIO将最耗时的I/O操作（即填充和提取缓冲区）转移回操作系统，因而可以极大地提高速度。

文中伊始我们就说过I/O是流方式处理数据，而NIO和它最大的区别就是 **数据打包和传输的方式** 。**NIO以块方式处理数据**

面向流的I/O系统一次一个字节地处理数据。一个输入流产生一个字节的数据，一个输出流消费一个字节的数据。为流式数据创建过滤器非常容易，链接几个过滤器，以便每个过滤器只负责单个复杂处理机制的一部分，这样也是相对简单的。缺点是，面向流的I/O通常非常慢

面向块的I/O系统以块的形式处理数据。每一个操作都在一步中产生或者消费一个数据块，所以它比流式的要更快，但是面向块的I/O比较复杂和难度。

都说NIO比IO更快，但是作为新时代的有为青年我偏偏不信这个邪。于是做了个测试，用两个方式读写70多M的文件看看谁更快：以下是测试数据

| 复制77.9M的CSV文件，单位ms | 第一次 | 第二次 | 第三次 | 第四次 |
| -------------------------- | ------ | ------ | ------ | ------ |
| IO                         | 3031   | 2388   | 1724   | 1833   |
| NIO                        | 1830   | 2018   | 1408   | 1287   |

这只是个定性的小demo，说明NIO的性能确实比IO要好一些。

##### NIO的核心：Buffer，Channel

几乎每个I/O操作都要使用它们。Channel是对原来I/O包中流的模拟，到任何目的地（或来自任何地方）的所有数据都必须通过一个Channel对象。而Buffer实质上是一个容器对象，发送给一个通道的所有对象都必须首先放在缓冲区；同样的从通道中读取的任何数据都要读到缓冲区中

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/8.png?raw=true">

Buffer是一个对象，包含一些要写入或者刚读出的数据，在NIO中加入Buffer对象，体现了与原I/O的一个重要区别。在面向流的I/O中，数据将直接写入或者读到Stream对象中。

在NIO库中，所有数据都是用缓冲区处理的。牢牢记住这一点，缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。缓冲区是指上是一个数组通常它是一个字节数组，也可能是其他数据类型的数组，

最常用的缓冲区类型是 ByteBuffer，一个ByteBuffer可以在其底层字节数组上进行get/set操作（即字节的获取和设置）事实上每一个基本类型都有一个对应的缓冲区

- ByteBuffer
- CharBuffer
- ...

每一个Buffer类都是Buffer接口的一个实例。除了ByteBuffer，其他Buffer操作完全一样，只是处理的数据类型不一样。因为大多数标准I/O都使用ByteBuffer，所以它具有所有共享缓冲区操作以及一些特有的操作。

如下面展示了 FloatBuffer的一个例子：

```java
public static void main(String args[]) throws Exception {
    FloatBuffer buffer = FloatBuffer.allocate(10);

    for (int i = 0; i < buffer.capacity(); ++i) {
        float f = (float) Math.sin((((float) i) / 10) * (2 * Math.PI));
        buffer.put(f);
    }

    buffer.flip();

    while (buffer.hasRemaining()) {
        float f = buffer.get();
        System.out.println(f);
    }
}
```

##### 说完了什么是Buffer，就轮到Channel了

Channel是一个对象，可以通过它读取和写入数据。拿NIO与原来的I/O做个比较，Channel就是流。从上面的图中可知与Channel打交道的是Buffer。与流不同的是Channel是双向的，而流是单向的（如InputStream，OutputStream），Channel可以用于读和写或者一边读一边写

练习1：从文件中读取一些数据，如果使用原来的I/O，我们需要创建一个FileInputStream并从它那里读取，而在NIO中；我们首先从FileInputStream中获取一个Channel对象，然后使用这个通道来读取数据。而在NIO系统中，任何时候执行一个读操作，都是从缓冲区中读取。所以步骤如下：

- 从FileInputStream中获取Channel对象
- 创建Buffer
- 将数据从Channel读到Buffer中

```java
public static void read() throws Exception {
    FileInputStream fin = new FileInputStream("C:\\Users\\12857\\Desktop\\iotest.txt");
    FileChannel channel = fin.getChannel();
    ByteBuffer byteBuffer = ByteBuffer.allocate(1024); //创建一个1K大小的缓冲区
    channel.read(byteBuffer); //将数据从通道读到缓冲区中
  }

public static void write() throws Exception {
    FileOutputStream fout = new FileOutputStream("C:\\Users\\12857\\Desktop\\iotest.txt");
    FileChannel channel = fout.getChannel();
    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
    byte[] message = {'S', 'o', 'm', 'e', ' ', 'b', 'y', 't', 'e', 's'};
    for (int i = 0; i < message.length; i++) {
      byteBuffer.put(message[i]);
    }
    byteBuffer.flip();
    channel.write(byteBuffer);
  }
```

我们不需要告诉通道要读或者写多少数据到缓冲区中，每一个缓冲区都有一个复杂的内部统计机制，它会跟踪已经读了多少数据以及还有多少空间可以容纳更多的数据。

读和写我们都已经介绍过了，那么边度边写是怎么回事呢？？

- 首先创建一个Buffer，然后从源文件中将数据读到这个缓冲区
- 然后将缓冲区写入目标文件
- 不断读写，直到源文件结束

```java
static public void main( String args[] ) throws Exception {
    
    String infile = "C:/users/12857/desktop/source.txt";
    String outfile = "C:/users/12857/desktop/dest.txt";

    FileInputStream fin = new FileInputStream( infile );
    FileOutputStream fout = new FileOutputStream( outfile );

    FileChannel fcin = fin.getChannel();
    FileChannel fcout = fout.getChannel();

    ByteBuffer buffer = ByteBuffer.allocate( 1024 );

    while (true) {
      buffer.clear(); //清除上一波缓存

      int r = fcin.read( buffer ); //读数据到缓存中，并获得文件指针位置

      if (r==-1) { //如果到了文件结尾
        break;
      }

      buffer.flip();  //让缓冲区可以将新读入的数据写入另一个通道，就像开闸了一样

      fcout.write( buffer );
    }
  }
```



##### 缓冲区内部细节：即Buffer的设计细节

NIO中两个重要的缓冲区组件：状态变量和访问方法（accessor）

状态变量是上文中提到的内部统计机制的关键。每一个读写的操作都会改变缓冲区的状态，通过记录这些变化，缓冲区就能够管理自己的资源。具体如何管理的见下面的细节

##### 状态变量

Buffer用三个属性表示缓冲区的状态

- position：跟踪已经写了多少数据，实际上就是数组的指针位置
- limit：从缓冲区写入通道，表示缓冲区还剩多少数据。没有开启缓冲区到通道的时候，limit=capacity，开始写出的时候limt=position。
- capacity：缓冲区的最大容量
- mark：是个备忘位置，使用mark()方法令 mark=position，当reset()的时候会设定position=mark，有些时候会用到。

状态管理如下图所示：从一个Channel通过Buffer到另一个Channel

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/9.png?raw=true">

##### 访问方法

到目前为止，我们只是使用缓冲区将数据从一个通道转移到另一个通道。然而，程序经常需要直接处理数据。例如：可能需要将用户数据保存到磁盘。在这种情况下，必须将这些数据直接放入缓冲区，然后用通道将缓冲区写入磁盘。还有就是需要从磁盘读取用户数据，在这种情况下，需要将数据从通道读到缓冲区，然后检查缓冲区中的数据。

这些例子都需要访问Buffer中的数据。

##### get()方法

以ByteBuffer为例，它有四个get()方法

- byte get() 获取当前position位置的单个字节
- ByteBuffer get(byte dst[])  里面调用的是get(dst,0,dst.length)也就是下一个get()方法，
- ByteBuffer get(byte dst[],int offset,int length) 从offset开始读取length个字节到dst[]中并返回ByteBuffer对象，
- byte get(int index) 获得index位置的字节

文章中对于第二第三个get()解释并不清楚，所以在jdk中找了出来。

```java
 public ByteBuffer get(byte[] dst, int offset, int length) {
     checkBounds(offset, length, dst.length);
     //remaining()=limit - position; 实际上就是缓冲区中还有多少数据
     if (length > remaining())  
         throw new BufferUnderflowException();
     int end = offset + length;
     for (int i = offset; i < end; i++)
         dst[i] = get();
     return this; //this就是 调用get()方法的ByteBuffer对象
 }
```

##### put()方法

ByteBuffer有5个put()方法

- ByteBuffer put(byte b) 往position位置写入一个字节
- ByteBuffer put(byte src[]) 写入一个数组，put(src,0,src.length)
- ByteBuffer put(byte src[],int offset,int length) 从offset开始写入length个src[]中的数据
- ByteBuffer put(ByteBuffer src) 将其他ByteBuffer写入
- ByteBuffer put(int index,byte b) 在index位置写入一个字节

还有一些其他的方法可以访问它，比如

- getByte()  getByte(int index)      putByte() putByte(int index)
- getChar() getChar(int index)      putChar() putChar(int index)
- getShort() getShort(int index)    putShort() putShort(int index)
- ....

事实上都是一对，一个相对位置 position，一个绝对位置 index

##### 如何使用缓冲区呢？？？只要一个内部循环

```java
while(ture){
    buffer.clear(); //position = 0;  limit = capacity;
    int r=fcin.read(buffer);
    
    if(r==-1){
        break;
    }
    buffer.filp(); //设置 limit=position, position=0 
    fcout.write(buffer);
}
```

read()和write()调用得到了极大的简化，细节都由缓冲区内部解决了。

嗯经过漫长的篇幅，我们已经基本掌握了缓冲区日常工作所需要的大部分内容了。接下来是更加复杂的缓冲区分配，包装，分片。还会讨论一些NIO带给Java平台的一些新东西。如何创建不同类型的缓冲区达到不同的目的，如何保护数据不被修改——只读缓冲区，和直接映射到底层操作系统缓冲区的 直接缓冲区。还有最后的NIO中创建内存映射文件



##### 缓冲区分配和包装

我们看文中的代码可以看到，初始化ByteBuffer并不是new ByteBuffer，而是使用静态方法allocate()方法来分配缓冲区

```java
ByteBuffer buffer=ByteBuffer.allocate(1024);
```

allocate()方法分配一个具有指定大小的底层数组，并将它包装到一个缓冲区对象中，还可以将现有的数组转为缓冲区

```java
byte array[]=new byte[1024];
ByteBuffer buffer=ByteBuffer.wrap(array);
```

使用wrap()方法将一个数组包装为缓冲区，必须非常小心地进行这类操作。一旦完成包装，底层数据就可以通过缓冲区或直接访问。稍微探究一下allocate和wrap方法的原理

```java
public static ByteBuffer allocate(int capacity) {
    if (capacity < 0)
        throw new IllegalArgumentException();
    return new HeapByteBuffer(capacity, capacity);
}

public static ByteBuffer wrap(byte[] array,int offset, int length){
    try {
        return new HeapByteBuffer(array, offset, length);
    } catch (IllegalArgumentException x) {
        throw new IndexOutOfBoundsException();
    }
}
```

我们看JDK中的源码，都是new HeapByteBuffer，而HeapByteBuffer中的构造方法是调用的super()父类的构造方法，它的父类是 ByteBuffer，而ByteBuffer的构造方法也是调用super()设置那些状态，

```java
ByteBuffer(int mark, int pos, int lim, int cap,byte[] hb, int offset){
    // package-private
    super(mark, pos, lim, cap); //初始化状态值
    this.hb = hb;  //这里的hb就是我们传进去的 byte数组
    this.offset = offset;
}

Buffer(int mark, int pos, int lim, int cap) {       // package-private
    if (cap < 0)
        throw new IllegalArgumentException("Negative capacity: " + cap);
    this.capacity = cap;
    limit(lim); //设置limit
    position(pos); //设置position
    if (mark >= 0) {
        if (mark > pos)
            throw new IllegalArgumentException("mark > position: ("
                                               + mark + " > " + pos + ")");
        this.mark = mark;
    }
}
```

##### 缓冲区分片

slice()方法根据现有缓冲区创建一种子缓冲区，也就是说创建一个新的缓冲区。新缓冲区与原缓冲区的一部分共享数据，子缓冲区也叫做片，片段。例子如下：

```java
ByteBuffer buf = ByteBuffer.allocate(10);
for (int i = 0; i < buf.capacity(); i++) {
    buf.put((byte) i);
}
//调整position和limit位置，截取3~7区间作为子缓冲区
buf.position(3);
buf.limit(7);
ByteBuffer slice = buf.slice();
```

别看上面的slice是一个新的ByteBuffer，但实际上slice和buf是共享那部分数据的，做个试验就知道了。程序接上

```java
for (int i = 0; i <slice.capacity() ; i++) {
    byte b=slice.get(i);
    b*=11;
    slice.put(i,b);
}

buf.clear(); //让 position和limit归位
for (int i = 0; i <buf.capacity() ; i++) {
    System.out.print(buf.get()+" ");
}
// 0 1 2 33 44 55 66 7 8 9
```

我们在程序中是改变slice中的数据，但是遍历buf中的数据，发现对应部分也发生了变化。所以它们这部分数据是共享的。子缓冲区是为了方便处理缓冲区中的部分数据，而不用我们自己写方法进行处理。也更加通用和方便。

##### 只读缓冲区

如名不能写入的缓冲区，是原来缓冲区转换过来的。用于处理不能被修改的关键数据。，**只需要调用asReadOnlyBuffer()方法，就可以返回一个和原缓冲区完全一样的Buufer，但是它是只读的**。

当我们将缓冲区传递给某个对象的方法时，我们不知道该方法会不会修改缓冲区中的数据，所以传入只读缓冲区可以保证数据的安全性。对于只读缓冲区它是不能逆转为可写缓冲区的。

```java
 ByteBuffer readonly= buf.asReadOnlyBuffer();
 readonly.put(2,(byte) 3); //会报错，java.nio.ReadOnlyBufferException
```

知道了结果，那我们追本溯源一下：调用asReadOnlyBuffer()会生成一个新的HeapByteBuffer()，然后设置`isReadOnly=true;` ，这样通过想通过put()方法写入数据的时候就会`throw new ReadOnlyBufferException();`

```java
public ByteBuffer asReadOnlyBuffer() {

    return new HeapByteBufferR(hb,
                               this.markValue(),
                               this.position(),
                               this.limit(),
                               this.capacity(),
                               offset);

}
protected HeapByteBufferR(byte[] buf,int mark, int pos, int lim, int cap, int off){
    super(buf, mark, pos, lim, cap, off);
    this.isReadOnly = true;
}
public ByteBuffer put(ByteBuffer src) {
    if (src == this)
        throw new IllegalArgumentException();
    if (isReadOnly())
        throw new ReadOnlyBufferException();
    int n = src.remaining();
    if (n > remaining())
        throw new BufferOverflowException();
    for (int i = 0; i < n; i++)
        put(src.get());
    return this;
}
```

##### 直接与间接缓冲区

直接缓冲区是为加快I/O速度，而以一种特殊方式分配其内存的缓存区。关于Sun公司给出的描述是这样的：

> 给定一个直接字节缓冲区，Java虚拟机将尽最大努力直接对它执行本机I/O操作。也就是说它会在每一次调用底层操作系统的本机I/O操作之前或者之后，尝试将缓冲区的内容拷贝到一个中间缓冲区（或者从一个中间缓冲区拷贝数据）

```java
ByteBuffer buffer = ByteBuffer.allocateDirect( 1024 );
```

代码也没多大变化，就是来了个allocateDirect()。让我们追踪一下，首先看一下它的继承关系

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/java/img/10.png?raw=true">

```java
public static ByteBuffer allocateDirect(int capacity) {
    return new DirectByteBuffer(capacity);
}

 DirectByteBuffer(int cap) {   // package-private
     super(-1, 0, cap, cap);
     boolean pa = VM.isDirectMemoryPageAligned();
     int ps = Bits.pageSize();
     long size = Math.max(1L, (long)cap + (pa ? ps : 0));
     Bits.reserveMemory(size, cap);

     long base = 0;
     try {
         base = unsafe.allocateMemory(size);
     } catch (OutOfMemoryError x) {
         Bits.unreserveMemory(size, cap);
         throw x;
     }
     unsafe.setMemory(base, size, (byte) 0);
     if (pa && (base % ps != 0)) {
         // Round up to page boundary
         address = base + ps - (base & (ps - 1));
     } else {
         address = base;
     }
     cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
     att = null;
 }
```

可以很清楚的看到，它是由unsafe和VM来掌控的以及处理位数据的Bits。VM.java在JDK1.2之后已经被删掉了，但是VM.class文件还保留着，兼容以前的版本。

> The following methods used to be native methods that instruct the VM to selectively suspend certain threads in low-memory situations. 
>
> They are inherently dangerous and not implementable on native threads. 
>
> We removed them in JDK 1.2. 
>
> The skeletons remain so that existing applications that use these methods will still work.

大致意思是：下面这些本地方法是可以让VM在内存不足的情况下有选择的挂起某些线程。它们非常危险并且不能再本地线程中实现，所以在JDK1.2删掉了，但是.class文件任然保留用于兼容以前的应用。

总的来说，直接缓冲区就是让别的东西给它让道，就是优先级更高。JVM会配合它工作。



##### 内存映射文件I/O

内存映射文件I/O时一种读和写文件数据的方法，它可以比常规的基于流或者基于Channel的I/O快得多。它是通过使文件中的数据变为内存数组的内容来完成的（内存的读写速度大大超过可以IO设备），听起来像是将整个文件读到内存中，但是实际上只有文件中实际读取或者实际写入的部分才会被送入内存。

现代操作系统一般根据需要，**将文件的部分映射为内存的部分，从而实现文件系统**。Java内存映射机制不过是在底层操作系统中可以采用这种机制时，提供了对该机制的访问。

尽管创建内存映射文件相当简单，但是向他写入可能是危险的。仅只是改变数组的单个元素这样的简单操作，就可能直接修改磁盘上的文件，修改数据与保存到磁盘是没有分开的。

来个例子：将FileChannel的前1024个字节映射到内存中，使用FileChannel.map()方法

```java
MappedByteBuffer mbb=fileChannel.amp(FileChannel.MapMode.READ_WRITE,0,1024);
```

MappedByteBuffer是ByteBuffer的子类，操作大致和ByteBuffer一样。操作系统将会在需要的时候负责执行映射

#### 分散和聚集

**分散/聚集 I/O**是使用多个而不是单个缓冲区来保存数据的读写。一个分散的读取就像一个常规的通道读取，只不过将它的数据读到一个缓冲区数组而不是单个缓冲区中；同样的，一个聚集写入是向缓冲区数组而不是当个缓冲区写入数据

分散/聚集 I/O对于将数据流划分为单独的部分很有用，这有助于实现复杂的数据格式。通道可以有选择地实现两个新接口：

ScatteringByteChannel： 分散读取

- long read(ByteBuffer[] dsts)
- long read(ByteBuffer[] dsts,int offset,int length)

GatheringByteChannel：聚集写入

- long write(ByteBuffer[] srcs)
- long write(ByteBuffer[] srcs,int offset,int length)

读取一个缓冲区数组。在分散读取中，通道依次填充每个缓冲区，填满一个缓冲区后就开始填充下一个。某种意义上来说，缓冲区数组就是一个大缓冲区。如果说单个缓冲区是一辆货车，那么缓冲区数组就是一个车队。文章给我们提供了一个例子，用来说明他的应用

```java
public class UseScatterGather {
    
  static private final int firstHeaderLength = 2;
  static private final int secondHeaderLength = 4;
  static private final int bodyLength = 6;

  static public void main(String args[]) throws Exception {

    int port = 10086; //占用10086端口
// 启动网络服务，绑定10086端口
    ServerSocketChannel ssc = ServerSocketChannel.open();
    InetSocketAddress address = new InetSocketAddress(port);
    ssc.socket().bind(address);

    int messageLength =
        firstHeaderLength + secondHeaderLength + bodyLength;
// 创建3个 Buffer 分别用于存储  firstsHeader SecondHeader Body
    ByteBuffer buffers[] = new ByteBuffer[3];
    buffers[0] = ByteBuffer.allocate(firstHeaderLength);
    buffers[1] = ByteBuffer.allocate(secondHeaderLength);
    buffers[2] = ByteBuffer.allocate(bodyLength);
      
// 等待客户端请求，线程进入阻塞，
//其中SocketChannel继承了 ScatteringByteChannel和GatheringByteChannel
    SocketChannel sc = ssc.accept();

    while (true) {

      // Scatter-read into buffers
      int bytesRead = 0;
      while (bytesRead < messageLength) {
        long r = sc.read(buffers); //一次读三个缓冲区
        bytesRead += r;

        System.out.println("r " + r);
        for (int i = 0; i < buffers.length; ++i) {
          ByteBuffer bb = buffers[i];
          System.out.println("b " + i + " " + bb.position() + " " + bb.limit());
        }
      }

      // Process message here

      // Flip buffers
      for (int i = 0; i < buffers.length; ++i) {
        ByteBuffer bb = buffers[i];
        bb.flip();
      }

      // Gather-write back out
      long bytesWritten = 0;
      while (bytesWritten < messageLength) {
        long r = sc.write(buffers); //一次写三个缓冲区
        bytesWritten += r;
      }

      // Clear buffers
      for (int i = 0; i < buffers.length; ++i) {
        ByteBuffer bb = buffers[i];
        bb.clear();
      }

      System.out.println(bytesRead + " " + bytesWritten + " " + messageLength);
    }
  }
}
```

##### 那么我们把缓冲区做大一点能代替缓冲区数组吗？

例如，可能在编写一个使用消息对象的网络应用程序，每个消息被划分为固定长度的头部和固定长度的正文。可以创建一个刚好容纳头部的缓冲区，以及另一个刚好容纳正文的缓冲区。当您将它们放入一个数组中并使用分散读取来向它们读入消息时，头部和正文将整齐地划分到两个缓冲区中。

这是由数据本身的划分决定的，使用缓冲区数组可以对应获得头部和正文。便于处理



#### 文件锁定

要知道不管是IO还是NIO都是要和文件打交道的，所以衍生出来的文件锁定知识真的是不得不了解一下。

实际上，文件锁定并不像名字这么唬人，它并不是将文件锁起来不让程序访问。它就像一个Java对象锁——根据获取的锁的种类来处理不同业务。还可以选择锁定整个文件或者部分

如果获取的是一个排它锁：那么其他人就不能获得同一个文件或者文件部分上的锁。共享锁的话：就可以访问。例如：可能临时锁定一个文件以保证特定的写操作称为原子的，并且不受其他程序的干扰，这就需要排他锁。

大多数操作系统都提供了文件系统锁，但是它们采用的锁可能各不相同；有些实现提供了共享锁，另一个部分提供排它锁。还有些操作系统就真的锁定文件不让访问，

##### 介绍了概念之后，我们来看下nio怎么锁定文件呢？

```java
RandomAccessFile raf=new RandomAccessFile("usefilelocks.txt","rw");
FileChannel fc=raf.getChannel();
FileLock lock=fc.lock(start,end,false);
//...
lock.release(); //记得操作完之后需要 释放锁哟
```

让我们看看lock的细节，开始锁的位置和长度以及是否共享。按照上面的程序，文件的10~30位置被排他锁锁定，不共享数据。

```java
public abstract FileLock lock(long position, long size, boolean shared) throw IOException;
```

文件锁定是一个复杂的操作，特别是考虑到不同操作系统锁是以不同方式实现锁这一事实。怎么保持代码在不同操作系统都能够正常运行呢？

- 只是用排他锁
- 将所有锁都视为劝告式的 advisory

到目前为止，我们介绍的都是NIO的同步特性，并没有涉及到异步I/O。接下来就要讲NIO中关于异步的核心。

#### 连网和异步I/O

连网是学习异步I/O很好的基础，NIO中的连网并没有和其他操作有大的差异，还是依赖Channel，Buffer。还是使用InputStream和OutputStream来获取Channel

##### 异步I/O

它是一种没有阻塞地读写数据的方法，通常，在代码进行read()调用时，线程会进入阻塞直到有可读数据为止。同样的write()也是陷入阻塞。

而异步I/O调用不会阻塞，这是它能打败阻塞I/O的重要原因。它会让程序 注册对特定I/O事件的关注——可读数据到达，新的套接字连接等等。而发生这些事情的时候，系统会通知哪些注册的程序。

没有了阻塞之后，它的优势就是允许同时根据大量的输入和输出执行I/O。同步程序通常要求助与轮询，或者创建许多的线程来处理大量连接，使用异步I/O，可以监听任何数量的Channel上的事件，不用额外的线程和轮询。

接下来通过一个例子，也是本文最后的部分了。看到这的同学们都坚持一下吧：能够监听多个端口并且处理来自这些端口的连接，而且是单线程的完成所有工作的server程序。

在看例子之前，要先知道nio异步的核心对象：**Selector**，它就是程序注册对各种I/O事件关注的地方，也是由Selector对象通知程序。

##### 所以，我们需要做的第一件事就是创建一个Selector：

```java
Selector selector=Selector.open();
```

Selector实际上由SelectorProvider提供，具体代码就同学们自己看代码吧，反正我是看不懂。接着就是对不同Channel对象调用register()方法，进行注册事件，第一个参数就是Selector，

我们需要创建一个Channel，而在网络服务中一般都是创建一个ServerSocketChannel，并注意绑定地址

##### 打开一个ServerSocketChannel

```java
ServerSocketChannel ssc = ServerSocketChannel.open();
ssc.configureBlocking( false ); //配置为非阻塞
ServerSocket ss = ssc.socket();
InetSocketAddress address = new InetSocketAddress( ports[i] );
ss.bind( address );
```

##### 有了Selector和Channel，接下来就是进行注册

```java
SelectionKey key = ssc.register( selector, SelectionKey.OP_ACCEPT );
```

这里 SelectionKey.OP_ACCEPT表示它想要监听的事件为 accept()事件。即在新连接建立时所发生的事件，这也是唯一适合ServerSocketChannel的事件类型。

返回的SelectionKey表示这个Channel在该Selector上注册，当某个Selector通知程序传入事件时，它是通过提供该事件的SelectionKey来进行的。同样取消注册也是通过它。

对每个端口都创建一个ServerSocketChannel并绑定和注册之后，进入主程序

```java
//造成线程阻塞，直到至少有一个已经注册的事件发生
int num = selector.select(); //如果发送多个时间，就返回发生事件的数量

// 获得在该Selector中注册的所有 SelectionKey
Set selectedKeys = selector.selectedKeys();
Iterator it=selectedKeys.iterator();

while(it.hasNext()){
	SelectionKey key = (SelectionKey)it.next();
    // .
}
```

##### 监听新连接

程序执行到现在，我们只是注册了事件，还要确认发生的事件类型是什么，以下程序接上面的循环中，key.readOps()返回一个整数和 SelectionKey进行与运算

```java
if((key.readOps()&SelectionKey.OP_ACCEPT)==SelectionKey.OP_ACCEPT){
    //...
}
```

readOps()方法告诉我们该事件是新的连接。

##### 接受新连接

因为我们知道这个服务器上有个传入连接在等待，所以可以安全的接受它，不用担心accept()会造成线程阻塞。

```java
ServerSocketChannel ssc=(ServerSocketChannel) key.channel();
SocketChannel sc=ssc.accept();
sc.configureBlocking(false); //新连接配置为 非阻塞
SelectionKey newKey=sc.register(selector,SelectionKey.OP_READ;
```

通过key.channel()方法获得注册事件的Channel，并让该Channel使用accept() 与新连接相对应。而且由于该连接的目的是为了读取来自套接字的数据，还必须将SocketChannel也注册到Selector上，参数是OP_READ，说明SocketChannel用于读取而不是接受新连接

##### 然后是删除处理过的SelectionKey

在处理SelectionKey之后，我们几乎可以返回主循环了，但是必须将处理过的SelectionKey从选定的集合中删除，如果没有删除，它将会在下次循环再次激活。使用 it.remove()即可。现在我们可以返回主循环并接受从一个套接字中传入的数据（或者传入I/O事件），程序见下面一个片段

##### 传入I/O

当来自一个套接字的数据到达时，它会触发一个I/O时间。这会导致在主循环汇总调用Selector.select()，并返回一个或多个I/O事件，SelectionKey被标记为OP_READ，这段程序接上面监听新连接的if

```java
else if((key.readyOps()&SelectionKey.OP_READ)==SelectionKey.OP_READ){
    SocketChannel sc=(SocketChannel)key.channel();
   
    // Echo data
      int bytesEchoed = 0;
      while (true) {
        echoBuffer.clear(); //echnoBuffer是类的实例变量，初始化为1024字节
		//从通道读取数据到缓冲区
        int r = sc.read( echoBuffer );

        if (r<=0) {
          break;
        }

        echoBuffer.flip();
       // 从缓冲区读到通道，SocketChannel是双向的，可读可写
        sc.write( echoBuffer );
        bytesEchoed += r;
      }

      System.out.println( "Echoed "+bytesEchoed+" from "+sc );

      it.remove(); //处理完删除掉该SelectionKey
    }

}
```

与之前一样，我们取得发生I/O事件的通道并处理它。在本例中，我们只希望从套接字中读取数据并马上将它发送出去。

例子基本演示完毕，实际情况中很可能是多线程并且创建一个线程池来负责I/O事件处理。

[本文中的大部分代码都在这里](https://www.ibm.com/developerworks/cn/education/java/j-nio/nio-src.zip) 

还是总结一下最后一部分内容把：

- 打开一个Selector
- 然后生成一些ServerSocketChannel，注册进Selector，监听accept事件即连接事件。用SelectionKey沟通
- 然后进入阻塞，等待事件发生
- 当事件发生之后，确定发生事件的数量，并遍历Selector中的SelectionKey
- 确立事件和SelectionKey中注册的事件是否吻合，在这里就是确定SelectionKey中是否为OP_ACCEPT，如果是就新来的连接，就将ServerSocketChannel从key中取出来，并从中获取读写的通道SocketChannel，配置完非阻塞之后就监听这个读写事件，**因为即使连接建立了，数据也没有第一时间送过来，为了不造成阻塞所以就监听了这个I/O事件**，将这个连接事件的key从迭代器中删除，避免再次处理它。
- 如果有套接字的数据到达，就会触发I/O事件。一般处理完I/O事件会把该事件的监听从key表中删除，然后继续添加一个连接事件的监听。程序中并没有，有点奇怪，难道是一锤子买卖？
- 处理完一轮之后，又陷入阻塞状态，直到有事件发生。

我知道最后一部分大部分同学看着都是懵逼的，连我这个认真做笔记，认真学习的学霸写下来都是一脸蒙蔽的。别急（其实是对我自己说的），让我们最后花10分钟理解一下。作者真的不想在这个13000字的文章里纠结下去了，从前天开始准备，昨天动手写笔记，今天下午还在忙碌。感觉人生到达了巅峰。

最后的最后，BIO就像一队人在排队买东西，第一个人没带钱，回去取钱了，整个队伍就停滞在这里：

NIO就会让第二个人先买，如果第二个人没有准备好，就第三个，，一直循环下去

AIO是让所有人先拿着菜单，点单，选好的再上前买单

参考文章：

[NIO入门](https://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html#icomments)                  

[Java NIO浅析](https://tech.meituan.com/2016/11/04/nio.html) 