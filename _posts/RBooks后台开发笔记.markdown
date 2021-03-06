#### 2019.3.5----2019.3.15

## 项目需求分析

**系统名称：**一起读书吧

**数据录入：**通过用户注册收集用户数据，书本的数据在pc端增加

- 系统功能模块

<img src="https://github.com/krystalics/RBooks/blob/master/img/%E7%B3%BB%E7%BB%9F%E5%8A%9F%E8%83%BD%E5%9B%BE.png?raw=true">

- **用户与权限管理：**

**用户角色：**一般用户，管理用户

**权限级别：**管理用户比一般用户多了几个功能：删除，修改，查阅用户信息

**数据可视性：**普通用户只能看见自己的信息，而管理用户可以看见所有用户信息。

<img src="https://github.com/krystalics/RBooks/blob/master/img/%E4%B8%9A%E5%8A%A1%E6%B5%81%E7%A8%8B%E5%9B%BE.png?raw=true">

- **用例设计图**

<img src="https://github.com/krystalics/RBooks/blob/master/img/%E7%94%A8%E4%BE%8B%E5%9B%BE.png?raw=true">



- **后台模块设计**

<img src="https://github.com/krystalics/RBooks/blob/master/img/%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84%E5%9B%BE.png?raw=true">

**以service为中心，通过调用dao读取和储存数据，反应到controller中，最后与前端完成交互**

更加抽象的层面如下图：

抽象的层面如下图：

<img src="https://github.com/krystalics/RBooks/blob/master/img/%E5%90%8E%E5%8F%B0%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84%E5%9B%BE.png?raw=true">

划分完层次结构之后的步骤：

1. 把DAO的接口设计出来，这一步基本就是将Entity中的表crud操作包装一下。
2. 然后把Service的接口设计并实现，调用DAO的Bean
3. 最后写出Controller，调用Service的Bean
4. 前端访问Controller

有时候还需要不断的和前端的设计页面对接，写出接口文档(在编写中不断的更改)，然后通过Controller需要的服务再次对之前的模块进行修正，毕竟不可能一开始就写出不需要修改的代码。只能说结构好点的代码修改的地方比较少。

---

#### 2019.3.17  周日

在写Controller模块，遇到了一个问题，那就是User表中的Id是自增的，我在犹豫是否要在参数中加入Id，查了资料感觉没什么用。于是就直接用postman试了一下，发现不加id也是可以的。会由hibernate自动生成的表 hibernate_sequence 中的值决定下一个id是多少，

<img src="https://github.com/krystalics/RBooks/blob/master/img/3.png?raw=true">

如上图中下一个id的值是6，就是这样。如果表中已经有值，id和next_val冲突就会报错，因为primary_key只能有一个值不能重复。

关于Hibernate的自增主键管理，这个项目只有两个自增主键。可是它们并不是分别自增，而是共享一个值，那个值就是之前图中的next_val，即使修改了三行next_val中的另外两行也是如此。

<img src="https://github.com/krystalics/RBooks/blob/master/img/7.png?raw=true"><img src="https://github.com/krystalics/RBooks/blob/master/img/6.png?raw=true">

不过这不是什么大问题，只是有点奇怪罢了。之后有时间再琢磨吧。

#### 还有关于Cookie和Session的问题：

我想搞清楚这个的原理还是很重要的。下面一段参考自[Session原理](https://www.jianshu.com/p/2b7c10291aad)

##### 无状态的 HTTP 协议

还记得每当入门一门 Web 端语言的进行服务器端开发的时候，仅次于「Hello World」的 demo 就是「登录功能」了。实现登录功能很简单，验证客户端发送过来的账户和密码，如果通过验证就把用户塞进 session 中，然后在后续的访问中，只需检测 session 是否有这个用户就能知道用户是否登录了。Session 的中文翻译为：「会话」，只属于某一个客户端和某一个服务器端沟通的工具。但，计算机网络老师又说了，HTTP 协议是无状态的，怎么能记录用户的登录状态呢？
 鉴于 HTTP 是无状态协议，之前已认证成功的用户状态是无法通过协议层面保存下来的，既，无法实现状态管理，因此即使当该用户下一次继续访问，也无法区分他和其他的用户。于是我们会使用 Cookie 来管理 Session，以弥补 HTTP 协议中不存在的状态管理功能。

##### 利用 Cookie 管理 Session

<img src="https://github.com/krystalics/RBooks/blob/master/img/4.png?raw=true">

而且关于Cookie和Session ，SpringBoot会有专门的模块处理的，后面具体的操作就不赘述了。



---

#### 2019-3.18

本来打算前两天就结束这个后台的0.2版本，可是开发过程并没有想象中的那么顺利。今天又是遇到了新的东西，就是接口文档，接口文档实际上并不好写，要把之前存在于想象中的数据格式具体化到json中，实际上是要花一点时间的，而且还要各种微调。以及把数据库中的一个多值依赖的表分成了两个，这样前面有工作没做完，又有新的东西加进来感觉整个人都有点混乱，所以在犹豫是否要熬个夜，把工作赶掉，明天去华科参加京东的实习招聘——去的话，回来就要打的。

开始正经的记笔记啦！

用postman测试接口的时候发现关于**删除数据的操作需要加`@Transactional`注解**，不然会报错。

**2019.3.20** 两天之后我查了下文档：**默认情况下JPA的每个操作都是事务的，在默认情况下，JPA的事务会设置为只读**，具体可以参考SimpleJpaRepository。**只有声明了@Transactional，本质上是声明了@Transactional(readOnly=false)，这样覆盖了默认的@Transactional配置便可以执行修改操作了**。提供事务支持，如隔离性等事务特性。下面是SimpleJpaRepository的开头部分源码，整个类都是标注了  readOnly=true的，

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
	....
}
```



写到ReadController时，想到需要有人写书，不过在手机上不好写，想着还是给出网页版的链接地址，到电脑上写吧。

总结：今天的工作完成了大约一半左右，难受，明天继续努力。



---

#### 2019.3.19

好了明天到了(:-) ，还是先记录一下昨天写SpringBoot的感受吧。真的是太强大了，关于CrudRepository，真的不用自己写逻辑就可以自定义按照某个字段查找返回的结果也不限定于单一的记录，也可以是一个List。完全靠自己定义，如下：

```java
public interface BookRepository extends CrudRepository<Book,Integer> {
  Book findById(int id);
  List<Book> findByAuthor(String author); //找到对应作者的所有书
  Boolean exsitsByAuthor(String author); //查找该作者是否在表中
}

```

还有就是Navicat真实比MySQL-WorkBench简单太多了，postman也很好用。生活在互联网工具高度发达的时代真爽。



今天遇到的问题：可能是改版后的**表中有like字段和关键字 like 冲突了**，所以insert时出错了，还是要提醒自己，命名要谨慎。



下午：遇到了**很麻烦的问题**，找了资料上面的解决方式都没用，所以后面我换了参数才解决。

Could not find acceptable representation  讲的是前端传来的数据与后端Content-Type的要求不一致，可是在项目中两者是一致的呀。还有说是命名重复导致，我还没涉及到js文件呢。

<img src="https://github.com/krystalics/RBooks/blob/master/img/8.png?raw=true">

<img src="https://github.com/krystalics/RBooks/blob/master/img/9.png?raw=true">



我先将其变为 get ，看看返回一个pojo是否可以。发现还是不行：`No converter found for return value of type` ，然后查了一下，pojo要先序列化才能用于传输，所以面向CSDN编程了下：查到如下资料

仅仅调用**ObjectMapper**  来实现，我们现在举个例子，可以创建一个POJO对象来与JSON相对应，jackson默认只能序列化 public 的字段，所以下面的例子中 name和 id都是public的。

```java

@Autowired
private ObjectMapper mapper;	
 
@GetMapping("/serialization.json")
public @ResponseBody String dataBind() throws IOException{
	User user = new User();
	user.setName("scg");
	user.setId((long) 18);
	String jsonStr = mapper.writeValueAsString(user);
	return jsonStr;
}

```

将参数改成`Map<String,String> ` 就可以运行了，这个问题后续还要继续Google。

```java
@RequestMapping(value = "/mypage", method = RequestMethod.POST, produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
public String getInformation(@RequestBody Map<String,String> userid) {
  // 参数是 Map<String,Integer> 好像不行，因为会报这个错误 Could not find acceptable representation，然后参考了
  return myPageServiceImpl.myPage(Integer.parseInt(userid.get("userid")));
}
```

后面才发现，如果需要整数 需要配套的是` MediaType.TEXT_PLAIN_VALUE` ,如下：

```java
@RequestMapping(value = "/deletebook", method = RequestMethod.POST, produces = MediaType.TEXT_PLAIN_VALUE)
public String deleteBook(@RequestBody Map<String,Integer> id_map) {
  int success = writeServiceImpl.deleteBook(id_map.get("id")); 
  if (success == 1) {
    return "删除成功";
  }
  return "没有这本书";
}
```



**总结一下上面的问题：关于content-type是json还是text/plain 的问题：据我现在的理解和对接口的测试，发现如下**： 

```json
{
    "username":"lin",
    "password":"32fjkl" //像这样格式的数据 json和 text/plain都是一样可行的
}
---
{
    "id":2  //有包含整数的，只能是 text/plain
}

---
 有明确json格式的数据  就必须是  json的
{
    "chapterid""{
    	"bookid":2,
    	"chaptername":"章节名称"
	},
	"content":"章节内容"
}


```

下面是一篇关于content-type的文章：[content-type](http://homeway.me/2015/07/19/understand-http-about-content-type/)

**Content-Type**是实体头域（或称为实体头部，entity header）用于向接收方指示实体（entity body）的介质类型的，或称为资源的MIME类型，现在通常称media type更为合适。

 这里简单介绍一下我认为重要的内容：**关于post的发包方式**——

1. `application/x-www-form-urlencoded` 是常用的表单发包方式，普通表单提交、或者js发包默认都是通过这种方式，使用`enctype`来设置

```html
<form enctype="application/x-www-form-urlencoded" action="目标网址" method="POST">
    <input type="text" name="name" value="homeway">
    <input type="text" name="key" value="nokey">
    <input type="submit" value="submit">
</form>
```

服务器收到的 raw header类似：

```html
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding:gzip, deflate
Accept-Language:zh-CN,zh;q=0.8,en;q=0.6,zh-TW;q=0.4,gl;q=0.2,de;q=0.2
Cache-Control:no-cache
Connection:keep-alive
Content-Length:17
Content-Type:application/x-www-form-urlencoded
```

row body : `name=homeway&key=nokey`  就是上面的数据。

2. `multipart/form-data` :用于发送文件的Post包

假设用python发送一个文件给服务器：

```python
data = {
    "key1": "123",
    "key2": "456",
}
files = {'file': open('index.py', 'rb')}
res = requests.post(url="http://localhost/upload", method="POST", data=data, files=files)
print res
```



服务器收到的数据内容如下：

```
POST http://www.homeway.me HTTP/1.1
Content-Type:multipart/form-data; boundary=------WebKitFormBoundaryOGkWPJsSaJCPWjZP

------WebKitFormBoundaryOGkWPJsSaJCPWjZP
Content-Disposition: form-data; name="key2"
456
------WebKitFormBoundaryOGkWPJsSaJCPWjZP
Content-Disposition: form-data; name="key1"
123
------WebKitFormBoundaryOGkWPJsSaJCPWjZP
Content-Disposition: form-data; name="file"; filename="index.py"
```

当文件太长，HTTP无法在一个包之内发送完毕，就需要分割数据，分割成一个一个chunk发送给服务端，

那么`--`用于区分数据快，而后面的数据`633e61ebf351484f9124d63ce76d8469`就是标示区分包作用

---

