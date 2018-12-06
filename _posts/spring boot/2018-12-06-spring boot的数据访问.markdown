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

​	 <a link="2018-12-05-About Docker">查看我关于Docker的文章</a>

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

至于什么是REST，请查看我的另一篇文章：<a href="2018-12-03-About Rest">About Rest</a>

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



#### 8.5 数据缓存