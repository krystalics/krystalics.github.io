批量sql的时候，需要设置batch size，并且关闭二级缓存，同时使用flush来同步数据库，在使用clear来清空session缓存，这样不至于内存溢出，hibernte文档上有这个例子查了下2018年的面试题，以那些问题为蓝本进行拓展挖掘：

---

1.[关于Hibernate](https://krystalics.github.io/2019/05/09/About-Hibernate/) 

2.[SQL性能优化](<https://krystalics.github.io/2019/05/09/SQL-%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/>) 

3.[消息队列mq](https://juejin.im/post/5b9a0f75e51d450e8b1392b8)

4.[ThreadLocal就是这么简单](https://juejin.im/post/5ac2eb52518825555e5e06ee) 

5.[MySQL事务原理与分布式](https://blog.csdn.net/mindfloating/article/details/49623161)    [MySQL企业常用集群](https://www.jianshu.com/p/5d522a068fa9) 

6.[高并发高性能系统设计](https://www.itcodemonkey.com/article/9980.html) 

7.[shell入门](https://github.com/qinjx/30min_guides/blob/master/shell.md)

8.[spring boot和spring mvc的区别](https://www.zhihu.com/question/64671972) 

9.[spring原理初探](https://www.jianshu.com/p/c403609185a5)

10.[常用设计模式](https://krystalics.github.io/2018/12/09/DesignPattern/)

11.java基础，IO，多线程，集合等基础类库

