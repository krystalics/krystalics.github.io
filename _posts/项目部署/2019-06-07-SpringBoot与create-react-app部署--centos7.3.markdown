前两天买的阿里云的服务，今天决定吧项目部署到远程服务器上，开始了踩坑之旅。

---

本来想要用时下最流行的docker，但是使用docker的方式出现各种错误，想了想还是之后有时间再慢慢学习吧。

##### 先来说说 create-react-app 的部署吧。

参考 [Create React App部署](https://www.html.cn/create-react-app/docs/deployment/)

在react项目根目录中`npm run build`或者`yarn build` ，创建一个build目录，传到服务器用于部署。

如果使用**静态服务器**，需要使用到Node环境。最简单的是安装`serve`：

```
npm install -g serve
```

然后在服务器中处于 build目录中，在3000端口服务

```
[root@*** build] serve -l 3000
```

但是我使用了`history API`路由（React Router使用 `browserHistory`），静态文件服务器将失败。例如我访问` www.rbooks.top:5000/home `的时候就直接404，包括**动态服务器**很多时候都不能够访问该路径。

采用express服务技术 的 exam.js中，它的目录结构是和 build同级，所以需要创建一个新的文件夹包括build和exam.js：

>app
>
>----build
>
>--------...build中的其他文件
>
>----exam.js

`npm install express --save` 服务器安装 express

```js
const express = require('express');
const path = require('path');
const app = express();

app.use(express.static(path.join(__dirname, 'build')));
// app.get('/', function(req, res) {  没有加*号的，服务器找不到路径 
app.get('/*', function(req, res) {
  res.sendFile(path.join(__dirname, 'build', 'index.html'));
});

app.listen(9000);
```

然后服务器在app目录中运行 `node exam.js`，但是当我们关闭该终端项目就默认被关闭了。所以需要在后台运行，这边使用`pm2`管理项目：

```js
npm install pm2 -g
```

```
pm2 start exam.js
```

这样用pm2启动项目之后，即使当前终端关闭了，node进程仍然在后台运行。

有关于pm2的一些命令请看 [PM2快速入门](https://pm2.io/doc/zh/runtime/quick-start/)



##### 后台jar项目部署： 

##### 首先上传项目.jar文件。

我只用了一个 pscp （windows中需要下载 cygwin）[在windows中把文件传到远程linux服务器](https://blog.csdn.net/onlyanyz/article/details/18663761)，例子如下：

```
pscp -r E:\rbooks\backend\target\backend-0.0.1-SNAPSHOT.jar  root@47.102.119.234:/home/user
-r 表示上传文件夹中所有文件   
E:\rbooks\frontend 是源文件夹的地址
root是用户名
@47.102.119.234是服务器ip地址
:/home/user 是服务器上的文件夹地址
之后需要输入root用户在服务器上的密码
```

将项目.jar文件上传之后，就要考率环境配置，运行项目。以我这个项目为例，后台需要

##### 安装jdk1.8

 `yum install java-1.8.0-openjdk-devel.x86_64 `

然后配置文件。`vi /etc/profile` 在最后加上这三条

```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-8.b10.el6_9.x86_64
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```

`source /etc/profile` 让配置立刻生效

`java -v` 验证成功。

- maven3.5+ 安装过程和jdk类似，就不写了。
- mysql5.7 [Centos 7安装mysql](https://www.jianshu.com/p/7cccdaa2d177)  创建rbooks数据库，将本地rbook.sql文件传到服务器并导入

mysql导入sql文件： 例子： 假如sql文件在 /home/user/demo.sql

```sql
create database demo;
use demo;
source /home/user/demo.sql;
```

- 以及mysql用户 user_rbooks  密码123456  ，并赋予权限（对rbooks数据库的以及登录连接权限）。在这期间可能遇到密码修改和用户创建与权限问题

[mysql设置密码](https://blog.csdn.net/kuluzs/article/details/51924374)

[mysql创建用户与权限](https://blog.csdn.net/DoneSpeak/article/details/55548779)

发现自己创建的用户登录不进去，于是 [MySQL中出现Access denied for user '**'@'localhost' (using password: YES)](https://blog.csdn.net/love_taylor/article/details/77198850) ，重启服务或者重新连接(我就是重新连接之后可以进去)



`java -jar backend-0.0.1-SNAPSHOT.jar` 就可以运行了。但是当关闭终端，服务就终止了，所以需要让服务进入后台运行。使用`nohup java -jar backend.jar &` 可以在后台运行服务。

但是我发现还是不能在外部访问服务，这个时候只需要在阿里云的防火墙中添加8080端口，就可以访问了。

 <img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/43.png?raw=true"> 



---

项目总算是上线了，虽然路途还远但一切都在脚下。网站正在备案，目前只能通过加端口访问 www.rbooks.top:5000/home bug很多，很多东西显示不出来，本来我都不想上传的.......











