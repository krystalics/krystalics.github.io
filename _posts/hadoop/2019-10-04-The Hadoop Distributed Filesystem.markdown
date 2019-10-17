from 《Hadoop：The Definitive Guide》

---

#### The Hadoop Distributed Filesystem

当数据量不断增长至超过一个机器能够存储的量之后，将数据划分成更小的块就成了必要的需求。**而管理一个网络中多个机器的文件系统被叫做 *distributed filesystem* 分布式文件系统 DFS**，它们是以网络为基础所以比单个磁盘文件系统要更加复杂。其中最大的挑战就是使该文件系统能够承受节点故障而不使数据丢失。

Hadoop的分布式文件系统也叫HDFS，它只是Hadoop中的一个比较普遍的DFS，还有比如HBase，Hive等支持不同的业务场景。

##### The Design of HDFS

HDFS就是设计来存储流数据访问模式的大文件的文件系统，运行在商用的硬件集群上。特点如下：

- Very large files：hundreds of **megabytes**,**gigabytes** or **terabytes**。而现在的Hadoop集群一般都存储 PB 级别的数据
- Streaming data access：HDFS采用的是最有效率的处理模式 **write-once , read-many-times** 。dataset通常都是从不同的数据源中拷贝的，然后对数据集进行各种分析，每次分析都可能涉及到很大的数据量，因此读取整个数据集的时间比读取第一条记录的延迟更重要
- Commodity hardware：Hadoop不需要特别的昂贵设备，只需要便宜的商用设备即可，HDFS的设计旨在出现节点故障的时候不会给用户带来困扰，继续执行程序

以上是HDFS的优点，但是它的**缺点**同样明显：**不能即时查询**(通常需要花费几分钟至更多)，对于**大量的小文件处理速度慢**，**不支持随机写与并发写入(这里是Multiple writers ，我的理解是并发写或者并行写)。**

- Low-latency data access   work not well：HDFS是做大规模数据处理的，所以十几毫秒的响应速度是做不到的。HBase可以即时响应
- Lots of files work not well ：**因为namenode存储文件系统的元信息，文件数量被namenode的内存限制。**根据经验 每个文件，目录和block都占150字节左右。所以如果有1百万个文件，每个文件占据一个block，最少需要300MB的内存。**即使可以存储百万级别的文件，当文件数上亿之后就不行了**。
- Multiple writers ，arbitrary file modifications ：HDFS只允许append 的写，不支持multiple writers与随机写



#### HDFS Concepts

##### Blocks

block这个概念其实不是HDFS独有的，在**磁盘中也有block这个概念：它是指磁盘中读写一次最少的数据量**。所以单个磁盘的文件系统就是基于这个基础优化，每次读写都是 一个block的整数倍来加快读写速度，通常一个block为512bytes，对于开发人员来说这个大小通常是透明的，但也有对其维护的**工具**可以调整磁盘block的大小，比如 *df* 和 *fsck* 

而HDFS中的block通常大小在128MB，文件在HDFS中通常都被划分成固定的 block-sized chunks 即分为数个block大小的块分别存储在独立的单元(这里的单元是指磁盘，估计)

像原本磁盘中的block，即使我们读的数据没有到512Bytes，整个block的大小也还是512字节，不会是数据的原始大小 比如1字节。HDFS则不然，它的大小和原始数据是一致的，只不过上限是128MB。

##### 为什么HDFS中的Block这么大呢？

和磁盘中的block概念相比，它这么大的原因就是为了减少 seek time 。要知道在传输速率一致的情况下，寻找时间越短，整体的IO时间就越短，块数越少 seek time自然越小。

**假设10ms的seek time ，传输速率100MB/s 这样要使seek time 占传输时间的1%，就必须让block的大小为100MB。**所以经过试验128MB会是比较通用的结果。

block这个概念对分布式文件系统有很多好处：

- **文件能够比单个磁盘还大**，因为是分块分节点存储的
- 使用block这个概念而不是file，**简化了存储子系统**，简单的系统出现故障容易修复，使用block存储数据，**简化了存储管理**(因为**block是固定的大小，很容易计算出一个磁盘能有多少个**block)并且**分离出文件的元信息进行集中管理**(比如文件权限等信息可以不存储在block中，block只有数据)
- block能够很方便的进行**备份**来提供**容错性和可用性**，为了应对可能的异常或者错误，**每个block都有多个备份在不同的机器上(通常是3个备份)**。如果某个block因为程序错误或者机器错误不可用，可以从它的备份中读写。

`hdfs fsck / -files -blocks` 就会列出HDFS中每个文件的blocks。

##### Namenodes and Datanodes

HDFS集群中有两种采用 master-worker模型 的节点 ：一个namenode (master) 和 一些 datanodes (workers) 。namenode管理文件系统的元空间，**维护了文件系统树和树上所有文件目录的元信息**，这些数据都是存储在namenode的本地磁盘中，分为两种文件 ：namespace image 和 edit log。**namenode存储了某个文件所有blocks的位置**，并不存储该文件的数据，**因为这些信息都是在系统启动时从数据节点重建的**。

客户端范文文件系统就意味着和namenode与datanode打交道，客户端就相当于一个接口，开发人员就不必知道namenode和datanode的细节。

Datanodes是文件系统工作的主要场所，它们存储由namenode或者客户端交代的blocks，并且**会定期的向namenode汇报它存储的blocks。**

没有namenode 整个文件系统都不能使用，事实上如果机器上的namenode被废弃，文件系统上的所有的文件都会丢失，也就没有办法重建体系。所以，必须要保证namenode的弹性以及可用性。HDFS提供以下两种机制：

- 备份namenode中的元信息，Hadoop可以配置 将 namenode的持久状态写入多个文件系统，这些写入操作都是同步和原子的。通常的配置都是将数据写入本地磁盘和远程NFS中挂载
- 也可以增加一个 secondary namenode，主要用于定期的融合namespace image 和 edit log来防止edit log变得太大。secondary namenode 通常都是跑在其他独立的节点中，通常会保留融合后的namespace image，当namenode 出错时可以用于恢复数据。这里要注意 secondary namenode是要落后于primary的，所以恢复数据的时候难免会造成数据丢失。通常的做法都是 将namenode上的元数据文件挂载到NFS中，让后从NFS复制到secondary 并且将这个secondary晋升为primary。

##### Block Caching

通常datanode都是从磁盘的block中读取数据，但是对于访问特别频繁的会从缓存(datanode's memory，**在堆外off-heap的block cache中**)读取数据，**通常一个block cache只会在一个节点中，不过这也可以通过配置进行调节**。作业调度器 Job schedulers (用于MapReduce，Spark和其他框架) 可以通过在**有cached block的datanode上运行tasks时利用缓存数据** 达到提高读写性能的目的。

##### HDFS Federation

namenode的内存中存储了文件系统中的每一个文件和block的reference，意味着当一个集群有非常多文件时，内存大小就会成为瓶颈(因为，这些文件的元信息大小之和 可能超过了当个节点的内存大小)。**HDFS 在2.x的系列中允许集群通过增加namenode的节点数量，且每个namenode只管理一部分的元信息（不像是1.x时需要管理所有的元信息）**。比如一个namenode可能管理/user目录下的所有文件元信息，另一个管理/share目录下的。

在federation中，每一个namenode管理一个*namespace volume*(namespace的元信息)。简单的说就是**元信息的元信息**，一个 *block pool* 包含了**namespace中所有文件的blocks位置**。Namespace volumes是相互独立的，也就是说一个namenode的出错不会影响到其他namenode。Block pool 的存储是全文件系统的，并不是像namenode那样的一部分，所以datanodes像集群中的每个namenode都有注册，并存储来自多个block pool的block

所以为了管理HDFS集群，客户端挂载关于namenodes的映射表，通过*ViewFileSystem* 和 *viewfs://URIs* 进行管理

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/11.png?raw=true">

##### HDFS High Availability

多文件系统备份和secondary namenode的结合使用防止了数据丢失但是并没有提供高可用性。namenode 仍然是一个 *a single point of failure* (SPOF) ，如果namenode出错，所有客户(包括MapReduce jobs)都不能够进行正常读写等操作。**因为namenode存储了唯一的元信息仓库以及文件到block的映射，在这种情况下，整个Hadoop系统都会挂掉，直到一个新的namenode出现** 

**为了从 failed namenode 的情况中恢复**，管理员开启一个新的primary namenode (包含了文件系统元信息备份并且配置了datanodes和clients(MapReduce job等))。缺陷是

- 新的namenode 必须先将namespace image加载进内存才能工作
- edit log 需要重新写
- 从datanode中接受足够的block report 才能进入正常工作模式

**对于一个大集群来说，通常这个过程消耗至少30分钟。**这对于日常维护来说是一个big problem ，实际上namenode failed的情况很少见。

Hadoop2 通过增强可用性来解决问题，在它的解决方案中有一对namenodes同时在工作 (active-standby模式，一个工作，另一个时刻准备)，当其中一个namenode 故障另一个就会接手。所以外部并不会感受到其中一个namenode故障了。 整个框架都进行了微调

- namenodes 必须使用高可用共享文件来share edit log。当一个备用namenode出现时，会读取共享edit log的末尾来同步活动节点的状态，然后继续同步后续的数据变化
- Datanodes 必须将block reports送到两个namenodes中，因为block 的映射存储在内存中，而不是磁盘中
- 必须使用对Clients透明的客户端配置为处理节点故障转移
- secondary namenodes的角色被备用节点取代，备用节点会定期的检查活跃的节点并同步数据

对于**高可用共享存储** 由两种选择：**NFS filer**(Network File System网络文件系统：允许网络中的计算机共享资源)或者是quorum journal manager(**QJM**)。**QJM 是HDFS的一种特化实现，设计用来提供edit log高可用性的**，是HDFS的官方推荐。QJM 运行一个journal nodes组，每个编辑必须写进主要的journal nodes中，一般有3个journal nodes，所以即使丢失了一个也不影响大局。这个策略很像ZooKeeper，但是QJM的实现并没有使用ZooKeeper （不过HDFS HA 倒是的确使用了ZooKeeper来选择 active namenode）

当active namenodes故障，standby 节点可以在几十秒中取代active ，因为大部分edit log和block mapping都在standby节点中有了，在实践中这个几十秒可能会放大至1分钟左右，因为系统需要保持稳定 当active namenode发生故障时。

当**active namenode 故障时，standby 也挂掉了**(虽然几率很低) 管理员可以**手动的启动一个standby 节点，这种情况类似于没有高可用性的例子，大约30分钟或者更久时间**。

##### Failover and fencing

将active namenode转成standby是由系统中一个叫 *failover controller* 控制的，有不同的故障转移控制器 ，但是默认的实现是使用ZooKeeper来保证只有一个namenode是active的（master选举）。每一个namenode都运行着一个轻量级的failover controller进程 来监控namenode的故障(采用简单的心跳机制)，当namenode fail就会触发 failover。

Failover 也可以被Administrator手动的初始化，比如在日常的维护时手动failover，这叫做*graceful failover*，将active 和standby 有序的转换角色。

在ungraceful failover即非正常故障转移中，我**们不能确保该namenode fail之后停止运行，因为网络速度慢或者网络分区都可能触发 failover，触发之后 active standby角色互换，但是原active仍然在运行还认为自己是active的**，即使备用的节点已经代替了它的角色。这个时候HA策略会保证原active的节点不会对系统有损伤，这就是**fencing**

QJM同一时刻只允许一个namenode去写edit log，但是原active的namenode仍然可能对clients提供过时的查询服务(参考上一段)，所以可以通过设置SSH隔离命令来杀死原active namenode进程。

当使用**NFS filer**作为共享edit log的策略时需要更强的fencing 方法，因为它**没办法在同一时刻只让一个namenode进行访问**（这也是官方推荐使用QJM的原因）

**fencing机制**包括了1.**控制namenode对于共享存储目录的访问权**(通常是通过使用 vendor-specific NFS command(供应商特定的NFS命令)达到目的) 2.**通过remote management command(远程管理命令)来禁用其网络端口**。3.最后的方法是通过 图形上称之为 **STONITH**的技术隔离原active namenode，**它是用专用的配电单元强制关闭主机电源**

客户端可以透明的处理Client failover ，最简单的策略就是采用客户端配置来控制failover，HDFS URI使用了逻辑主机名(logical hostname)来 映射namenode地址（存储在配置文件中）。

#### The Command-Line Interface

在这一节里我们会尝试通过命令行和HDFS交互，有很多方式可以和HDFS交互，但是命令行无疑是开发人员最熟悉的也是最简但的方式。

这里只在一个机器中运行HDFS，先下载Hadoop按照教程走之后，开始进行操作。这里就不提供途径了。

##### Basic Filesystem Operations

这里就不谈它具体的命令了，有兴趣自己去查查

##### File Permissions in HDFS

HDFS的文件和目录的权限模型类似于POSIX模型，有三种：

- r  -read permission：读文件和列举目录中所有的文件
- w -write permission：写文件或目录，创建和删除权限也有
- x -execute permission：对文件不起作用，对于目录是 访问它的孩子

每一个文件和目录都有一个 owner, group 和 mode，mode由**该文件的owner用户** ，以及**该用户所处的group**和**既不是该文件的owner用户也不在该用户的group中**，三种模式构成

**默认情况下，Hadoop禁用security**，**这意味着不会对clients进行身份验证**。因为clients是远程的，只需在远程系统上创建一个具有该名称的账户，client可以成为任何一个用户。无论怎么说，开启权限系统都是值得的，可以避免用户或者自动工具程序对 文件系统的意外修改或删除。

权限系统会先检查 client's username 是不是 owner，然后再检查是否在该owner的group中，最后会检查其他权限。值得注意的是 **权限系统不会对 超级用户进行权限检查**，它是namenode进程的标知。

##### Hadoop Filesystem

Hadoop有一个文件系统的抽象概念，HDFS只是它的一个实现。Java的 `org.apache.hadoop.fs.FileSystem` 抽象类就代表了hadoop文件系统的client 接口，这里就不列出了

##### Interfaces

Hadoop是用Java实现的，所以大多数的Hadoop文件系统都是通过Java API来交流的。文件系统 shell就是使用Java FileSystem类提供的文件系统操作，...

##### HTTP

通过将文件系统作为java API，Hadoop使得其他非Java程序访问HDFS变得困难，HTTP REST API作为WebHDFS协议使得其他语言的程序能够更方便的和HDFS交流，**但是HTTP接口会比native Java client要慢**(本地通过RPC协议互联)，所以使用HTTP接口时应该避免大量数据的传输

通过**HTTP访问HDFS有两种途径：1.直接访问，2.通过代理访问**。HDFS daemons有专门运行一个HTTP 服务于clients，proxy代理是通过DistributedFileSystem API作为client访问的HDFS

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/12.png?raw=true">

直接通过HTTP访问，namenode和datanode内部的web服务器作为WebHDFS的端点，由namenode操作文件元数据，**文件读写操作首先发送到namenode节点，然后从namenode节点重定向到client 再从client到datanode进行数据交流**

第二种方式是通过代理服务器来访问HDFS，(**代理是无状态的，所以它们可以在一个标准的负载均衡器之后运行**)，所有通往集群的操作都要经过代理服务器，所以client从不直接访问namenode或者datanode，这就运行集群采取更加严格的防火墙和带宽限制策略。

在一个集群中不同的数据中心交互一般都是通过代理服务器，或者是从外部网络访问在云端运行的Hadoop集群时也会通过proxy。

##### NFS

在client本地文件系统使用Hadoop's NFSv3 gateway可能可以挂载HDFS，可以使用Unix工具(比如 ls,cat)来和文件系统进行交流，上传文件。还可以通过POSIX库使用不同的编程语言进行访问，不过HDFS只允许写文件的append

##### FUSE

Filesystem in Userspace （FUSE）允许在用户空间运行的文件系统和Unix文件系统集成。

#### The Java Interface

这节我们主要深挖 FileSystem 类，用于和Hadoop 文件系统交互的接口。在Hadoop2中改为了FileContext，尽管我们主要关心HDFS的实现 DistributedFileSystem，但是FileSystem是抽象类，针对接口编程会更好。

##### Reading Data from a Hadoop URL

要连接Hadoop FileSystem最简单的方式就是利用 java.net.URL 对象打开一个流

```java
InputStream in=null;
try{
    in=new URL("hdfs://host/path").openStream();
}
```

要使Java识别Hadoop的hdfs url方案，还要做更多的工作

```java
import org.apache.hadoop.fs.FsUrlStreamHandlerFactory;
import org.apache.hadoop.io.IOUtils;

import java.io.InputStream;
import java.net.URL;

public class URLCat {
    //设置URL，通常一个JVM只跑一次，所以是static块中
    //但如果第三方库或者其他地方也 设置如下了URL，这里的代码就不能正常从Hadoop读取数据了
    static {
        URL.setURLStreamHandlerFactory(new FsUrlStreamHandlerFactory());
    }

    public static void main(String[] args)throws Exception {
        InputStream in=null;
        try{
            in=new URL("").openStream();
            //通过IOUtils 将从hadoop读取的数据 通过标准输出到控制台，缓存大小为4096字节
            IOUtils.copyBytes(in,System.out,4096,false);
        
        }finally {
            IOUtils.closeStream(in);
        }
    }
}
```

##### Reading Data Using the FileSystem API

上面有说到，有时候不能通过设置*URLStreamHandlerFactory*来读取数据，这里提供另外一个API，FileSystem有一些静态工厂方法可以获得FileSystem对象

```java
public static FileSystem get(Configuration conf)
public static FileSystem get(URI uri,Configuration conf)
public static FileSystem get(URI uri,Configuration conf,String user) 

```

`Configuration`对象是client或者server的配置信息，从类路径中读取配置文件 比如 */etc/hadoop/core-site.xml* ，

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;

import java.io.InputStream;
import java.net.URI;

public class FileSystemCat {
    public static void main(String[] args) throws Exception {
        String uri = args[0];
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(URI.create(uri), conf);
        InputStream in = null;
        try {
            //这样通过FileSystem和Path对象 就可以确保读取到数据
            in = fs.open(new Path(uri));
            IOUtils.copyBytes(in, System.out, 4096, false);
        } finally {
            IOUtils.closeStream(in);
        }
    }
}
```

##### FSDataInputStream

上面代码中的 `fs.open();`实际上是返回一个FSDataInputStream对象，该类支持随机读写

```java
package org.apache.hadoop.fs;

public class FSDataInputStream extends DataInputStream implements Seekable,PositionReadaBle{
    ...
}

public interface Seekable{
    void seek(long pos) throws IOException;
    long getPos() throws IOException;
}
```

像上面读取数据的代码用了FSDataInputStream就可以

```java
public class FileSystemDoubleCat {
    public static void main(String[] args)throws Exception {
        String uri = args[0];
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(URI.create(uri), conf);
        FSDataInputStream in = null;
        try {
            in = fs.open(new Path(uri));
            IOUtils.copyBytes(in, System.out, 4096, false);
            in.seek(0);
            IOUtils.copyBytes(in, System.out, 4096, false);
            //利用seek(0)定位到0，重新读了一遍
        } finally {
            IOUtils.closeStream(in);
        }
        
    }
}
```

##### Writing Data

最简单的方式还是利用FileSystem和Path对象，创建文件和输出流

```java
public FSDataOutputStream create(Path f) throws ..
```

上面的create方法可以配置1.是否覆盖已经存在的文件，2.文件的复制因子(复制几分？) ，3.buffer的大小，4.文件的block大小，5.文件权限

如果路径中的目录都不存在，他还会自动创建它们。如果不希望创建它们，可以使用`exists()`方法来确定目录是否存在。

还有一种用于传递回调接口的重载方法——Progressable，所以我们可以知道往datanodes写数据的进度。

```java
package org.apache.hadoop.util;

public interface Progressable{
    void progress();
}
```

可以使用 `append(Path f)`方法来创建文件，如果该文件存在会从尾开始写。不过append方法不是所有Hadoop文件系统都支持的，HDFS支持，S3 文件系统不支持。

下面的例子中，展示了从本地文件到Hadoop 文件系统的复制过程，通过 progress()方法展示进度，

```java
public class FileCopyWithProgress {
    public static void main(String[] args) throws Exception {
        String localSrc = args[0];
        String dst = args[1];

        InputStream in = new BufferedInputStream(new FileInputStream(localSrc));

        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(URI.create(dst), conf);
        OutputStream out=fs.create(new Path(dst), new Progressable() {
            public void progress() {
                System.out.println(".");     
            }
        });//每复制一次数据，都会打印出 . 

        IOUtils.copyBytes(in,out,4096,false);
    }
}
```

Progress在MapReduce中比较重要，但是其他Hadoop文件系统并没有调用progress()。

##### FSDataOutputStream

和FSDataInputStream差不多，create()也是返回一个FSDataOutputStream，有一个返回位置的方法

```java
public class FSDataOutputStream extends DataOutputStream implements Syncable{
    public long getPos() throws IOException(){
        ...
    }
}
```

**FSDataOutputStream不允许seek，这是因为HDFS不支持随机写，只允许写新文件或者append已存在文件。**

##### Directories

关于创建目录`public boolean mkdirs(Path f) throws IOException`，这个方法会在目录不存在的情况下创建目录，如果创建成功就会返回true，不过很多时候都是通过 `create()`创建文件的时候，自动创建目录

##### Querying the Fielsystem

File metadata---> FileStatus

任何文件系统都有一个重要的特点：持有整个系统的目录的结构和存储的目录与文件信息，**FileStatus类封装了文件系统关于文件和目录的元信息**，包括文件长度，block size，复制，修改时间，拥有者等等。

可以通过 `fs.getFileStatus(new Path("file path"));`获得文件的元信息，也可以获得一个目录的所有文件元信息`fs.listStatus(new Path("directory path"));`返回一个FileStatus数组，如果该参数路径是一个文件，那么数组长度为1。也可以通过FileStatus数组得到Path数组 `FileUtil.stat2Paths(status)` 

##### File patterns

在一次操作中处理多个文件时很普遍的，比如 MapReduce job 对于log进程可能会分析一个月以来的存储在一些目录中log文件，**和一个一个枚举出文件和目录的表达式相比使用使用通配符来匹配多个文件会是更好的选择** 在Hadoop体系中，PathFilter就是起到匹配正则表达式的作用

```java
public class RegexExcludedPathFilter implements PathFilter{
    private Final String regex;
    public RegexExcludedPathFilter(String regex){
        this.regex=regex;
    }
    public boolean accept(Path path){
        return !path.toString().matches(regex);
    }
}

FileStatus[] fileStatuses = fs.globStatus(new Path("/2007/*/*"), new RegexExcludedPathFilter("^*/2007/12/31$"));
 
```

过滤器只对文件名起作用，不能对文件的属性：比如创建时间进行过滤。

##### Deleting Data

还是FileSystem的方法，

`public boolean delete(Path f,boolean recursive)throws IOException` 如果 路径是一个文件或者空目录，recursive的值就会被忽略，因为有它没它都一样。**一个非空目录要删除掉，recursive=true时才可以完成，意味着递归删除文件**。

##### Data Flow

下面的图展示了HDFS的读数据流程

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/13.png?raw=true">

- 通过` FileSystem.open() ` 打开这个文件，这时候DistributedFileSystem 通过RPC(Remote Procedure Call) 来 访问namenode，**来获得文件的各个block的位置，其实就是存放block的datanode的位置**，并且datanode将根据和client的距离进行排序（这里的距离参考下节**Network Topology and Hadoop**），如果client本身是datanode，并且该文件有block在该节点上，就可以从本地读取数据啦。

- DistributedFileSystem **返回一个FSDataInputStream(支持file seeks) 到client并用于读数据**，FSDataInputStream其实是包装了DFSInputStream（这个类管理datanode 和 namenode 的I/O）

- client **调用 read()方法读取数据**，DFSInputStream存储了文件的前几个block的datanode的地址，**然后连接上文件中第一个block(因为一个block有放在多个datanode中)最近的的datanode**，在datanode的数据以流的形式返回到client，client不断调用read() 读取流中的数据。当block被读完了，DFSInputStream就会关闭datanode的连接，然后找到下一个block的最近datanode。这个过程对client是透明的，所以在client看来只是读取一个连续的流即可

- 在按顺序读取block的时候，client还会调用namenode取回下一批datanodes的位置，当文件读取完毕时，FSDataInputStream.close() 关闭文件。

在读数据的过程中，**如果DFSInputStream和datanode交流的时候遇到了error，DFSInputStream会尝试读取该block的另外的datanode(也是最近原则)，也会记下出现故障的datanode，之后读取数据的时候会避开这些datanode**。DFSInputStream还会验证从datanode中传输过来的数据的checksums。**如果遇到了故障的block，就会从其他datanode中读取该block副本，并且向namenode汇报这个故障**

这种从namenode获得最佳datanode位置的设计让HDFS能更拓展更多的client，因为数据在整个集群中传递，流量就分散了速度就上来了，同时namenode只需要负责datanode的location请求，这些location都是在内存中的，速度快。

##### Network Topology and Hadoop

在局域网中**两个节点对对方是 close 状态**是什么意思呢？在高容量数据处理的环境中，限制因素就是两个节点之间传递数据的速度---带宽 （因为带宽在集群网络中是稀缺资源）。 而Network Topology的想法是**使用两个节点的带宽作为距离的度量**

然而在实际情况中是**很难去度量集群中两个节点的带宽**，因为这**需要集群是静态**的并且**节点的增加会增加更多节点对**(因为需要知道这个新增节点和其他所有节点的 距离)，Hadoop采用了一个简单的方法 **将集群网络表示成树，并且它们之间的距离就是两个节点之间的最近共同祖先**

**以下情景 可用带宽逐渐变小**

- Processes on the same node
- Different nodes on the same rack(机架，大致是连接同一台交换机)
- Nodes on different racks in the same data center
- Nodes in different data centers

总共有三种级别 node ， rack ， data center。用简单的字母表示**第一个data center 第一个rack 第一个节点** 为**/d1/r1/n1**  

- distance(/d1/r1/n1,/d1/r1/n1)=0 相同节点距离为0
- distance(/d1/r1/n1,/d1/r1/n2)=2 不同节点相同机架 
- distance(/d1/r1/n1,/d1/r2/n3)=4 不同机架同一数据中心
- distance(/d1/r1/n1,/d2/r3/n4)=6 不同数据中心

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/14.png?raw=true">

Hadoop是如何定义 Network topology 呢？这个在后续章节中有说。简单来说就是 它**假设**网络都是平面的单层结构，换句话说**所有节点都在一个数据中心的一个机架上**

##### Anatomy of a File Write

下面的图中展示了 一个文件的创建，写入和关闭的整个过程。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/15.png?raw=true">

先调用 **DistributedFileSystem.create()** 创建文件，**然后它会RPC在namenode的namespace中创建一个新文件**，这时候没有block与之关联。 **namenode会确保之前没有过同名文件并且该client有权限创建这个文件**。

如果创建文件失败(比如file already exists)，client会抛出一个IOException。DistributedFileSystem返回一个FSDataOutputStream 给client，用于写入数据。**和read的情况类似，FSDataOutputStream包装了DFSOutputStream，用于处理与datanodes和namenode的通信。**

当client写入数据的时候，DFSOutputStream会将它们切分成数据包并将其写入数据队列(*data queue*)。data queue被DataStreamer消费，**DataStreamer用于让namenode通过选择一系列合适的datanodes来存储备份来分配新的blocks**。这些datanodes组成一个pipeline(一般都是3个数据备份)。

DataStreamer会将数据流式的传输进管道的第一个节点，这之后第一个节点的数据会通过管道流进第二个节点再到第三个节点。最后数据在管道中保持一致。(这是第四步)

DFSOutputStream也维护了一个内部数据包队列，这些数据都是等待datanodes确认的，这个队列有个专门的名称 *ack queue*。只有当pipeline中的所有datanodes都确认了该数据包，才会将它从ack queue中remove。（这是第5步）

**如果其中某个datanode失败了**，后面的策略对client是透明的。

- First，关闭pipeline，并且将ack queue中的数据包都加到 data queue的最前面：这样故障节点的下游节点不会丢失数据包
- **namenode会将该管道中其他datanode的该block一个新的标知，所以当故障节点恢复过来时会将该部分block删除**。故障节点被移除出pipeline，所以这个pipeline只剩两个节点了，这时候namenode注意到这个block只有两个备份就**会安排一个新的datanode做备份**，
- 也可能出现多个节点都出现故障，虽然可能性微乎其微，只要`dfs.namenode.replication.min`replicas（默认为1）写入了，这个write就会成功，而备份的数量由`dfs.replication` (默认为3) 决定
- 当client结束write，就会close流，这个操作会将剩下的所有数据包都刷进datanode pipeline等待确认，并与namenode联系表示这个文件已经完成了

总的过程就是数据在进入pipepine之前被分成数据包进入data queue，进入pipeline之后就是进入ack queue等待pipeline的三个节点都有该数据包之后将其从ack queue中删除。最后ack queue空了之后，就继续从外面data queue那数据。

##### Replica Placement

namenode是如何选择datanodes来存放数据的呢？这里是可靠性和写带宽与读带宽的一个trade-off。比如，将所有的备份都放在同一个datanode中 最节省写带宽，但是它没有提供真正的数据冗余 一旦这个datanode故障了，数据就丢失了。而放在不同机架上读带宽又消耗很大

事实上，将数据放在不同的数据中心才是真正的做到数据冗余，但是太消耗带宽了。即使放在同一个数据中心，也有不同的放置策略：

**Hadoop的默认策略是将第一份数据放在client本身**(如果client是在集群外的，那么第一个节点就是随机的)，**第二个备份放在不同的机架上(也是随机的)，第三份也是在第二份的机架上，但是节点不一样(随机选取)**。再之后的备份就完全随机了，hadoop默认是3个备份。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/16.png?raw=true">

这个策略兼顾了可靠性，读写带宽。

##### Coherency Model

FileSystem的一致性模型描述了文件读写的数据可见性问题，HDFS权衡了一些POSIX性能要求，所以某些操作的行为可能与预期不符。

当创建一个文件之后，她在namespace中是可见的

```java
Path p=new Path("p");
fs.create(p);
assertThat(fs.exists(p),is(true));
```

但是写文件到该文件中并不保证可见，即使stream 被刷进去了，下面的代码显示 长度为0，其实我们已经写内容进去了

```java
FSDataOutputStream fsDataOutputStream = fs.create(new Path(""));
fsDataOutputStream.write("content".getBytes("UTF-8"));
fsDataOutputStream.flush();   assertThat(fs.getFileStatus(p).getLen(),is(0L))
```

一旦写入数据超过了一个block，该block对于其他新的reader来说就可见了。后续的block也是这样的，**正在写的block对其他reader来说是不可见的**

所以Hadoop提供了一种方式来保证数据在正在写的block是可见的，`hflush()`

```java
fsDataOutputStream.hflush();   assertThat(fs.getFileStatus(p).getLen(),is(0L))
```

`hflush()`并不保证datanodes的数据写进了磁盘，只是刷进了内存中(而在内存中的坏处就是可能造成数据丢失，比如停电)。如果一定要刷进磁盘，可以用`hsync()`代替。当然直接close()也会隐式调用hflush。

对于这种一致性设计，如果没有调用hflush()，hsync()就要做好数据丢失的准备(因为系统随时可能故障)。对于很多应用来说，这是不可接受的，所以**必须在合适的时候调用hflush()，比如在写了确定数量的records或者确定数量的字节后就调用一次**。

而hflush()会带来一些性能开销(hsync更是)，所以次数需要把握好，这是数据健壮性(data robustness)和吞吐量之间的trade-off。

##### Parallel Copying with distcp

到目前为止HDFS的访问模式都是单线程访问，但是很多时候都是并行访问文件，这部分的代码需要开发人员自己写，hadoop提供一个有用的工具 *distcp*用于在集群中并行的复制数据

```sh
hadoop distcp sourcefile1 destfile
hadoop distcp dir1 dir2
```

distcp是利用MapReduce job来实现的，不过没有reducers，每个文件都在单个的map中复制

还有一些其他操作就不细说了

##### 

























