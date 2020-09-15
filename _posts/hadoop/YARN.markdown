from 《Hadoop：the Definitive Guide》

---

#### YARN 

Apache YARN(Yet Another Resource Negotiator) 是Hadoop的集群资源管理系统，**YARN在Hadoop2中引入来改善MapReduce，但是也可以支持其他的分布式计算**

YARN并不和开发人员直接交互，它提供的API都是给分布式计算框架的，具体层级如下：

 <img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/17.png?raw=true">

其实还有一层应用是在计算框架之上构建的，比如Pig ，Hive 和 Crunch是在MapReduce，Spark，Tez之上构建的，并没有直接和YARN交互。

##### Anatomy of a YARN Application Run

YARN通过两种类型的 长期运行的daemon 来提供核心服务：一个叫 *resource manager*(一个集群中只有一个) 来管理集群中的资源，另一个是*node managers* 在所有节点都运行来启动和监视 *containers* 。一个container 使用一组受约束的资源(比如内存，cpu，等)执行一个特定的程序，通常都是 Unix 进程或者Linux cgroup。下图说明了YARN的运行机制

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/18.png?raw=true">

首先client会联系resource manager并且让它跑一个 *application master process* , 这时候resource manager会找一个node manager**在container中运行 application master** 。

在container中会简单的计算然后将结果返回给client。或者向resource manager请求更多的containers，使用这些containers来做分布式计算(这一步是MapReduce YARN的做法)

YARN**并没有提供任何方式**让程序的各个部分(包括client，master，process)**相互通信** ，只是使用了远程交流方式例如Hadoops' RPC层来传递状态更新和结果返回。

##### Resource Request

YARN有一个非常灵活的**资源请求模型**，**对一组容器的请求所需的计算机资源(CPU,内存)，以及该请求中容器的位置限制**都能有良好的表述。

机器的**分布位置** 对于**确保分布式数据处理算法**能够**有效利用集群带宽**至关重要，所以**YARN允许应用程序自己指定容器组的位置**，这种位置可以是特定节点或者机架上或者是集群中任意位置。

有时候不能满足应用程序的 位置约束请求，要么不分配，要么让应用程序放宽约束(这一点就像我们人类处理事情一样的)。比如请求中**要求某个特定节点作为container，但是这个节点已经运行了其他的container**，所以这个请求就无法命中 **这时候YARN就会尝试在该节点的机架上的另一个节点进行start container的，实在不行就在整个集群中选一个节点。**

通常来说 启动一个container来处理一个HDFS block(可能是运行一个MapReduce的map tastk)，**应用程序会在该block的三个备份中选择一个节点作为container**，如果失败就选择该节点所处**机架**上的节点，如果还失败就扩散至**集群**

YARN可以随时发出resource requests，可以在一个application运行前期就将所有的requests发出，也可以随着application运行动态的根据变化发出resource requests。

而Spark基于YARN做的就是第一个选择，在运行之初就在集群中启动一定数量的executors；而MapReduce分两个阶段，map task的时候是先要求一定数量，reduce task则是动态确定。

##### Application Lifespan

YARN application的生存周期跨度很大，有几十秒的也有几个月的。重点不是它们活了多久，它们的**分类是根据应用程序如何映射user job**，最简单的例子就是一个user job一个application，这也是MapReduce采取的映射方式（1:1的映射方式）

第二种模型就是每个工作流或者用户会话运行一个application，这种方式比第一种更加有效率，因为这样jobs能够重用containers，并且作业之间的中间数据也会缓存下来。Spark就是采用这种方式 （applications：user job= n:1）

第三种模型是 长期运行的application被多个不同的users共享，这样的application通常作为某种协调角色，比如 Apache Slider 有一个长期运行的application master 就是为了启动集群中其他applications。这种模式被Impala采用，它提供一个代理application用于请求集群资源 （application:user jobs=1:n）

##### Building YARN Applications

从头开始写一个YARN application太繁琐了，也没有必要，因为通常都有现有的程序符合我们的要求。比如你对directed acyclic graph (DAG有向无环图 ) 有兴趣 Spark或者Tez是符合要求的：还有Spark，Samza或者Storm可以用于处理 stream

其他细节就不深究了

##### YARN Compared to MapReduce 1

在Hadoop1甚至更早期没有使用YARN，而MapReduce在Hadoop 2之后就基于YARN做了改变，所以这一小节的目的就是对比两个版本的MapReduce有什么不同。

当然不是表面上对比API了，一般来说后续版本的项目都会兼容以往版本的API。

MapReduce 1有两种daemon来控制作业执行处理 ： 1个*jobtracker* 和多个 *tasktrackers*。jobtracker安排task在tasktracker上运行，**tasktracker运行任务并返回进度报告给jobtracker**，这样每一个job都有一个任务进度。如果task失败了，jobtracker会再安排另一个tasktracker运行它

jobtracker只关心两件事1.分配任务 2.任务进度监督（包括 记录task进度，重启失败或者缓慢的task等）。在YARN中这被分为多个不同的实体——> resource manager和 application master(one for each MapReduce job). 

jobtracker还会存储完成的job记录，这是在一个单独的daemon中完成的（分离jobtracker的负载），而在YARN中，是使用timeline服务器存储 application history。

| MapReduce1  | YARN                                                |
| ----------- | --------------------------------------------------- |
| Jobtracker  | Resource Manager,application master,timeline server |
| Tasktracker | Node manager                                        |
| Slot        | Container                                           |

YARN的设计可以弥补MapReduce1的很多缺陷。使用它的好处如下：

- **scalability**

YARN 可以在更大的集群中运行，MapReduce1的伸缩性瓶颈是 4000个节点和40000个tasks，因为jobtracker需要同时管理jobs和tasks。YARN 将jobtracker分的更细了，可以承受10000个节点和100000个tasks

- **Availability**

High availability （HA）通常都是通过备份需要的状态，达到冗余来防止单个节点故障来达到的。但是，jobtracker中有大量快速变换的复杂数据(each tasks status is updated every few seconds) 使得HA变得非常的困难

YARN中将Jobtracker分为两个部分，使程序更可用，就像分治法 分别为 resource manager和application master提供HA。失败措施

- **Utilization**

在MapReduce1中每个tasktracker是通过静态配置固定数量的 slots，slot又分为map slots和reduce slots 两种slots不能通用。

在YARN中 一个 node manager管理了a pool of resources，不是固定的数量，在YARN中运行的MapReduce不会需要因为只有map slots可用而等待reduce task

- **Multitenancy**

某种程度上来说，YARN最大的好处就是可以为其他分布式计算提供服务，不只于MapReduce。在YARN 集群中甚至可以运行不同版本的MapReduce。



##### Scheduling in YARN

 在简单的抽象世界中 YARN application的request可以保证立刻完成，然而实际的应用中这一点是不可能的。首先资源有限、其次集群很忙、一个application通常需要等满足了一些要求之后才能request。

这就是YARN scheduler的工作，按照定义好的策略 分配资源给applications。Scheduling是很困难的问题，并没有一个通用的最优解，所以YARN scheduler会提供不同的选择给applications。

##### Scheduler Options

在YARN中大概有三种schedulers，1.FIFO  2.Capacity  3.Fair Schedulers 。FIFO Scheduler 将applications放到一个队列中并按照它们的先后顺序运行，队首的application的请求最先分配，当请求被满足之后出队，下一个application占据队首。

FIFO Scheduler易于理解并且不需要任何配置，但不适用于共享集群。大一点的application会使用整个集群的资源，这种情况下使用**Capacity Scheduler** 或者 **Fair Scheduler** 会是更好的选择。这两者都运行**长时间运行的作业及时完成**，同时允许正在运行并发 较**小 ad hoc 查询 的用户在合理的时间内获得结果**。

三种调度器如下图：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/19.png?raw=true">

FIFO按照submit的顺序执行application request，而Capacity Scheduler则有**两个队列分别给大application和小的application**，牺牲集群利用率。Fair Scheduler 无需保留一定的数量的容量，因为**它动态平衡所有正在运行的作业之间的资源** ，从图中直观的看，它会让出一定资源给小application。

Capacity Scheduler Configuration 以及后面的Fair Scheduler Configuration这里就不详说了。

##### Preemption 抢占

为了使一个job的完整工作时间可以预测，而不是讲时间浪费在等待资源上，Fair Scheduler 支持抢占式请求。可以通过配置`yarn.shceduler.fair.preemption=true`来支持

通过抢占Scheduler会杀死之前请求的containers，为本次请求腾出空间，但是这很明显需要重新运行之前那个请求，所以集群效率会降低。可以通过时限设置来抢占：前一个请求超出时限就抢占 更详细的策略会在遇到的时候仔细研究

##### Delay Scheduling

所有的YARN Scheduler都尝试满足locality请求，在一个繁忙的集群中，如果application请求一个特定的节点，**该节点很有可能有其他的containers在运行。YARN就会尝试安排该机架上另一个节点来满足locality 需求。**

可是**在实践的过程中发现**，**request时只需要等待几秒钟 就能够大大增加该request的成功率并提高集群的效率**。这个特点被叫做 *delay scheduling* 。Capacity 和 Fair都支持这种特性

在YARN集群中每一个 node manager都会定期发送 heartbeat request到resource manager中 （通常都是1秒1次心跳请求）。heartbeat 携带了node manager正在运行的containers情况和新container能够获取的资源情况，所以**每个heartbeat对于application想要运行 container来说 都可能是 *scheduling opportunity***  

当使用 delay scheduling时，并不是简单的使用第一个 scheduling opportunity ，而是等到给定的 最大scheduling opportunites 数出现后，再放松locality 约束并获取下一个scheduling opportunity。

##### Dominant Resource Fairness 优势资源公平

当只有当个类型的资源需要被调度（比如CPU资源）会比较容易一些，比如两个applicatio需要使用的CPU资源可以很容易的进行比较。而当资源类型多了之后（比如再加上内存资源） ，一个application需要较多CPU资源和较少内存，而另一个相反。这两个application怎么比较使用的资源多寡呢？

YARN中的scheduler是通过查询每个用户的主要资源来解决此问题的，将其作为衡量集群使用情况的一种方法。这种方法就是 Dominant Resource Fairness 或者DRF。举个例子：

假设一个集群有100个CPU和10TB的内存，Application A 请求2个CPU和300GB内存，Application B请求6个CPU和100GB内存。A的请求占了资源总量的(2%,3%) 所以A的优势资源就是内存，而B的(6%,1%) 所以B的优势资源就是CPU

由于B的优势资源占6%，而A的优势资源占3%，在公平共享的前提下 相对而言B会被分配6份资源而A只有3份

简单来说就是将优势资源对比，进行资源分配。





