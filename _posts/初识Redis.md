<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [第一章：初始Redis](#%E7%AC%AC%E4%B8%80%E7%AB%A0%E5%88%9D%E5%A7%8Bredis)
  - [Redis数据结构简洁](#redis%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E7%AE%80%E6%B4%81)
    - [Redis 中的字符串](#redis-%E4%B8%AD%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2)
    - [Redis 中的列表](#redis-%E4%B8%AD%E7%9A%84%E5%88%97%E8%A1%A8)
    - [Redis中的集合](#redis%E4%B8%AD%E7%9A%84%E9%9B%86%E5%90%88)
    - [Redis的散列](#redis%E7%9A%84%E6%95%A3%E5%88%97)
    - [Redis的有序集合](#redis%E7%9A%84%E6%9C%89%E5%BA%8F%E9%9B%86%E5%90%88)
  - [你好Redis](#%E4%BD%A0%E5%A5%BDredis)
    - [对文章进行投票](#%E5%AF%B9%E6%96%87%E7%AB%A0%E8%BF%9B%E8%A1%8C%E6%8A%95%E7%A5%A8)
    - [如何在SpringBoot里使用Redis](#%E5%A6%82%E4%BD%95%E5%9C%A8springboot%E9%87%8C%E4%BD%BF%E7%94%A8redis)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

来自*《Redis实战》* 

---

### 第一章：初始Redis

----

​	**Redis 是一个远程内存数据库，不仅性能高，而且具有复制特性以及为解决问题而生的独一无二的数据模型。** 提供了5种类型的数据结构，通过复制，持久化(persistence) 和客户端分片(client-side sharding) 等特性，用户可以很方便地将 Redis 拓展成一个能够包含数百GB数据，每秒处理百万次请求的系统。

​	实际上Redis是一个速度非常快的**非关系数据库**(non-relational database，或者说**NoSql**) ，它可以**存储key 与 5种不同类型的值(value)之间的映射(mapping)**，**可以将存储在内存的键值对数据持久化到硬盘中**，可以使用***复制特性***来拓展*读性能*。

​	Reids 不使用表，它的数据库也**不会预定义**或者强制要求用户对Redis存储的**不同数据进行关联**。高性能键值缓存服务器 *memcached* 也经常被拿来与Redis比较： 这两者都可以用于存储键值映射，性能也相差无几

 	1. memcached ： 只能存储普通的字符串键
 	2. Redis ：能够自动以两种不同的方式将数据写入硬盘，并且Redis除了能存储普通的字符串键之外，还有其他4种类型数据。 而且它既可以作为 **主数据库(primary database)** ，也可以作为其他存储系统的**辅助数据库(auxiliary database)**。自然 Redis 就胜出了。

***在实际的开发过程中，如果程序对性能要求不高，又或者因为费用原因而没办法将大量数据存储到内存中，还是考虑像mysql这样的关系数据库，或者将Redis作为二级辅助数据库来用。***​	

| 名称       | 类型                                   | 数据存储选项                                                 | 查询类型                                                     | 附加功能                                                     |
| ---------- | -------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Redis      | 使用内存存储(in-memory) 的非关系数据库 | 字符串，列表，集合，散列表，有序集合                         | 每种数据类型都有自己专属命令，另外还有批量操作(bulk operation)和不完全(partial)的事务支持 | 发布与订阅，主从复制(master/slave replication),持久化，脚本(存储过程 stored procedure) |
| memcached  | 使用内存存储的键值缓存                 | 键值之间的映射                                               | 创建，读取，更新，删除以及其他几条命令                       | 为提升性能而设计的多线程服务器                               |
| MySQL      | 关系数据库                             | 每个数据库可以包含多个表，表中也可以包含多个行；可以处理多个表的视图(view)；支持空间(spatial)和第三方扩展 | select,insert,delete,update,函数，存储过程                   | 支持ACID性质(存储引擎需要使用InnoDB)，主从复制和主主复制(master/master replication) |
| PostgreSQL | 关系数据库                             | 和Mysql差不多，还支持可定制类型                              | 和Mysql差不多，但自定义存储过程                              | 支持ACID性质，主从复制，由第三方支持的多主复制(multi-master replication) |
| MongoDB    | 使用硬盘存储(on-disk)的非关系文档存储  | 每个数据库中包含多个表，表中可以包含多个schema 的BSON文档    | 创建，读取，更新，删除，条件查询等命令                       | 支持map-reduce操作，主从复制，分片，空间索引(spatial index)  |

​	内存数据库有一个首先要考虑的问题，那就是当服务器被关停时，在内存中的数据如何保存下来。Redis 有两种不同形式的持久化方法，都可以用小而紧凑的格式将数据从内存写入硬盘。

 1. **时间点转储**(point-in-time dump) ，转储操作既可以在 “ **指定时间段内有指定数量的写操作执行**” 这一条件满足时执行，又可以通过**两条转储到硬盘命令**中的任何一条来执行。

 2. 将所有修改了数据库的命令都写入一个**只追加(append-only)文件**里面 ，用户可以根据数据的重要程度，将**只追加写入**设置为从不同步(sync)，每秒同步一次，或者每写入一个命令就同步一次。

    虽然Redis的性能很好，但是有时候一台服务器是无法满足所有需求的，所以Redis提供了故障转移(failover) 支持，**Redis实现了主从复制的特性：执行复制的从服务器会连接上主服务器，接受主服务器发送的整个数据库的初始副本(copy)；之后主服务器执行的写命令，都会被发送给所有连接着的从服务器去执行，从而实时地更新从服务器的数据集。**因为从服务器包含的数据会实时更新，所以客户端可以向任意一个服务器发送读请求，以此来避免主服务器进行集中式的访问。

---

#### Redis数据结构简洁

​	**Redis 可以存储键与5种不容数据结构类型之间的映射**，分别为 *1. string(字符串) ，2. list(列表) ，3. set (集合），4. hash(散列) ，5. zset (有序集合)* 。 有一部分Redis命令对于这5种结构是通用的，如：del , type , rename 等。也有一部分命令只能对一种或两种命令有作用。在后面会介绍。

​	string ，list，hash 这三种结构和其他很多编程语言中的大致相同，set 也是某种集合数据结构(如java)。zset在某种程度上来说是Redis特有的一种结构。下面给出它们的异同点

1. string ： **可以是字符串，整数或者浮点数**；可对整个字符串或者字符串中的一部分执行操作，对整数和浮点数执行自增(increment) 或者自减(decrement) 操作。
2. list ：**一个链表，链表上的每个节点都包含了一个字符串**；可以从链表的两端推入或者弹出元素，根据偏移量对链表进行修剪(*trim*) ，读取单个或多个元素，根据值查找或者移除元素
3. set ：包含字符串的**无序收集器**(unordered collection) 并且被包含的每个字符串都是独一无二的。可以添加、获取、移除单个元素，检查一个元素是否在集合内，计算交集、并集、差集，从集合里面随机获取元素
4. hash：**包含键值对的无序散列表**。可以添加、获取、移除单个键值对，获取所有键值对
5. zset ：**字符串成员(member)与浮点数分值(score)之间的有序映射，元素的排列由分值大小决定**。可以添加、获取、删除单个元素，根据分值范围或者成员来获取数据



##### Redis 中的字符串

​	字符串和java中的map很像，但是拥有一些存储相似的命令，比如 **get set del** 命令，下面是在redis-cli内的演示代码以及解释。

>set ping lin    #将键 ping 的值设为 lin
>
>​	ok
>
>get ping        #获取ping键的值
>
>​	"lin" 
>
>del ping       #删除键值对
>
>​	(integer)1   #在键值对删除后，del命令将返回被成功删除的值的数量
>
>get ping  #由于键已经不存在，所以返回nil，如果在java客户端 是返回 null
>
>​	(nil)



##### Redis 中的列表

​	Redis 对链表(linked-list) 结构的支持使得它在键值存储世界中独树一帜。一个列表结构可以有序的存储多个字符串。Redis列表可执行的操作和很多编程语言里面的列表操作非常相似：下面是一些命令

 - **lpush ，rpush**：分别用于将元素推入列表左端和右端

 - **lpop，rpop** ：分别从列表左端和右端弹出元素

 - **lindex** ：用于获取列表在给定位置上的一个元素

 - **lrange** ： 用于获取列表在给定范围上的所有元素

   ![23.png](https://github.com/MrAlan/MyPostPicture/blob/master/23.png?raw=true)

   还有一些其他的命令之后再了解



##### Redis中的集合

​	Redis的集合和列表都可以存储多个字符串，他们之间的不同之处在于：列表可以存储多个相同的字符串，集合则通过使用散列表来保证自己存储的每个字符串都是各不相同的(这些散列表只有键，没有值)。因为Redis的集合使用了无序方式存储，所以不能像列表那样将元素push进一端再弹出。下面是一些命令

  - **sadd** ：将元素添加到集合中

  - **srem** ：从集合中移除元素

  - **sismember** : 快速检查一个元素是否已经存在于集合中

  - **smembers** ：获取集合中所有元素 (如果集合中元素很多，速度会很慢)

    ![24.png](https://github.com/MrAlan/MyPostPicture/blob/master/24.png?raw=true)

##### Redis的散列

​	Redis 的散列可以存储多个键值对之间的映射，它的值既可以是字符串也可以是数值，并且用户同样可以对散列存储的数字执行自增操作或者自减操作。下面是一些命令

 - **hset**：在散列里面关联起给定的键值对

 - **hget**：获取指定键值对的值

 - **hgetall**：获取散列包含的所有键值对

 - **hdel**：如果给定键值存在于散列中，那么移除这个键

   ![25.png](https://github.com/MrAlan/MyPostPicture/blob/master/25.png?raw=true)



##### Redis的有序集合

​	有序集合和散列一样都用于存储键值对；有序集合的键被称为**成员**(member),每个成员都是各不相同的；而有序集合的值被称为**分值**(score),分值必须为浮点数。zset是Redis里面唯一一个既可以根据成员访问元素，又可以根据分值以及分值的排列来访问元素的结构。下面是一些命令

 - **zadd**：讲一个带有给定分值的成员添加到有序集合中

 - **zrange**：根据元素在有序排列中的位置，从有序集合中获取多个元素

 - **zrangebyscore**：获取有序集合在给定分值范围内的所有元素

 - **zrem**：如果给定成员存在于有序集合，那么移除这个成员

   ![26.png](https://github.com/MrAlan/MyPostPicture/blob/master/26.png?raw=true)



#### 你好Redis

​	在对Redis的5种结构有了基本的了解之后， 现在是时候来学习一下怎么使用这些结构来解决实际问题了。最近几年，越来越多的网站开始提供对网页链接，文章或者问题进行投票功能：这些网站会根据文章的发布时间和文章获得的投票数量计算一个评分，然后按照这个评分来决定如何排序和展示文章。

​	本节将展示如何使用Redis来构建一个简单的文章投票网站的后台。(ps: 书中是python的代码，我主意是想用java，最近又在看SpringBoot所以我想尝试着把代码改成SpringBoot框架下的java代码)。

##### 对文章进行投票

​	要构建一个文章投票网站，我们首先要做的是为了这个网站设置一些数值和限制条件：如果一篇文章获得了至少两百张支持票，那么就认为这篇文章是有趣的文章；假如这个网站每天发布1000篇文章，其中50篇符合网站对有趣文章的要求，那么网站要做的就是把这50篇文章放到文章列表前100位至少1天；另外，这个网站暂时不提供反对票的功能。

​	为了产生一个能够随时间流逝而不减少的评分，程序需要根据文章的发布时间和当前时间计算文章的评分，算法为：将文章得到的支持票数乘以一个常量，然后加上文章发布的时间，得出的结构就是文章的评分。

​	我们使用**UTC时区1970年1月1日到目前为止的经过的秒数来计算时间**，这个值通常被称为**Unix时间**。因为Unix时间能够用任何编程语言轻易获得。另外，常量的计算时通过 **将一天的秒数(86400)除以展示一天所需要的支持票数(200)得出 为 432** 。即文章每获得一张支持票，程序就将文章的评分增加432分。

​	构建文章投票网站除了需要计算文章评分之外，还需要使用Redis结构存储网站上的各种信息。对于网站里的每篇文章，程序都使用一个散列来存储文章的标题，指向文章的网址，发布的用户，发布的时间，得到的投票数等信息。一个例子如下图示

![](https://github.com/MrAlan/MyPostPicture/blob/master/32.png?raw=true)

	>	本书使用冒号来分割名字的不同部分；比如上图中article:92617 分割了单词 article和文章ID 92617 ，以此来购进命名空间(namespace) 。使用 ： 作为分隔符只是个人习惯，不过大部分Redis用户也都是这么做的。还有一些常见的分隔符如 句号( 。)，斜线( / ) 。无论哪种都需要保持分隔符一致。同时请读者观察注意和学习本书使用冒号创建嵌套命名空间的方法。

​	我们的文章投票网站将使用两个有序集合来有序地存储文章：第一个有序集合的成员为**文章ID**，分值为文章发布的时间；第二个有序集合的成员同样是**文章ID**，分值为文章评分。通过这两个有序集合，网站既可以根据文章发表的先后顺序来展示文章，又可以根据文章评分高低来展现文章。如下面两个图示：time和score 为有序集合名字也代表了分值，article：id 是成员   

![](https://github.com/MrAlan/MyPostPicture/blob/master/33.png?raw=true)![](https://github.com/MrAlan/MyPostPicture/blob/master/34.png?raw=true)

​	为防止用户对同一篇文章给你进行多次投票，网站需要为每篇文章记录一个已投票用户的名单。为此，程序将为每篇文章创建一个集合，并使用这个集合来存储所有已投票用户的ID。如下面的表格

| voted :100408         set |
| ------------------------- |
| user:234487               |
| user:253378               |
| user:363680               |
| ....                      |

​	知道了网站计算文章评分的方法，也知道了网站存储数据所需要的数据结构，那么现在是时候来实际实现投票这个功能了！当用户尝试对一篇文章投票时，程序需要使用**ZSCORE** 命令检查记录文章发布时间的有序集合，判断文章的发布时间是否超过一周。**如果文章仍然处于可以投票的时间范围内，那么程序将使用SADD命令**，尝试将用户添加到记录文章已投票用户名单的集合中，如果操作执行成功，那么说明用户时第一次对这篇文章进行投票，程序将使用**ZINCRBY** 命令为文章的评分增加432(**zincrby 命令用于对有序集合成员的分值执行自增操作**) , 并使用**HINCRBY**命令对散列记录的文章投票数量进行更新。



##### 如何在SpringBoot里使用Redis

[本小节内容来自篇文章：SpringBoot 中使用Redis数据库](http://blog.didispace.com/springbootredis/)

前置：使用IDEA选择Spring Initialzr ，选择web和nosql里的redis。

 	1. 引入依赖，SpringBoot 提供**数据访问框架 Spring Data Redis** 基于 **Jedis**。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-redis</artifactId>
</dependency>
```

2. 参数配置，在application.properties 中加入Redis 服务端的相关配置，可以设置数据库的数量，默认为16个。

```properties
# REDIS (RedisProperties)
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=localhost
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=0
```

3. 编写测试用例，访问 Redis

```java
package com.example.springbootredis;

import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootredisApplicationTests {

  @Autowired
  private StringRedisTemplate stringRedisTemplate;

  @Test
  public void test() {
    stringRedisTemplate.opsForValue().set("aaa","111");
    Assert.assertEquals("111",stringRedisTemplate.opsForValue().get("aaa"));

  }

}

```

​	上面的代码是一段极为简单的测试案例，演示了如何自动配置的 **StringRedisTemplate** 对象进行Redis的读写操作，该对象的命名中就可以知道支持String，在**Spring-data-redis中是 RedisTemplate<K,V> 接口**，二者实际上是相通的。

​	除了String类型外，我们还会在Redis中存储对象，*在spring-data-redis 中是使用类似于 RedisTemplate<String，User> 来初始化并操作*，但是Spring Boot 中并不支持。这需要我们自己实现 RedisSerializer<T> 接口 来对传入对象进行序列化和反序列化，下面是个例子：

​	1. 创建要存储的对象User,要实现 Serializable

```java
package com.example.springbootredis.storeobject;

public class User implements Serializable{
  private static final long serialVersionId=-1L;
 
  private String  username;
  private int age;
  
  public User(String username,int age){
    this.username=username;
    this.age=age;
  }

  public String getUsername() {
    return username;
  }

  public void setUsername(String username) {
    this.username = username;
  }

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }
}

```

2. 实现对象(Object)的序列化接口 , 包括反序列化

```java
public class RedisObjectSerializer implements RedisSerializer<Object> {

  private Converter<Object, byte[]> serializer = new SerializingConverter();
  private Converter<byte[], Object> deserializer = new DeserializingConverter();

  static final byte[] EMPTY_ARRAY = new byte[0];

  public Object deserialize(byte[] bytes) {
    if (isEmpty(bytes)) {
      return null;
    }

    try {
      return deserializer.convert(bytes);
    } catch (Exception ex) {
      throw new SerializationException("Cannot deserialize", ex);
    }
  }

  public byte[] serialize(Object object) {
    if (object == null) {
      return EMPTY_ARRAY;
    }

    try {
      return serializer.convert(object);
    } catch (Exception ex) {
      return EMPTY_ARRAY;
    }
  }

  private boolean isEmpty(byte[] data) {
    return (data == null || data.length == 0);
  }
}
```

3. 配置针对User和RedisTemplate 实例

```java
package com.example.springbootredis;

import com.example.springbootredis.storeobject.RedisObjectSerializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

  @Bean
  public RedisTemplate<String, Object> redisTemplate(
      RedisConnectionFactory redisConnectionFactory) {

    RedisTemplate<String, Object> template = new RedisTemplate<>();

    template.setConnectionFactory(redisConnectionFactory);
    template.setKeySerializer(new StringRedisSerializer());
      
    template.setValueSerializer(new RedisObjectSerializer()); //这个是我们定义的
      
    template.setHashKeySerializer(new GenericJackson2JsonRedisSerializer());
    template.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
    template.afterPropertiesSet();

    return template;
  }
}
```

4. 编写测试用例，将之前创建的改写。

```java

package com.example.springbootredis;

import com.example.springbootredis.storeobject.User;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootredisApplicationTests {

  @Autowired
  private StringRedisTemplate stringRedisTemplate;

  @Autowired
  private RedisTemplate<String, Object> redisTemplate;
  /*
   redisTemplate.opsForValue();//操作字符串
   redisTemplate.opsForHash();//操作hash
   redisTemplate.opsForList();//操作list
   redisTemplate.opsForSet();//操作set
   redisTemplate.opsForZSet();//操作有序set
  */
    
  @Test
  public void test() {
    stringRedisTemplate.opsForValue().set("aaa","111");
    Assert.assertEquals("111",stringRedisTemplate.opsForValue().get("aaa"));

  }

  @Test
  public void test2()throws Exception{
    User user=new User("超人",20);
    redisTemplate.opsForValue().set(user.getUsername(),user);
    Assert.assertEquals(20,User(redisTemplate.opsForValue().get("超人")).getAge());
  }
}

```

更多内容参考 [spring-data-redis 官方文档](https://docs.spring.io/spring-data/redis/docs/2.1.2.RELEASE/reference/html/)



---

下面的代码展示了投票功能的实现代码。(原书中是python，这里就用springboot)

