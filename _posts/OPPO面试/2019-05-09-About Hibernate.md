**Hibernate和JPA (Java Persistence API) 的关系： **参考自 [Hibernate与Jpa的关系](Hibernate与Jpa的关系? - 潜龙勿用的回答 - 知乎
https://www.zhihu.com/question/30691648/answer/255800241) 

JPA是ORM规范而Hibernate是它具体的实现。ORM是Object-Relation-Mapping ，对象关系映射是对象持久化的核心。ORM是对JDBC的封装，解决JDBC的各种问题：

1.繁琐的代码：包括大量的SQL语句

2.数据库对象连接问题：在关系数据对象之间存在各种关系、例如：1对1,1对多，多对多等。在数据库对象更新的时候，如果是JDBC就必须很小心的处理以保证关系不出错

3.系统架构问题：JDBC属于数据访问层，但却必须要知道后台是什么数据库，有那些表以及字段等各种关系。ORM可以将数据库屏蔽，只让程序员操作Java对象。

4.性能问题：根据具体数据库操作，ORM会自动延迟向后台数据库发送sql请求，将数据库访问操作合并以减少不必要的数据库操作请求。



在使用SpringBoot开发的时候无可避免的会使用到jpa的annotation (javax.persistence.Entity 、Table...) 提供基础功能，如果想定义更细小的规则就难免会使用到Hibernate。Spring Data JPA默认实现就是Hibernate



##### [Hibernate多对多](https://www.cnblogs.com/kubixuesheng/p/5300437.html) 

我刚接触到这些表关联的时候根本不知道它有啥作用，查了几篇文章才知道。当表关系非常复杂时(几百张表)，**我们查询的时候如果不用表关联获取关联表的数据而是靠我们自己写代码获取关联数据，将是非常繁琐的过程**。比如：我在项目中写过的一段代码：

```java
public MyPage myPage(int userid) {
    //先利用 userid寻找 用户信息
    Information information = getInformation(userid);
    if (information == null) {
      return "找不到该用户";
    } 
    // 根据找到的用户信息再去获取其他表中的数据
    List<Book> bookList_write = getBooks_write(information.getUsername());
    List<Book> bookList_follow = getBooks_follow(userid);
    List<User> userList_follow = getUsers_follow(userid);
    List<User> userList_be_followed = getUsers_be_followed(userid);

    MyPage myPage = new MyPage(information, bookList_write, bookList_follow, userList_follow,
        userList_be_followed);
    
    return myPage;
  }
```

这只是几个表的查询，如果表的数量一多，查询的深度就会大大提高，要手写的代码就会非常的多。

上面的代码如果用sql语句来写 类似于这样：

```sql
select * 
from book 
where 
author = (select name from information where information.userid=1);
```







老师和学生的关系就是多对多的： 下面的代码中只保存学生的部分，当我们查看数据库的数据时发现老师的数据也已经添加到数据库中了。

```java
Set<Teachers> teachers = new HashSet<>();
            
teachers.add(new Teachers("Wang"));
teachers.add(new Teachers("Li"));
teachers.add(new Teachers("Song"));
teachers.add(new Teachers("Zhang"));

Students s = new Students();
s.setSname("zhangsan");
s.setTeachers(teachers);

session.save(s); //保存s
tx.commit(); //提交事务 
```

annotation配置如下： **以下是学生单向关联老师，即保存学生会一起保存它设置过的老师，保存老师却不会保存学生，也有双向关联的，当然单向关联的方向也可以是从老师到学生。**

```java
import javax.persistence.*;
import java.util.Set;

//学生实体类
@Entity
public class Students {
    private int sid;  //编号

    private String sname; //姓名

    private Set<Teachers> teachers ;  // 我设置学生这个多方去持有老师这个多方的集合，去控制老师

    //注意：一定要保留这个默认不带参数的构造方法
    public Students() {
        
    }

    public Students(String sname)
    {
        this.sname = sname;
    }

//    	先设置多对多的关联，之后必须生成一个中间表，使用JoinTable注解
//    	cascade 级联 这里是指无论 CRUD 都会触发这个操作，即更新时这里也更新，
    @ManyToMany(cascade=CascadeType.ALL)
    @JoinTable(
//      设置中间表名
      name="teachers_students",
//      指定当前对象的外键,本表在中间表的外键名称
      joinColumns={@JoinColumn(name="sid")},
//      指定关联对象的外键,另一个表在中间表的外键名称。
      inverseJoinColumns={@JoinColumn(name="tid")}
     ) //这个配置说明 teachers_students表中就两个字段 sid tid
    // 下面是单向关联中设置老师 的方法
    public Set<Teachers> getTeachers() {
        return teachers;
    }

    public void setTeachers(Set<Teachers> teachers) {
        this.teachers = teachers;
    }
    
    @Id
    @GeneratedValue
    public int getSid() {
        return sid;
    }

    public void setSid(int sid) {
        this.sid = sid;
    }

    public String getSname() {
        return sname;
    }

    public void setSname(String sname) {
        this.sname = sname;
    }
}
```

单向关联中，只要有一个类中设置关联关系，另一个类就不用了：

```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Teachers {
    private int tid;//教师的编号

    private String tname;//教师姓名
    
    public Teachers() {
        
    }

    public Teachers(String tname)
    {
      this.tname = tname; 
    }
    
  /*   下面的方法是 双向关联时 的做法
  	private Set<Student> stus;
  	
  	@ManyToMany(mappedBy="teachers")  把控制权交给student类——teachers集合引用
    public Set<Students> getStus(){
        return stus;
    }
    */
    @Id
    @GeneratedValue
    public int getTid() {
        return tid;
    }

    public void setTid(int tid) {
        this.tid = tid;
    }

    public String getTname() {
        return tname;
    }

    public void setTname(String tname) {
        this.tname = tname;
    }
}
```



##### Hibernate优化策略   [Hibernate优化](https://blog.csdn.net/blueheart20/article/details/21019043)

由于Hibernate自行管理从数据库中读入的数据，容易出现内存占用过大的问题。所以需要进行优化

1.尽可能不用或者少用多对多，将其拆分成一对多和多对一

2.根据程序实际需求，选择性加载表中的数据 ，除非必须，一般无需加载全部字段

3.在Hibernate中进行查询之时使用分页降低一次性的内存负载

4.在性能关键的地方可以尝试使用经过优化的原生SQL 而非HQL

5.优化fetch_size/batch_size大小，大量写入日志会产生巨大的IO操作，所以可以提高日志级别降低日志的写入量，或者通过缓存批量写入。

- 批量sql的时候，需要设置batch size，并且关闭二级缓存，同时使用flush来同步数据库，在使用clear来清空session缓存，这样不至于内存溢出，hibernte文档上有这个例子

6.延迟加载策略的使用，二级缓存优化，关联表优化 

- 一级缓存 是session中，存放私有数据，随着session产生和消亡

```java
//一开始获取缓存中没有的数据时 发出sql请求 与数据库交互
Student stu1=(Student) session.get(Student.class,1000L);
// session中刚刚缓存了，所以这行代码不发出 sql请求
Student stu2=(Student) session.get(Student.class,1000L);

session.close(); //一级缓存结束

```

- 二级缓存 是在sessionFactory中，主要存放共有数据，生命周期和Hibernate相同

```java
Session session=sessionFactory.openSession();
Classes classes=(Classes)session.get(Classes.class,1L);
//sessionFactory.getStatistics().getEntityLoadCount();
```

