<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [第五章：Spring Boot 基础](#%E7%AC%AC%E4%BA%94%E7%AB%A0spring-boot-%E5%9F%BA%E7%A1%80)
  - [5.1 Spring Boot 概述](#51-spring-boot-%E6%A6%82%E8%BF%B0)
    - [5.1.1 什么是Spring Boot](#511-%E4%BB%80%E4%B9%88%E6%98%AFspring-boot)
    - [5.1.2 Spring Boot 核心功能](#512-spring-boot-%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD)
    - [5.1.3 Spring Boot 的优点](#513-spring-boot-%E7%9A%84%E4%BC%98%E7%82%B9)
    - [5.1.4 关于本书的Spring Boot版本](#514-%E5%85%B3%E4%BA%8E%E6%9C%AC%E4%B9%A6%E7%9A%84spring-boot%E7%89%88%E6%9C%AC)
    - [5.x.x 这本书第五章的后面内容](#5xx-%E8%BF%99%E6%9C%AC%E4%B9%A6%E7%AC%AC%E4%BA%94%E7%AB%A0%E7%9A%84%E5%90%8E%E9%9D%A2%E5%86%85%E5%AE%B9)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

*《springboot 实战》 汪云飞*

### 第五章：Spring Boot 基础

---

#### 5.1 Spring Boot 概述

##### 5.1.1 什么是Spring Boot

​	随着动态语言的流行**(Ruby,Groovy,Scala,Node.js)**，java开发显得格外笨重；繁多的配置，低下的效率，复杂的部署环境以及第三方技术的集成难度大。

​	在上述环境中，Spring Boot 应运而生。使用**“习惯由于配置”**(项目中存在大量的配置，此外还内置一个习惯性配置，让你无需手动进行配置)的理念让你的项目快速运行起来。用Spring Boot 很容易创建一个独立运行(运行Jar，内嵌Servlet) ，准生产级别的基于Spring框架的项目。只需要很少的配置

##### 5.1.2 Spring Boot 核心功能

 1. **独立运行的Spring 项目**：Spring Boot 可以以jar包的形式独立运行，运行一个Spring Boot 项目只需要通过 java -jar xx.jar 来运行。

 2. **内嵌Servlet容器**：Spring Boot 可选择内嵌Tomcat，Jetty或者Undertow ，这样我们无须以war包形式部署项目。

 3. **提供 starter 简化Maven配置**：Spring 提供了一系列的 starter pom 来简化 Maven 的依赖加载，例如 spring-boot-starter-web 依赖会引进很多的包，这里就不列举了。

 4. **自动配置 Spring**：Spring Boot 会根据类路径中的**jar包，类**，为jar包里的类提供自动配置的Bean,这样会极大减少我们要使用的配置。当然，Spring Boot 只是考虑了大多数开发场景，并不是所有，在实际开发中我们需要自动配置的Bean，而Spring Boot 没有提供，可以自定义 自动配置(后面6.5节中)

 5. **准生产的应用监控**：Spring Boot 提供基于http，ssh，telnet对运行时的项目进行监控

 6. **无代码生成和xml配置**：Spring Boot 的神奇不是借助代码生成来实现的，而是通过条件注解来实现的，这是Spring4.x提供的新特性。本章将讲解 Spring Boot 实现的核心技术。Spring4.x 提倡使用Java配置和注解配置组合，而 Spring Boot 不需要任何xml配置即可实现所有Spring的配置。

##### 5.1.3 Spring Boot 的优点

**优点**：

 	1. 快速构建项目
 	2. 对主流开发框架的无配置集成
 	3. 项目可独立运行，无需外部依赖Servlet
 	4. 提供运行时的应用监控
 	5. 极大地提高了开发，部署的效率
 	6. 与云计算浑然天成

##### 5.1.4 关于本书的Spring Boot版本 

​	1.3.0 RELEASE（这是书上的版本），我个人在看这本书的时候使用的是 2.1.0RELEASE 版本，工具已经很先进了。



##### 5.x.x 这本书第五章的后面内容

​	讲的是快速构建一个 Spring Boot 项目，但是之前我已经在网上百度过了，虽然它介绍了多种方法，这里我就不写了，想看的可以看书。

​	**这里多说一个解释：在IDEA里选择 Spring Initialzr 后，勾选的 project dependencies  (如Web，redis) 实际上是在pom.xml 中自动配置了 starter pom ,就比如： spring-boot-starter-web  ，spring-boot-starter-data-redis** 









