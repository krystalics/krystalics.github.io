<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Docker的安装](#docker%E7%9A%84%E5%AE%89%E8%A3%85)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

记得老师上课说过：“Docker是互联网这几年唯一一个有重大突破的技术” 。它是个轻量级的容器技术，类似于虚拟机技术(xen,kvm,vmware,virtual)。Docker是直接运行在linux系统上的，并不是像虚拟机一样模拟计算机硬件再重新建立一个系统，所以它的开销远比虚拟机要小，性能也是远远高于虚拟机。

​	Docker支持将软件编译成一个镜像(image)，在这个镜像里做好对软件的各种配置，然后发布这个镜像，使用者运行这个镜像，运行中的镜像称之为容器(container，像是 **类 和对象的关系**)，容器的启动是非常快的，一般以秒为单位。

​	目前各大主流云平台都支持Docker技术，这里的云平台一般指PaaS(Platform as a Service ，平台即服务)。它是这样一个云计算：平台提供了存储，数据库，网络，负载均衡，自动拓展等功能，用户只需要将程序交给云计算平台就可以了。可以用不同的语言开发，**各种应用之间的就是用的Docker来隔离**

​	**目前主流和非主流的软件大部分都有人将其封装成Docker镜像，我们只需要下载Docker镜像，然后运行镜像就可以快速获得已做好配置可以运行的软件。**

​	本章开始将进行  Oracle XE，安装Redis作为缓存，用MongoDB进行NoSQL数据库的演示。以及第九章需要安装ActiveMQ，RabbitMQ进行异步消息的演示。在第10张会基于Docker做Spring Boot的部署。特别指出，Docker可以运用在实际的生产环境中。



#####  Docker的安装

​	因为Docker运行原理是基于Linux的，所以只能在Docker上运行。至于Windows在win10的专业版已经有docker for windows了，它是基于Hyper-v创建虚拟机让docker运行的。

> win10 企业版，专业版和教育版，具有二级地址装换（SLAT）的64位处理器，CPU支持VM监视器模式扩展，才支持Hyper-v

​	

至于其他版本就只能安装DockerToolBox了。[Docker官方文档](https://docs.docker.com/toolbox/toolbox_install_windows/#what-you-get-and-how-it-works)

安装完之后，启动Docker QuickStart Terminal 。如果出现 windows 正在查找bash.exe，这是因为

你之前已经安装过了git，只要打开Docker QuickStart Terminal的属性，在目标处将bash.exe的路径改成你的路径就行了

<img src="https://github.com/krystalics/mypostpicture/blob/master/43.png?raw=true">

在之后你可能会发生下图中的情况

<img src="https://github.com/krystalics/mypostpicture/blob/master/44.png?raw=true">

发现使用快捷方式Docker Quickstart Terminal下载boot2docker.iso根本下不了。到官网下又特别慢，这个时候你就可以选择一手 迅雷下载 (你值得拥有)。

再之后复制文件boot2docker.iso到C:\Users\username\\.docker\machine\cache\目录下，重新打开Docker Quickstart Terminal。

<img src="https://github.com/krystalics/mypostpicture/blob/master/45.png?raw=true">

在很多文章中，就已经可以运行了。

---

参考文章：

《Spring boot实战》汪云飞 第八章 spring boot的数据访问

[微软官方文档](https://docs.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v)

