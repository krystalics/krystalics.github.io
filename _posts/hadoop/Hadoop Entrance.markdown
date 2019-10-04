Reading《Hadoop: The Definitive Guide》

---

分布式出现的很早也很自然，在以往网络不发达的时候，数据量比较小。在最近几十年，数据量指数型增长，单个机器的存储量别说跟不上就算跟上了，以现在计算机的速度想要遍历一遍磁盘的数据，可就太慢了。书中给出了一个例子：

1990年 一个经典的磁盘1370MB，传输4.4MB/s ，我们在大约5分钟就可以将整个磁盘读完；现在 20年之后，**1TB** 的磁盘是很常见的，但是传输速度却只有**100MB/s** ，将整个磁盘读完需要**2.5**个小时。这是一个不能忍受的速度，写磁盘的速度更慢

所以我们很自然的想到了人类社会的分工，一个人不行就多来点人了嘛。这就是并行计算，将1TB分为100个数据集，由100个计算机来读，只需要2分钟就可以读完所有数据。

但是，和人类社会很像的是，团队合作出错的概率会比一个人要高的多，假设一个计算机在io的时候出错的概率是1%，100个计算机至少有一台出错的概率就是接近1，我们需要面对很多问题。

**第一个问题**就是 **hardware failure**，一个很常见的应对方式是 **将数据复制一份**，当**该数据集出错的时候，可以有另一份数据可用** （这就是数据库主备思想）

这也是磁盘阵列 RAID 的工作原理，而Hadoop Distributed Filesystem(HDFS) 的处理方式和这有点不一样。



**第二个问题**：将数据分散存储，需要它们的时候却需要从不同磁盘中combine，不同的分布式系统都允许多数据源的combine ，但是真正 正确的做到这点是非常难的。Google提出的MapReduce 提供了一个计算模型来帮助完成这一点。



**Hadoop 为存储(storage)和分析(analysis)提供一个可靠，可扩展的平台。** HDFS是分布式存储，MapReduce则是分布式计算

MapReduce需要整个数据集(或者其中部分)都是可以查询的，它拥有**批量查询处理机制**并且**能够在可以承受的时间内进行即时查询**，它解决了上述的读取整个数据集太慢的问题却也引入了新的问题，这似乎是一个技术进步必备的步骤，使用解决方案的同时引入新的问题，再到新的解决方案和新的问题。

要知道**MapReduce**可以说只是一个批处理系统，并不能够在很短的时间(比如几秒)，通常都是需要几分钟的处理时间所以适用于离线转态的数据处理。

随着时间的推移，Hadoop不再只用于批处理了，不再只包含HDFS和MapReduce，添加了很多组件，比如HBase，kafka等。而HBase是第一个提供在线查询的组件，它是一个使用了HDFS为基础的key-value数据仓库。同时提供了在线服务以及批处理服务来应对不同的业务场景

虽然已经出现了很多组件，但是真正推动Hadoop变革的是**YARN (Yet Another Resource Negotiator) ，这是Hadoop2的标志**。它是一个集群管理系统**，可以运行不同的分布式程序(不只是MapReduce) 来对Hadoop集群上的数据进行处理。**

我们从上文中知道MapReduce适用于离线状态，那要即时处理该怎么办呢？

> Interactive SQL：互动SQL，对应即时查询
>
> Impala ：分布式查询引擎，有专门的always on 守护线程
>
> Hive或TeZ ：容器重用

> Iterative processing：迭代处理
>
> Spark：在内存中做计算，更搞笑和即时

> Stream processing：流处理
>
> Storm或者Spark Streaming或者Samza ：可以在无限制的数据流中进行分布式计算，并将结果存储到Hadoop storage或者外部系统中

> Search：搜索
>
> Solr：运行在Hadoop集群上，对添加到HDFS中的数据建立索引，方便查询



##### 那么Hadoop和传统的关系型数据库管理系统有什么不一样吗？

要知道MySQL它们也有集群呀。这个问题的答案其实是因为存储设备，从磁盘中获取数据的时间=寻道时间+读取数据时间  ，数据量越大平均寻道时间也就越大，整体的速度就越慢，而传输速率的发展远比寻道时间要快。

数据库的读写都是需要找到对应的磁盘存储位置，这个过程就是寻道。而通过Hadoop进行流传输会更快(因为传输速率更快)

> If the data access pattern is dominated by seeks, it will take longer to read or write large
> portions of the dataset than streaming through it, which operates at the transfer rate.

还有就是在更新小规模数据的时候，关系型数据库的索引B-Tree可以工作得很好，但是当需要更新大量数据时，关系型数据库的工作量大大增加(因为还要更新索引)。而MapReduce的 Sort/Merge可以更加快速的重建数据库。

|              | 传统关系型数据库          | MapReduce                      |
| ------------ | ------------------------- | ------------------------------ |
| Data Size    | GigaBytes                 | PetaBytes                      |
| Access       | Inteactive and batch      | batch                          |
| Updates      | Read and write many times | write once and read many times |
| Transactions | ACID                      | None                           |
| Structure    | Schema-on-write           | Schema-on-read                 |
| Integrity    | High (数据不怎么冗余)     | Low (数据冗余)                 |
| Scaling      | NonLinear                 | Linear                         |

传统关系型的数据库操纵的都是结构化数据，比如XML文档或者数据库中预先按照某种格式定义的表。对于半结构化或者没有结构的数据来说(文本，图片)Hadoop的表现更佳

而且**MapReduce或者说是Hadoop的其他组件都是规模线性的**：假设1个工作需要100台计算机工作1天，如果是200台就会是需要半天，这种**集群规模和工作能力呈线线性关系**。而RDBMS则是非线性关系。



##### Grid Computing

高性能计算和网格计算是做大规模数据处理的策略，而通过Message Passing Interface (MPI) 消息传递接口就可以使用这策略。简单来说，高性能计算就是 **将任务分给一个区域内的集群，这些机器共享文件系统，然后由统一的存储区域网络(storage area network,SAN)来控制**。

高性能计算在计算密集型作业中运行良好，但是在数据量庞大的情况下(比如几百GB)，数据的传输受到带宽的限制 数据要从一个节点到另一个节点会非常慢，而且会有大量的空闲节点出现(没有计算任务，因为数据还没流到它那儿) 。补充一点（从这段的描述来看，数据是从原始节点流向集群中其他节点的）

Hadoop 就解决了高性能计算在的数据量大的情况下遇到的问题，它的策略是：**将数据就存储到不同的节点中，所有节点需要的运算只需要从本地获取数据而不需要经过网络传输**。这就是 **data locality**，它是Hadoop高性能的主要核心点。

在一个集群中，节点之间经常发生交流，这个时候带宽就成了很大的资源限制，所以Hadoop做了很多措施来保证没有无意义的交流(这里**我个人的理解就是，如果A->B A->C 那么B->C就是无意义的交流**)，即通过网络拓扑模型来显式的保护带宽资源。

开发人员可以随意使用MPI，权限很高，但是他需要通过C语言程序或者高级算法来显式的操纵数据流。而Hadoop只是通过key-value的数据模型来隐式操纵数据流

协调大规模分布式计算的程序是非常难做的，其中最难的部分就是处理故障：因为我们不知道这个节点是否成功作业。就继续执行在它之后的作业。像是MapReduce这样的分布式处理框架，**通过实施异常检测和重新作业来保证数据处理正常，让开发人员不用在意这种异常。**

为什么MapReduce能够做到上述的点呢？因为它是一个 shared-nothing architecture (无依赖架构) ，其中一个节点的结果并不依赖其他节点的结果。 简单的说，就是mapper的输出是reducer的输入，整体是在MapReduce框架的控制下进行。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/1.png?raw=true">

所以开发人员只需要将数据交给MapReduce处理，**通过定义mapper函数和reducer函数** 来最终得到结果，中间的过程不需要关心。怎么定义这两个函数呢？这个在第二章中会讲到。

该书总体的流程如下：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/2.png?raw=true">











