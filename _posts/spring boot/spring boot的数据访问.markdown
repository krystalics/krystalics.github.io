<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [第八章 ——Spring Boot 的数据访问](#%E7%AC%AC%E5%85%AB%E7%AB%A0-spring-boot-%E7%9A%84%E6%95%B0%E6%8D%AE%E8%AE%BF%E9%97%AE)
  - [8.1 Docker 引入](#81-docker-%E5%BC%95%E5%85%A5)
  - [8.2 Spring Data JPA](#82-spring-data-jpa)
    - [8.2.1 点睛](#821-%E7%82%B9%E7%9D%9B)
  - [8.2.3 实战](#823-%E5%AE%9E%E6%88%98)
  - [8.3 Spring Data REST](#83-spring-data-rest)
    - [8.3.1 点睛 Spring Data REST](#831-%E7%82%B9%E7%9D%9B-spring-data-rest)
    - [8.3.2 Spring Boot的支持](#832-spring-boot%E7%9A%84%E6%94%AF%E6%8C%81)
  - [8.4 声明式事务](#84-%E5%A3%B0%E6%98%8E%E5%BC%8F%E4%BA%8B%E5%8A%A1)
    - [8.4.1 Spring 的事务机制](#841-spring-%E7%9A%84%E4%BA%8B%E5%8A%A1%E6%9C%BA%E5%88%B6)
    - [8.4.2声明式事务](#842%E5%A3%B0%E6%98%8E%E5%BC%8F%E4%BA%8B%E5%8A%A1)
    - [8.4.3 注解事务行为](#843-%E6%B3%A8%E8%A7%A3%E4%BA%8B%E5%8A%A1%E8%A1%8C%E4%B8%BA)
    - [8.4.4 Spring Data JPA的事务支持](#844-spring-data-jpa%E7%9A%84%E4%BA%8B%E5%8A%A1%E6%94%AF%E6%8C%81)
    - [8.4.5 自动开启注解事务的支持](#845-%E8%87%AA%E5%8A%A8%E5%BC%80%E5%90%AF%E6%B3%A8%E8%A7%A3%E4%BA%8B%E5%8A%A1%E7%9A%84%E6%94%AF%E6%8C%81)
    - [8.4.6 实战](#846-%E5%AE%9E%E6%88%98)
  - [8.5 数据缓存 Cache](#85-%E6%95%B0%E6%8D%AE%E7%BC%93%E5%AD%98-cache)
    - [Spring Boot支持下的缓存](#spring-boot%E6%94%AF%E6%8C%81%E4%B8%8B%E7%9A%84%E7%BC%93%E5%AD%98)
    - [实战](#%E5%AE%9E%E6%88%98)
    - [更换缓存](#%E6%9B%B4%E6%8D%A2%E7%BC%93%E5%AD%98)
  - [8.6 非关系型数据库NoSQL](#86-%E9%9D%9E%E5%85%B3%E7%B3%BB%E5%9E%8B%E6%95%B0%E6%8D%AE%E5%BA%93nosql)
    - [8.6.1 MongoDB](#861-mongodb)
    - [8.6.2 Redis](#862-redis)
    - [SpringBoot对Redis的支持](#springboot%E5%AF%B9redis%E7%9A%84%E6%94%AF%E6%8C%81)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

*《Spring Boot实战》汪云飞*

---

### 第八章 ——Spring Boot 的数据访问

​	Spring Data 项目是Spring用来解决数据访问问题的一揽子解决方案，Spring Data 是一个伞形项目，包含了大量关系型数据可的数据访问方案。Spring Data 使我们可以快速而简单的使用新的数据访问技术。

​	Spring Data包括的子项目有很多，比如： **JPA , MogoDB ，Neo4j，Redis，Solr，Hadoop，REST，JDBC Extensions ，CouchBase....等**

​	Spring Data为我们提供统一的API来对上述数据存储技术进行数据访问操作提供了支持。这是通过Spring Data Commons 项目来实现的，它是上述各种项目的依赖。让我们在使用关系型数据库和非关系型数据库访问技术时都使用基于Spring的统一标准，包含CRUD，查询，排序和分页的相关操作。

​	Spring Data Commons 其中一个重要概念：Spring Data Repository 抽象。使用它可以极大减少数据访问层的代码。根接口是



```java
public interface Repository<T,ID extends Serializable>{
    
}
```



​	从源码中看，它接受领域类(JPA为实体类)，和领域类的ID类型为参数。它的子接口CrudRepository定义了和CRUD操作有关的内容



```java
@NoRepository
public interface CrudRepository<T,ID extends Serializable> implements Repository<T,ID>{
    <S extends T>S save (S entity);
    <S extends T>Iterable<S> save (Iterable<S> entities);
    T findOne(ID id);
    boolean exists(ID id);
    Iterable<T> findAll();
    Iterable<T> findAll(Iterable<ID> ids);
    long count();
    void delete(ID id);
    void delete(T entity);
    void delete(Iterable<? extends T> entities);
    void deleteAll();
}
```



​	CrudRepository的子接口 PagingAndSortingRepository 定义了分页和排序操作的相关内容

​	

```java
@NoRepository
public interface PagingAndSortingRepository <T,ID extends Serializable> implements CrudRepository<T,ID>{
    Iterable<T> findAll(Sort sort);
    Page<T> findAll(Pageable pageable);
}
```



​	不同的数据访问技术也提供了不同的Repository，如 JpaRepository和 MongoRepository 。还有一个激动人心的功能，即可以根据属性名进行计数，删除，查询方法等操作。例如



```java
public interface PersonRepository implements Repository<Person,Long>{
	Long countByAge(Integer age); //按照年龄计数
    Long deleteByName(String name); //按照名字删除
    List<Person> findByName(String name); //按照名字查询
    List<Person> findByNameAndAddress(String name,String address); //按照名字和地址查询
}
```



​	会在之后的章节对其进行更详细的分析。



#### 8.1 Docker 引入

​	 <a href="https://krystalics.github.io/2018/12/05/About-Docker/">查看我关于Docker的文章</a>

#### 8.2 Spring Data JPA

##### 8.2.1 点睛

​	在介绍什么是Spring Data JPA时，先认识一下什么是 Hibernate。Hibernate是数据访问解决技术的绝对霸主，使用O/R映射（Object-Relaional Mapping）技术实现数据访问，O/R映射将领域类模型和数据库的表进行映射，通过程序操作对象而实现表数据操作的能力，让数据访问操作无须关注数据库相关技术。

​	随着Hibernate的盛行，Hibernate主导了EJB3.0的JPA规范，JPA即 Java Persistence API ，JPA是一个基于O/R映射的标准规范。所谓规范即只定义标准规则(如注解，接口)，不提供实现**，软件提供商可以按照标准规范来实现，而使用者只需要按照其中定义的方法来使用**，而不用和软件提供商打交道。JPA的主要实现有Hibernate，EclipseLink和OpenJPA等。

​	同时上文中也提到过，Spring Data JPA是Spring Data的一个子项目，提供基于JPA的Repository极大的减少了JPA作为数据访问方案的代码量。

**定义数据访问层**

​	使用Spring Data JPA建立数据访问层十分简单，只需要定义一个实现JPARepository的接口即可。JPARepository代码如下：



```java
@NoRepository
public interface JpaRepository<T,ID extends Serializable> implements PagingAndSortingRepository <T,ID>{
    List <T> findAll();
    List<T> findAll(Sort sort);
    List<T> findAll(Iterable<ID> ids);
    <S extends T> List<S> save(Iterable<S>entities);
    void flush();
    <S extends T>S saveAndFlush(S entity);
    void deleteInBatch(Iterable<T> entities);
    void deleteAllInBatch();
    T getOne(ID id);
}
```



**配置使用Spring Data JPA**

​	在Spring环境中，可通过@EnableJpaRepository 注解来开启Spring Data JPA的支持。还需要做一些工作。而Spring Boot当然已经给我们配好了。

**1.JDBC支持**：spring-boot-starter-data-jpa 依赖于 spring-boot-starter-jdbc ，而Spring Boot对JDBC做了一些自动配置。比如放在类路径下的schema.sql文件会自动用来初始化表结构，data.sql文件会自动用来填充表数据。

**2.对JPA的自动配置**：Spring Boot默认JPA的实现者是Hibernate，配置JPA可以使用spring.jpa为前缀的属性在application.properties 中配置。在web项目中我们经常会遇到在控制器或者页面访问数据的时候出现会话连接已关闭的错误，这时候会自动配置一个OpenEntityManagerInView 这个Bean(作为过滤器)。并注册到Spring MVC的拦截器中。



#### 8.2.3 实战

​	在本节的实战中，我们将演示基于方法名的查询，基于@Query的查询，分页及排序，最后我们将结合Specification和自定义Repository实现来完成一个通用实体查询，即对于任意类型的实体类的传值对象，只要有值的属性我们就进行自动构造查询(字符型用like,其他类型用 =)

1.原书中是安装Oracle XE。这里我们就使用MySQL5.7.23。安装的话自己百度就可以了。

具体的见[spring boot mysql入门  ](https://segmentfault.com/a/1190000014244807)

运行之后会发现有下图错误

<img src="https://github.com/krystalics/mypostpicture/blob/master/45.png?raw=true">

因为我使用了当前最新的mysql-connector-java-8.0.13.jar的MySQL驱动包，新的驱动包中`com.mysql.jdbc.Driver`类已经过时，新的`com.mysql.cj.jdbc.Driver`通过SPI自动注册，不再需要手动加载驱动类。我们需要将 ?serverTimezone=GMT%2B8 时区设置也加入到DB_url中。修改application.properties如下。之后即可按照文章运行正常。

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/db_example   改为下面样式

spring.datasource.url=jdbc:mysql://localhost:3306/db_example？serverTimezone=GMT%2B8      //GMT%2B8 代表东8区
```



#### 8.3 Spring Data REST

##### 8.3.1 点睛 Spring Data REST

**1.什么是Spring Data REST**

​	Spring Data JPA 是基于Spring Data的repository之上，可以将repository自动输出为REST资源。目前Spring Data REST支持将Spring Data JPA，Spring Data MongoDB，等的repository自动转为REST服务。

至于什么是REST，请查看我的另一篇文章：<a href="https://krystalics.github.io/2018/12/03/About-REST/">About Rest</a>

**Spring MVC中配置使用Spring Data REST** 与Spring Boot中使用方式一致

- 继承方式演示

```java
@Configuration
public class MyRepositoryRestMvcConfiguration extends RepositoryRestMvcConfiguration{
    @Override
    public RepositoryRestConfiguration config(){
        return super.config();
    }
}
```

- 导入方式演示

```java
@Configuration
@Import(RepositoryRestMvcConfiguration.class)
public class AppConfig{
    
}
```



##### 8.3.2 Spring Boot的支持

​	在application.properties 中配置以 spring.data.rest 配置 RepositoryRestMvcConfiguration 

1.实体类

```java
package com.example.data_ch8.rest;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Person {

  @Id
  @GeneratedValue
  private Long id;

  private String name;

  private Integer age;

  private String address;

  public Person() {
    super();
  }

  public Person(Long id, String name, Integer age, String address) {
    this.address = address;
    this.id = id;
    this.age = age;
    this.name = name;
  }

  public Long getId() {
    return id;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public Integer getAge() {
    return age;
  }

  public void setAge(Integer age) {
    this.age = age;
  }

  public String getAddress() {
    return address;
  }

  public void setAddress(String address) {
    this.address = address;
  }
}
```



2.实体类的 Repository

```java
package com.example.data_ch8.rest;

import org.springframework.data.jpa.repository.JpaRepository;

public interface PersonRepository extends JpaRepository<Person,Long> {
  Person findByNameStartingWith(String name);
}
```



3.安装Chrome插件 Postman REST Client ：本节会将postman插件放在源码中，src/main/resources目录下。实际上postman现在已经被废弃了，建议使用postman的客户端。



4.REST服务测试：**Spring Data REST的默认规则是在实体类后加s来形成路径**，如Person => persons ，User=> users .

- jQuery：在实际开发中，在jQuery我们使用$.ajax方法的type属性来修改提交方法：

```js
$.ajax({
    type:"GET",
    dataType:"json",
    url:"http://localhost:10086/persons",
    success:function(data){
        alert(data);
    }
})
```

- Angular ：在实际开发中，可以用$http对象来操作

```js
$http.get(url)
$http.post(url,data)
$http.put(url,data)
$http.delete(url)
```

- 列表：在postman中使用GET 访问http://localhost:10086/persons 并获得列表数据。

<img src="https://github.com/krystalics/mypostpicture/blob/master/46.png?raw=true">

5.定制

- 定制根路径：我们访问路径一般是类路径，如何定制呢？

```properties
spring.data.rest.base-path=/api  就像 
server.servlet.context-path=/api 
```

- 定制节点路径：就像上文所述中，persons是访问Person的路径，而如果要修改这个。因为person的复数是people，需要在实体类的Repository上使用 @RepositoryRestResource 注解的path来进行修改 。这个时候访问路径就变成 localhost:10086/api/people 

```java
@RepositoryRestResource
public interface PersonRepository extends JpaRepository<Person,Long>{
    @RestResource(path="nameStartsWith",rel="nameStartsWith")
    Person findByNameStartsWith(@Param("name")String name);
}
```



#### 8.4 声明式事务

##### 8.4.1 Spring 的事务机制

​	所有的数据访问技术都有事务处理机制，这些技术提供了API用来开启事务，提交事务来完成数据操作，或者发生错误的时候回滚数据。

​	而Spring的事务机制是用统一的机制来处理不同数据访问技术的事务处理。Spring的事务机制提供了PlatformTransactionManager接口，不同的数据访问技术事务的接口如下表

| 数据访问技术 | 实现                         |
| ------------ | ---------------------------- |
| JDBC         | DataSourceTransactionManager |
| JPA          | JpaTransactionManager        |
| Hibernate    | HibernateTransactionManager  |
| JDO          | JdoTransactionManager        |
| 分布式事务   | JtaTransactionManager        |

在程序定义事务管理器的代码如下：



```java
@Bean
public PlatformTransactionManager transactionManager(){
    JpaTransactionManager transactionManager=new JpaTransactionManager();
    transactionManager.setDataSource(dataSource());
    return transactionManager;
}
```



##### 8.4.2声明式事务

​	Spring支持声明式事务，即使用注解来选择需要使用事务的方法，它使用@Transactional注解在方法上表明该方法**需要事务支持**，这是一个基于AOP的实现操作，被注解的方法在被调用的时候，Spring 开启一个新事务，当方法无异常运行结束后，Spring会提交这个事务。



```java
@Transactional //来自org.springframework.transaction.annotation
public void saveSomething(Long id,String name){
    
}
```



Spring 提供了@EnableTransactionManagement 注解在配置类上来开启声明式事务的支持。使用了之后，会自动扫描注解了@Transactional的方法和类。如果在某个类上用了这个注解，那么这个类中所有的public方法都是开启事务的。如果在类和方法上同时使用该注解，则相当于方法的注解重载了类的注解，以方法的注解为准。



##### 8.4.3 注解事务行为

​	@Transactional 有很多属性来定制事务行为，比如 

- propagationtion 定义了事务的生命周期
- isolation 隔离，决定了事务的完整性，主要包含4个隔离级别，这个还是要看数据库的支持
- timeout 事务过期时间
- readOnly :指定为只读事务  默认为false
- roolbackFor 指定哪些异常可以引起回滚
- noRollbackFor ： 指定哪些不引起回滚

 

##### 8.4.4 Spring Data JPA的事务支持

​	 Spring Data JPA对所有的默认方法都开启了事务支持，且查询类事务默认启用 readOnly=true属性。



##### 8.4.5 自动开启注解事务的支持

​	TransactionAutoConfiguration 依赖于 JpaBaseConfiguration和DataSourceTransactionManagerAutoConfiguration 。已经自动配置了，无须开启@EnableTransactionManagement。



##### 8.4.6 实战

依托于上文中的 Person和 PersonRepository 类。

-  业务服务Service

```java
package com.example.data_ch8.transaction;

import com.example.data_ch8.rest.Person;

public interface DemoService {

  Person savePersonWithRollBack(Person person);

  Person savePersonWithoutRollBack(Person person);
}
```

- 实现 DemoService接口

```java
package com.example.data_ch8.transaction;

import com.example.data_ch8.rest.Person;
import com.example.data_ch8.rest.PersonRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class DemoServiceImp implements DemoService {

  @Autowired
  PersonRepository personRepository;

  @Transactional(rollbackFor = {IllegalArgumentException.class}) // 如果遇到参数错误就回滚
  @Override
  public Person savePersonWithRollBack(Person person) {
    Person p=personRepository.save(person);

    if (person.getName().equals("汪云飞")){
      throw new IllegalArgumentException("汪云飞已经存在");
    }
    return p;
  }

  @Transactional(noRollbackFor = {IllegalArgumentException.class}) // 如果不是遇到参数错误 就回滚
  @Override
  public Person savePersonWithoutRollBack(Person person) {
    Person p=personRepository.save(person);

    if(person.getName().equals("汪云飞")){
      throw new IllegalArgumentException("汪云飞虽以存在，但是数据不会回滚"); 
      //这样就 有两个汪云飞了
    }
    return p;
  }
}
```

- 控制器

```java
package com.example.data_ch8.transaction;

import com.example.data_ch8.rest.Person;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TranController {
  @Autowired
  DemoService demoService;

  @RequestMapping("/rollback")
  public Person rollback(Person person){
    return demoService.savePersonWithRollBack(person);
  }

  @RequestMapping("/norollback")
  public Person noRollback(Person person){
    return demoService.savePersonWithoutRollBack(person);
  }

}
```



#### 8.5 数据缓存 Cache

​	当我们知道一个程序的瓶颈在数据库，需要重复获取相同数据的时候，数据缓存起到了重大作用。

Spring 支持CacheManager，针对不同的缓存技术，定义了如下实现



| CacheManager              | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| SimpleCacheManager        | 使用简单的Collection来存储缓存，主要用来测试                 |
| ConcurrentMapCacheManager | 使用ConcurrentMap来存储缓存                                  |
| NoOpCacheManager          | 仅用于测试，不做实际缓存                                     |
| EhCacheCacheManager       | 使用EhCache作为缓存技术                                      |
| GuavaCacheManager         | 使用Google Guava 的GuavaCache作为缓存技术                    |
| HazelcastCacheManager     | 使用 Hazelcast作为缓存技术                                   |
| JCacheCacheManager        | 支持JCache(JSR-107)标准的实现作为缓存技术，如Apache Commons JCS |
| RedisCacheManager         | 使用Redis作为缓存技术                                        |

在使用任意一个CacheManager的时候都要注册相应的Bean。

```java
@Bean 
public EhCacheCacheManager manager(CacheManager m){
    return new EhCacheCacheManager(m);
}
```

当然，每种缓存技术都是有很多的额外配置的，但是CacheManager是必不可少的。



**声明式缓存注解**

​	Spring 提供4个注解来声明缓存规则(又是使用注解式的AOP的一个生动例子)：而且Spring要开启声明式缓存的支持，就要在配置类里加 @EnableCaching  。



| 注解        | 解释                                                         |
| ----------- | ------------------------------------------------------------ |
| @Cacheable  | 在方法执行前，Spring先查看缓存中是否有数据，如果没有则调用方法并将其存入缓存，有则直接返回缓存数据 |
| @CachePut   | 无论怎么样都会讲方法的返回值放到缓存中。属性与@Cacheable保持一致 |
| @CacheEvict | 将一条或多条数据冲缓存中删除                                 |
| @Caching    | 可以通过@Caching注解这多个注解策略在一个方法上               |

前三个都是有value属性指定要使用的缓存名字，key 属性指定数据在缓存中存储的键。如：

```java
@Cacheable(value="RedisCacheManager",key="id")
public...
```



##### Spring Boot支持下的缓存

​	springboot自动为我们配置了多个CacheManager的实现。在application.properties中以如下形式配置：只需要引入相关的依赖包和配置类中使用@EnableCaching就可以。

```properties
spirng.cache.type= 可选redis，generic，ehcache，....
spring.cache.cache-name=程序启动时创建的缓存名称
spirng.cache.ehcache.config= ehcache配置文件地址
spirng.cache.infinispan.config=
spirng.cache.jcache.config=
spirng.cache.jcache.provider= 多个jcache实现在类路径中的时候，指定jcache
```



##### 实战

​	借用上文的Person和Repository。

- 接口定义

```java
package com.example.data_ch8.cache;

import com.example.data_ch8.rest.Person;

public interface CacheService {
  Person save(Person person);
  void remove(Long id);
  Person findOne(Person person);
}
```

- 实现接口

```java
package com.example.data_ch8.cache;

import com.example.data_ch8.rest.Person;
import com.example.data_ch8.rest.PersonRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class CacheServiceImp1 implements CacheService {

  @Autowired
  PersonRepository personRepository;
  
  @Override
  @CachePut(value = "people",key = "#person.getId()")  //缓存新增或者更新的数据到缓存，名称为people，key是person.id
  public Person save(Person person) {
    Person p=personRepository.save(person);
    System.out.println("为id，key为："+p.getId()+"数据做了缓存");
    return p;
  }

  @Override
  @CacheEvict(value = "people") //从缓存people中删除key为 id的键
  public void remove(Long id) {
    System.out.println("删除了id,key为"+id+"的数据缓存");
    personRepository.deleteById(id);
  }

  @Override
  @Cacheable(value = "people",key = "#person.getId()") //缓存key为person.id的键到people中
  public Person findOne(Person person) {
    Person p = personRepository.findById(person.getId()).get();
    System.out.println("为id，key为："+p.getId()+"数据做了缓存");
    return p;
  }
}
```

- 控制器

```java
package com.example.data_ch8.cache;

import com.example.data_ch8.rest.Person;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class CacheController {
  @Autowired
  CacheService cacheService;

  @RequestMapping("/put")
  public Person put(Person person){
    return cacheService.save(person);
  }

  @RequestMapping("/able")
  public Person cacheable(Person person){
    return cacheService.findOne(person);
  }

  @RequestMapping("/evit")
  public String evit(Long id){
    cacheService.remove(id);
    return "ok";
  }

}
```

- 最后在主类或配置类中开启缓存支持，然后运行既可。



##### 更换缓存

​	只需要将对应依赖包加入到pom.xml中既可。有配置文件的要将其放在类路径下。



#### 8.6 非关系型数据库NoSQL

​	特点是不使用SQL语言作为查询语言，数据存储也不是固定的表，字段。主要有文档存储型(MongoDB)，图形关系存储型(Neo4j)和键值对存储型(Redis)。本节将演示基于MongoDB的数据访问和就要Redis的数据访问



##### 8.6.1 MongoDB

​	基于文档的存储型数据库，使用面向对象的思想，每一条数据记录都是文档的对象。

​	spring 对MongoDB的支持是通过Spring Data MongoDB来实现的。也是要开启支持，增加@EnableMongoRepositories 到配置类。还提供了如下功能

- Object/Document 映射注解支持

| 注解      | 描述                            |
| --------- | ------------------------------- |
| @Document | 映射领域对象与MongoDB的一个文档 |
| @Id       | 映射当前属性时Id                |
| @DbRef    | 当前属性将参考其他文档          |
| @Field    | 为文档的属性定义名称            |
| @Version  | 将当前属性作为版本              |

- MongoTemplate

和下面对Redis的支持差不多，先连接MongoClient以及MogoDbFactory

```java
@Bean
public MongoClient client()throws UnknownHostException{
    MongoClient client=new MongoClient(new ServerAddress("127.0.0.1",27017));
    return client;  //上面设置ip地址和端口
}

@Bean
public MongoDbFactory mongodb()throws Exception{
    String database=new MongoClientURI("mongodb://localhost/test").getDataBase();
    return new SimpleMongoDbFactory(client(),database); //上面创建一个test数据库
}
```

- Repository的支持

```java
public interface PersonRepository extends MongoRepository<Person,String>{}
```



**Spring Boot 的支持**

​	application.properties中的配置如下

```properties
spring.data.mongodb.host= #默认为localhost
spring.data.mongodb.port=27017 #默认
spring.data.mongodb.url=mongodb://localhost/test #连接URL
spring.data.mongodb.database=
spring.data.mongodb.authentication-database=
spring.data.mongodb.grid-fs-database=
spring.data.mongodb.username=
spring.data.mongodb.password=
spring.data.mongodb.repositories.enabled=true #默认是开启的
... 
```

- 安装MongoDB

- 引入spring-boot-starter-data-mongodb依赖

- 建立领域模型，Teacher，包含工作过的地点Location

```java
package com.example.data_ch8.mongo;

import java.util.Collection;
import java.util.LinkedHashSet;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.Field;

@Document //注解映射领域模型和 MongoDB文档
public class Teacher {
  @Id //表明这个属性时文档的id
  private String id;
  private String name;
  private int age;

  @Field("locs") //此属性在文档中名为 locs ，locations属性将以数组形式存在当前数据记录中
  private Collection<Location> locations=new LinkedHashSet<>();

  public Teacher(String name,int age){
    this.name=name;
    this.age=age;
  }

  public String getId() {
    return id;
  }

  public void setId(String id) {
    this.id = id;
  }

  public Collection<Location> getLocations() {
    return locations;
  }

  public void setLocations(Collection<Location> locations) {
    this.locations = locations;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }
}
```

```java
package com.example.data_ch8.mongo;

public class Location {

  private String place;
  private String year;

  public Location(String place, String year) {
    this.place = place;
    this.year = year;
  }

  public String getPlace() {
    return place;
  }

  public void setPlace(String place) {
    this.place = place;
  }

  public String getYear() {
    return year;
  }

  public void setYear(String year) {
    this.year = year;
  }
}
```

- 数据访问接口

```
package com.example.data_ch8.mongo;

import java.util.List;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;

public interface TeacherRepository extends MongoRepository<Teacher, String> {

  Teacher findByName(String name);  //支持方法名查询

  @Query("{'age':?0}") //支持@Query查询，查询参数构造JSON字符串即可
  List<Teacher> withQueryFindByAge(int age);
}
```

- 控制器

```java
package com.example.data_ch8.mongo;

import java.util.Collection;
import java.util.LinkedHashSet;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TeacherContorller {
  @Autowired
  TeacherRepository teacherRepository;

  @RequestMapping("/save")
  public Teacher save(){
    Teacher t=new Teacher("ljb",32);

    Collection<Location> locations=new LinkedHashSet<>();
    Location loc1=new Location("上海","2009");
    Location loc2=new Location("合肥","2014");
    Location loc3=new Location("广州","2015");
    Location loc4=new Location("马鞍山","2016");
    locations.add(loc1);
    locations.add(loc2);
    locations.add(loc3);
    locations.add(loc4);

    t.setLocations(locations);
    return teacherRepository.save(t);
  }

  @RequestMapping("/q1")
  public Teacher q1(String name){
    return teacherRepository.findByName(name);
  }
  
  @RequestMapping("/q2")
  public List<Teacher> q2(int age){
    return teacherRepository.withQueryFindByAge(age);
  }

}
```

- 运行之后，访问localhost:10086/save 测试保存， /q1?name=ljb   /q2?age=20。图就不截了。

##### 8.6.2 Redis

​	Redis是一个基于键值对的开源内存数据存储，也可以做数据缓存。具体可以看看 [Redis实战第一章的介绍](https://krystalics.github.io/2018/11/20/%E7%AC%AC%E4%B8%80%E7%AB%A0-%E5%88%9D%E8%AF%86Redis/)

Spring 对它的支持是通过Spring Data Redis来实现的，为我们提供了连接相关的ConnectionFactory和数据操作相关的RedisTemplate。根据不同的Java客户端，还有不同的连接器。这里就不详述。据其中一种作为例子

```java
@Bean
public RedisConnectionFactory redisConnectionFactroy(){
    return new JedisConnectionFactory();
}
```

```java
@Bean
public RedisTemplate<Object,Object> redisTemplate() throws UnknownHostException{
    RedisTemplate<Object,Object> template=new RedisTemplate<>();
    template.setConnectionFactory(redisConnectionFactory());//调用上面的方法
    return template;
}
```

还提供了 RedisTemplate 和 StringRedisTemplate两个模板来进行数据操作。其中StringRedisTemplate只针对字符型的数据操作。主要提供下面这些方法操作数据。



| 方法          | 说明       |
| ------------- | ---------- |
| opsForValue() | 操作String |
| opsForList()  | list       |
| opsForSet()   | set        |
| opsForZSet()  | zset       |
| opsForHash()  | hash       |

定义Serializer：当我们将数据存储到Redis时，我们的key和value都是通过Spring的序列化，来存到数据中的。RedisTemplate 默认使用的是JdkSerializationRedisSerializer，StringRedisTemplate默认使用StringRedisSerializer。



##### SpringBoot对Redis的支持	

​	默认配置了JedisConnectionFactory，RedisTemplate以及StringRedisTemplate，让我们可以直接使用Redis作为数据存储。在application.properties中的配置如下：

```properties
spring.redis.database=0 #数据库名称，默认为db0
spring.redis.host=localhost #服务器地址默认为这个
spring.redis.password=
spring.redis.port= #端口号默认为6379
....
```

**实战**

- 安装Redis，可以去官网也可以用Docker
- 添加依赖 spring-boot-starter-redis
- 领域类模型必须要实现Serializable 接口，作为序列化用Jackson构造时还需要一个空的构造函数

```java
package com.example.data_ch8.redis;

import java.io.Serializable;

public class Student implements Serializable {
  private static final long serialVersionId=1L;

  private String id;
  private String name;
  private int age;
  
  public Student(){}//为Jackson构造的空函数
    
  public Student(String id,String name,int age){
    this.id=id;
    this.name=name;
    this.age=age;
  }

  public String getId() {
    return id;
  }

  public void setId(String id) {
    this.id = id;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }
}
```

- 数据访问

```java
package com.example.data_ch8.redis;

import javax.annotation.Resource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Repository;

@Repository
public class StudentDao {
  @Autowired
  StringRedisTemplate stringRedisTemplate; //注入Spring Boot已经配置好的stringRedisTemplate

  @Resource(name = "stringRedisTemplate")
  private ValueOperations<String,String> valOpsStr;

  @Autowired
  RedisTemplate<Object,Object> redisTemplate;

  @Resource(name = "redisTemplate")
  private ValueOperations<Object,Object> valOps;

  public void strDemo(){
    valOpsStr.set("xx","yy");
  }

  public void save(Student student){
    valOps.set(student.getId(),student);
  }

  public String getString(){
    return valOpsStr.get("xx");
  }

  public Student getStudent(){
    return (Student)valOps.get("1");
  }
}
```

- 控制器

```java
package com.example.data_ch8.redis;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class StudentController {
  @Autowired
  StudentDao studentDao;

  @RequestMapping("/set")
  public void set(){
    Student student=new Student("1","ljb",20);
    studentDao.save(student);
    studentDao.strDemo();
  }

  @RequestMapping("/getStr")
  public String getStr(){
    return studentDao.getString();
  }

  @RequestMapping("/getStudent")
  public Student getStudent(){
    return studentDao.getStudent();
  }
}
```

- 运行之后，访问 localhost:10086/set  /getStr  /getStudent  再观察页面和 Redis数据库的变化

spring boot 为我们自动配置的RedisTemplate ，使用JskSerializationRedisSerializer，使用二级制形式存储数据，要是不满意，我们可以自己配置定义Serializer,在主类下加入以下代码。

```java
@Bean
@SuppressWarnings({"rawtypes","unchecked"})
public RedisTemplate<Object,Object> redisTemplate(RedisConnectoryFactory connection)throws UnknownException{
    RedisTemplate<Object,Object> template=new RedisTemplate<>();
    template.setConnectionFactory(connection);
    
    Jackson2JsonRedisSerializer jackson=new Jackson2JsonRedisSerializer(Object.class);
    ObjectMapper om=new ObjectMapper();
    om.setVisibility(PropertyAccessor.ALL,JsonAutoDetect.Visibility.ANY);
    om.ebableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
    
    //设置value的序列化采用 Jackson2JsonRedisSerializer，实际上这个也可以自己定义
    template.setValueSerializer(jackson); 
    //设置key的序列化采用 StringRedisSerializer
    template.setKeySerializer(new StringRedisSerializer()); 
    
    
    template.afterPropertiesSet();
    return template;
}
```





