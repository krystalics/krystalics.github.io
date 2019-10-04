from 《Hadoop：The Definitive Guide》

---

#### Chapter II ：MapReduce

实际上MapReduce是一种简单的编程模型，但是想要编写出有用的程序仍然是不容易的。在这一章中，将使用不同的编程语言Ruby，Java，Python来完成一个例子，体验MapReduce的本质 并行，我们可以使用任何一种集群来完成一个作业。

##### A Weather DataSet

example:  National Climatic Data Center ，NCDC使用的气象传感器会从全球各个地方收集大量的气象数据日志数据，很适用与MapReduce来处理，不仅是数据量大，还因为这些数据都是半结构化并且是文本类型的log文件。

##### Data Format

数据使用ASC II码并且以行为记录，一行就是一个记录。由于数据并没有一个确定的格式，比如某一列对应着确定的列名，**这些数据是半结构化(一部分有规律，另一部分是随机的),所以为了简化问题我们只关注最基本的信息：温度。**

下面有一张图来显示该数据的格式，将某一行分割开来，便于阅读。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/3.png?raw=true">

很清晰的可以看到由一堆字和字母来表示特定的含义。这也是便于处理这些数据而设定的格式。一般文件的命名都是 date+weather station --》 `010010-99999-1990.gz` 中间的99999表示WBAN天气站。

因为全球有成千上万个气象站，所以数据集是非常繁多的小文件组成的。一般来说，处理**数量小而大小相同的大文件**会比处理这些小文件要快一些，所以我们会先把数据进行预处理，即融合一年的各个气象站的文件成为一个单独的大文件 [数据源](https://github.com/tomwhite/hadoop-book/blob/master/input/ncdc/all/1902.gz) 

##### Analyzing the Data with Unix Tools

每一年最高温度是多少呢？接下来先演示不使用Hadoop的例子，使用一个工具 awk ：处理行式数据 ，加一个小脚本：

```bash
#!/usr/bin/env bash
for year in all/*
do
 echo -ne `basename $year .gz`"\t"
 gunzip -c $year | \
 awk '{ temp = substr($0, 88, 5) + 0;
 q = substr($0, 93, 1);
 if (temp !=9999 && q ~ /[01459]/ && temp > max) max = temp }
 END { print max }'
done
```

在上面我们也介绍了文件的格式，这个脚本就是针对该格式进行处理。这里就不细说了，我也看不懂》.《。使用**AWS EC2的高性能CPU机器 运行这段脚本总共花费42分钟**。

为了提高速度，引进了并行计算；理论上我们可以让CPU处理一个年份的数据(这里考虑多核计算机的情况)。但是仍然会有一些问题：

- 很多时候需要**将数据切分成大小相同的块chunk**，但这又不是容易的工作。比如，这个例子中每一年的数据大小肯定是不一样的，假设就按照每一年的数据交给一个CPU处理，那么就会造成最终的时间是最长的文件处理时间，所以需要将所有数据重新切分，形成固定大小的chunks。
- 将所有结果从互不依赖的进程中combining，需要更多的处理。在这个例子中，每年的数据都是没有依赖关系的，它们最终的结果是依靠拼接并排序形成的。如果**使用了固定大小的chunk，这个combine操作就会更加复杂**。在这个例子中，每年分成多个chunk，然后每个chunk都会有一个最高温度，最后combine的时候要先将每年的chunk先combine。
- 最大的问题是单个机器的容量有限，CPU数量也有限，他有固定的上限值。而使用多台机器，集群就可以更加扩展这方面的上限，提高协调性和可靠性。

我们之前也讨论过，机器存在出错的概率，当集群数量越来越大，我们该怎么处理失败的进程呢？所以即使多态机器并行计算是可行的，在实际操作过程中也是很难的。而使**用Hadoop，就可以让开发人员不用在去关心机器怎么处理失败的进程。**

#### Analyzing the Data with Hadoop

MapReduce的处理流程如下：以NCDC的数据为输入，组成key-value(这里key为该数据的行数，value为该数据)到map阶段，得到的是(year,temperature)的key-value输出，作为reduce的输入，得到当年最高温度

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/4.png?raw=true">

在网上找教程写了Hadoop的第一个程序WordCount却因为windows没有权限运行，书上的MaxTemperature程序也是一样的。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/5.png?raw=true">

由一个入口类来开启一个Job，然后配置job的mapper和reducer以及其他的东西。设置input目录即输入文件夹和输出文件夹，通过mapper读取文件然后写到中间文件（**即使使用集群：中间文件也是存储在本地，并没有在HDFS中**，因为mapper的输出文件最后是要被删除的，没有必要放到hdfs中），得到最后结果再到reducer中处理得到最终的输出文件。

**可以有多个输入文件夹，只能有一个输出文件夹**，而且每次运行程序，都需要把output文件夹删掉，否则程序会运行失败。这是hadoop的机制吧，至于为什么？

> This precaution is to prevent data loss (it can be very annoying to accidentally overwrite the output of a long job with that of another).

翻译过来的意思大概是，可能两个作业的**输出文件夹**相同，导致另一个作业的结果被覆盖。所以，**运行一个作业的时候需要输出文件夹不存在**

上面是MapReduce在单机中的程序，如果要拓展到分布式中，则需要做一些配置。

##### Scaling Out

首先需要将文件存储在分布式文件系统中(例如HDFS) ，这样**Hadoop就可以采用 YARN 资源管理系统将MapReduce的计算任务分散至各个机器中。**

##### Data Flow

一个MapReduce job 是客户端要执行的工作单元，**包括 输入数据，MapReduce程序和配置信息**。Hadoop将job分解成tasks，任务大约有两种-->map tasks 和 reduce tasks。使用YARN来管理任务并且管理集群中的节点，**如果某个任务在节点中失败了，YARN会自动重新安排其他节点来运行该task **(它们的通讯估计是rpc调用)

首先会将输入数据分解成固定大小的chunk，为每一个chunk分一个map task，为每一行record都运行开发人员定义的map函数。

至于为什么需要将数据切成大小相同的块呢？**当块比较小的时候呢，速度快的机器可以分析更多的块，更好的做到负载均衡。而如果块太小了，整个任务的运行时间就会约等于管理拆分和创建任务的时间，所以块一般为128MB，在集群中也可以手动的去调整数据块的大小**。

因为HDFS采用的策略是数据本地化 data locality，所以不会占据集群的宽带资源，也叫 data locality optimization 数据本地优化(**这是和高性能计算对比，高性能计算的数据是存储在一台机器上，然后在运算的时候分发给其他机器**)

 HDFS也是分块的结构，**block应该和我们的数据分片大小一致**，如果数据块比HDFS的block要大，HDFS就要用两个block来存储一个chunk了。所以某些作业的chunk就需要分散到两个节点去计算，这需要将数据传输到另一个节点中，显然会比只在本地计算要慢的多。

有时候该节点和它的副节点都在运行其他任务，这时候作业调度器就会安排其他空闲的节点来做这个作业，这时候就需要将数据传过去，而这集群中的节点都是分组的，有时(概率很小)找到的空闲节点在其他组

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/6.png?raw=true">

而Reduce tasks并没有data locality的优势，输入reduce的值通常都是所有mapper的输出集合，这些**数据从不同的节点中汇聚到reduce task的节点中执行开发人员定义的reduce函数**，而reduce的输出则会存储在HDFS中，数据在reduce节点存储，而数据备份在其他组的节点中，所以写reduce会消耗集群的带宽资源，不过消耗与普通hdfs的管道消耗一致。

整个数据流向如下图：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/7.png?raw=true">

Reduce task的数量并不依赖于输入数据的大小，而是独立的。关于如何选择Reduce task的数量有专门的规则。当有多个reducer的时候，所有mapper的结果自然的会分出相应数量的分区，对应reducer。这个分区可以由开发人员定义，但是**通常的分区策略都是使用 哈希算法** 

多个reduce task时的数据流向，有时候map和reduce都可以称之为shuffle

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/8.png?raw=true">

有多个reduce节点自然也可以没有reduce节点，当没有该节点时，mapper的结果直接存储到hdfs中。



##### Combiner Functions

很多MapReduce jobs的性能瓶颈就是集群的带宽资源，所以最小化map传输到reduce节点的数据可以大大提高作业速度。Hadoop允许开发人员定义一个 *combiner function* 来对map的输出进行处理，从而减少map节点传输到reduce节点的数据量。这是一个优化选项，Hadoop可能会调用多次combiner function，所以要保证他的输出和reduce的输入一致。

也不是所有的job都能combine的，像书中的例子，寻找最大值：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/9.png?raw=true">

如果是寻找平均值，就不能使用combiner function

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/hadoop/img/10.png?raw=true">

在代码中加combiner也很容易， `job.setCombinerClass(MaxTemperatureReducer.class);` 这里采用的是Reducer，因为二者的目的是一致的，只不过combiner是选出一个节点中的最大值，而reducer是整个Job的。

关于Hadoop的跨语言特性，因为它本身有定义一套体系以及和各个语言对接的API，所以可以很轻松的对接Java，Ruby和Python等语言。







