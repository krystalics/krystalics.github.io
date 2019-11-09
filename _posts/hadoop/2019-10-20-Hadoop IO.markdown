from 《Hadoop : the Definitive Guide》

---

#### 第五章：Hadoop I/O 

Hadoop做了很多 I/O 方面的优化，**一些优化不只是对于Hadoop有效，更是通用的技术**。例如 数据完整性和压缩，其他的是Hadoop工具或者API，构成了开发 分布式系统的基础(例如 序列化框架和磁盘上的数据结构)

##### Data Integrity 数据完整性

Hadoop的用户们都希望在**存储和处理数据的时候不会丢失数据**，但是很明显每一个磁盘或者网络的读写数据操作都有可能会失败，随着数据量增大，故障几率越大。

通常检测数据损坏的方式是 计算数据进入系统时的 *checksum*，当数据通过网络或者其他不可信的（可能损坏数据）通道，都会再次对比 原checksum和现checksum是否一致。如果不一致就视为数据损坏。

只是作为检测，不会做数据修复，checksum 虽然也有可能损坏导致最后对比的时候不一致，但是这个概率很低，可忽略不计。

通常都使用 CRC-32 错误检测码，使用32个字节的整数来计算输入数据的size。 在Hadoop是 *ChecksumFileSystem* ，HDFS 使用更有效率的CRC-32C

##### Data Integrity in HDFS

HDFS透明的校验所有写入其中的数据，并且默认情况下在读数据时验证 *checksum* 。对每一个 *dfs.bytes-perchecksum* 字节的数据创建一个单独的 checksum，默认512字节，因为 CRC-32C checksum是4个字节 存储开销小于1%

##### 

























