<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [REST的优点](#rest%E7%9A%84%E4%BC%98%E7%82%B9)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

[以下一部分来自维基百科：](https://zh.wikipedia.org/wiki/%E8%A1%A8%E7%8E%B0%E5%B1%82%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2)

REST(Representational State Transfer，表现层状态转换)是Roy Thomas Fielding博士于2000年在他的博士论文中提出的一种互联网规范。目的是便于不同软件/程序再网络(如：互联网)中互相传递信息。匹配或者兼容这种架构风格的简称为Rest或Restful网络服务，允许客户端发出URI访问和操作网络资源的请求，而与预先定义好的无状态操作集一致化。

因此表现层状态转换提供了在互联网络的计算系统之间，彼此资源可交互使用的协作性质(interoperability)。相对于其它种类的网络服务，例如 SOAP服务则是以本身所定义的操作集，来访问网络上的资源。

##### REST的优点

- 可更高效利用缓存来提高响应速度
- 通讯本身的无状态性可以让不同的服务器的处理一系列请求中的不同请求，提高服务器的扩展性
- 浏览器即可作为客户端，简化软件需求
- 相对于其他叠加在[HTTP协议](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)之上的机制，REST的软件依赖性更小
- 不需要额外的资源发现机制
- 在软件技术演进中的长期的兼容性更好

值得注意的是REST是设计风格，不是标准。



**下面一段来自我的老师陈明俊上课讲的内容：**

**REST将互联网上能够访问的都定义为资源**，而**一种资源就有多种表示状态**(如数据库中的数据，可以被写入txt,json各种格式文件中)，**将资源的表示状态在互联网上传输就是REST定义的东西**。用URI(统一资源定位标识符),实际上用的更过是它的子集URL,定义访问方法(post,get,put,delete)对资源进行操作。实际上面讲的4种方法可以对应数据库中的CRUD

| post   | Create |
| ------ | ------ |
| get    | Read   |
| put    | Update |
| delete | Delete |



在技术上与SOAP没有什么区别，用post之后做delete操作也与业务无碍，应用程序可以正常运转。但是互联网在设计之初就是为REST风格做得，很多基础的设施和服务也是这样的，比如浏览器上的缓存。再加上REST相比于复杂的SOAP和XML-RPC相比更加简洁，所以REST基本上是现在WEB服务的主流。