# Dubbo

### 学习目标

- 了解与软件性能有关的各种指标，例如: 吞吐量，并发量，平均/最大/最小响应时间
- 能够区分什么是集群，什么是分布式
- 了解软件架构的分类，以及演进
- 能够仿写并理解 dubbo入门案例代码
- 能够配置超时重试，版本控制，以及知道负载均衡的种类，以及代码实现。



中文官网：http://dubbo.apache.org/zh-cn/docs/user/quick-start.html



### 1. 简介

- Dubbo是阿里巴巴公司开源的一个高性能优秀的[服务框架](https://baike.baidu.com/item/服务框架)，使得应用可通过高性能的 RPC 实现服务的输出和输入功能，可以和 Spring框架无缝集成。Dubbo是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：**面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现**。
- RPC协议：基于tcp进行数据的传输，数据报文封装和解析有自己一套方式网络协议，都可以成为RPC协议，类似HTTP协议，都是tcp的上层协议。
- Dubbo是一款框架（需导入jar包），并提供了后台管理系统对Dubbo的使用进行监控和管理。
- Dubbo 框架在使用时，存在两种角色，一个提供服务（服务提供者： 
  **Provider**），一个消费服务（服务消费者：**Consumer**）



### 2. 互联网架构相关概念

#### 2.1 特点

- 用户多 
- 流量大，并发高
- 海量数据 
- 易受攻击
- 功能繁琐 
- 变更快

#### 2.2 性能

- 让系统具备更高的**性能**， 让用户感受到快速的访问效果。
- 软件**性能指标**：
  - **响应时间**：指执行一个请求从开始到最后收到响应数据所花费的总体时间。 
  
  -  **并发数**：指系统同时能处理的请求数量。
  
    ​         1）**并发连接数**：指的是客户端向服务器发起请求，并建立了TCP连接。每秒钟服务器连接的总TCP数量 
  
    ​         2）**请求数**：也称为QPS(Query Per Second) 指每秒多少请求.
  
    ​         3）**并发用户数**：单位时间内有多少用户  
  
  - **吞吐量**：指单位时间内系统能处理的请求数量。 
  
    ​         1） QPS：Query Per Second 每秒查询数。
  
    ​         2）TPS：Transactions Per Second 每秒事务数。
  
    ​         3） 一个事务是指一个客户机向服务器发送请求然后服务器做出反应的过程。客户机在发送请求时开始计时，收到服务器响应后结束 计时，以此来计算使用的时间和完成的事务个数。 
  
    ​         4） 一个页面的一次访问，只会形成一个TPS；但一次页面请求，可能产生多次对服务器的请求，就会有多个QPS。
  
    

#### 2.3 目标

- **高性能**：提供快速的访问体验。
- **高可用**：网站服务一直可以正常访问。
- **可伸缩**：通过硬件增加/减少，提高/降低处理能力。
- **高可扩展**：系统间耦合低，方便的通过新增/移除方式，增加/减 少新的功能/模块。 
- **安全性**：提供网站安全访问和数据加密，安全存储等策略。
- **敏捷性**：随需应变，快速响应。



#### 2.4 集群和分布式

##### 软件架构

- 软件架构分为：**单体架构 和 分布式架构**
- **单体架构**: 整个系统所有需求代码都写在一个java项目中
- **分布式架构**：整个系统，按照需求划分为多个java项目，系统要想正常运行，需要将所有java项目全都启动。
- 值得一说的是，微服务架构也是分布式架构的一种，它是在分布式架构设计思想之上，在进行更细致的划分，了解即可，后面会讲微服务！



##### 集群&分布式-相同点

- 都是多台机器为软件提供部署
- 都是为了提高软件整体的负载能力，毕竟多个人干活比一个人效率更优

##### 集群&分布式-不同点

- 集群： 一个业务或者项目，部署在多个服务器上 , 每个服务器部署代码都一样

- 分布式： 一个业务或者项目**拆分**多个子业务或者子项目，部署在不同的服务器上，每个服务器部署代码不一样

  

<font color=red>**简单讲**：都是多台机器部署java项目，集群部署的代码都是一样的，分布式下多台机器部署的代码都是不一样的。</font>



#### 2.5 架构的演进

##### 单体架构

- 单体架构也称之为 “集中式架构”，也是最早的软件架构，只需一个应用，将应用系统所有功能模块都集中编写到一个java项目中，统一进行部署，当网站流量很小时，以减少部署节点和成本。 

- **优点**： • 简单：开发部署都很方便，小型项目首选，例如：使用量不高的后台管理系统
- **缺点**：
  - 代码耦合，开发维护困难 
  - 无法针对不同模块进行针对性优化 
  - 水平扩展性差 
  - 单点容错率低，并发能力差



##### 垂直架构

- 当访问量逐渐增大，单一应用无法满足需求，此时为了应对更高的并发和业务需求，我们根据业务功能对系统进行拆分，
- **例如**：电商系统，将商品搜索业务  和 秒杀业务 和 直播带货业务 从一个项目中拆分出去

- 优点：
  - 系统拆分实现了流量分担，解决了并发问题 
  - 无法针对不同模块进行针对性优化
  - 水平扩展性差 
  - 并发能力差
- 缺点：
  -  服务之间相互调用，如果某个服务的端口或者ip地址发生改变，调用的系统得手动改变 
  -  搭建集群之后，实现负载均衡比较复杂 



##### 分布式架构

-  当垂直应用越来越多，应用之间交互不可避免，此时将业务模块进行更细致的划分，常见分为: **基础服务** 和  **核心业务服务**  两类，**规定只能核心业务服务 调用 基础服务**，更细致的服务划分和规定，让服务管理更方便，置为划分更清晰，基础服务模块的复用性也大大提高。

- **优点**：
  - 将基础服务进行了抽取，系统间相互调用，提高了代码复用和开发效率
- **缺点**：
  - 系统间耦合度变高，调用关系错综复杂，难以维护
  - 搭建集群之后，负载均衡比较难实现

##### 服务治理（SOA）

-  当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键

- 阿里巴巴内部目前使用的框架：HSF（好舒服），是dubbo的升级版

- 以前出现了什么问题？

  - 服务越来越多，需要管理每个服务的地址
  - 调用关系错综复杂，难以理清依赖关系
  - 服务过多，服务状态难以管理，无法根据服务情况动态管理

- 服务治理要做什么？

  - 服务注册中心，实现服务自动注册和发现，无需人为记录服务地址
  - 服务自动订阅，服务列表自动推送，服务调用透明化，无需关心依赖关系
  - 动态监控服务状态监控报告，人为控制服务状态

- 缺点：

  - 服务间会有依赖关系，一旦某个环节出错会影响较大
  - 服务关系复杂，运维、测试部署困难，不符合DevOps思想

  

##### 微服务

-  前面说的SOA，英文翻译过来是面向服务的编程。微服务，似乎也是服务，都是对系统进行拆分。因此两者非常容易混淆，但其实却有一些差别 

- 微服务的特点：

  - 单一职责：微服务中每一个服务都对应唯一的业务能力，做到单一职责

  - 微：微服务的服务拆分粒度很小，例如一个用户管理就可以作为一个服务。每个服务虽小，但“五脏俱全”。

  - 面向服务：面向服务是说每个服务都要对外暴露服务接口API。并不关心服务的技术实现，做到与平台和语言无关，也不限定用什么技术实现，只要提供Rest的接口即可。

  - 自治：自治是说服务间互相独立，互不干扰

    ​      1） 团队独立：每个服务都是一个独立的开发团队，人数不能过多。 

    ​      2）  技术独立：因为是面向服务，提供Rest接口，使用什么技术没有别人干涉 

    ​      3）  前后端分离：采用前后端分离开发，提供统一Rest接口，后端不用再为PC、移动端开发不同接口 

    ​      4）  数据库分离：每个服务都使用自己的数据源 

    ​      5）部署独立，服务间虽然有调用，但要做到服务重启不影响其它服务。有利于持续集成和持续交付。每个服务都是独立的组件，可复用，可替换，降低耦合，易维护 Docker部署服务

- **优点**：
  - 每个微服务都很小，足够内聚，足够小，代码容易理解。团队能够更关注自己的工作成果, 聚焦指定的业务功能或业务需求。
  - 开发简单、开发效率提高，一个服务可能就是专一的只干一件事, 能够被小团队单独开发，这个小团队可以是 2 到 5 人的开发人员组成。

- **缺点**：
  - 微服务架构带来过多的运维操作, 可能需要团队具备一定的 DevOps 技巧.
  - 分布式系统可能复杂难以管理。因为分布部署跟踪问题难。当服务数量增加，管理复杂性增加。





### 3. Dubbo快速入门

#### 3.1 概述

-  Dubbo是阿里巴巴公司开源的一个高性能、轻量级的Java RPC 框架。 
-  致力于提供高性能和透明化的RPC 远程服务调用方案，以及SOA 服务治理方案。
-  官网：http://dubbo.apache.org
-  dubbo 可以和 spring 无缝结合，在使用dubbo时，我们可以借助spring配置文件对dubbo做一些配置

##### Dubbo架构

-  Provider：暴露服务的服务提供方 
- Container：服务运行容器
- Consumer：调用远程服务的服务消费方 
- Registry：服务注册与发现的注册中心 
- Monitor：统计服务的调用次数和调用时间的监控中心



#### 3.2 zookeeper安装

- Dubbo官方推荐使用Zookeeper作为注册中心


##### Windows

```java
//下载zookeeper  win 64位压缩包
//解压缩就能用
//F:\worksoft\apache-zookeeper-3.5.6-bin\bin  双击：zkServer.cmd
```

##### Centos7

```shell
#root登录后, 进入到安装目录
[root@192 /]$ cd itcast/install/

#jdk环境 web核心已提供安装教程, zookeeper 需要java环境, 另外zookeeper 默认监听2181端口
#上传zookeeper linux版本压缩包
[root@192 install]$ rz

#解压缩 apache-z#root登录后, 进入到安装目录
[root@192 /]$ cd itcast/install/

#jdk环境 web核心已提供安装教程, zookeeper 需要java环境, 另外zookeeper 默认监听2181端口
#上传zookeeper linux版本压缩包
[root@192 install]$ rz

#解压缩 apache-zookeeper-3.5.6-bin.tar.gz
[root@192 install]$ tar -zxvf apache-zookeeper-3.5.6-bin.tar.gz

#进入zookeeper核心配置目录
[root@192 install]$ cd apache-zookeeper-3.5.6-bin/conf/

#拷贝 zk 核心配置文件
[root@192 conf]$ cp zoo_sample.cfg zoo.cfg

#创建zooKeeper存储目录
[root@192 conf]$ mkdir ../zkdata

#修改zoo.cfg  dataDir=/itcast/install/apache-zookeeper-3.5.6-bin/zkdata
[root@192 conf]$ vim zoo.cfg 

#启动ZooKeeper
[root@192 conf]$ cd ../bin/
[root@192 bin]$ ./zkServer.sh start

#查看ZooKeeper状态, 如果存在：not running 启动失败
[root@192 bin]$ ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /itcast/install/apache-zookeeper-3.5.6-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: standalone

#查看进程获取端口监听都可以监测zk 启动是否成功
[root@192 bin]$ netstat -ant|grep 2181

[root@192 bin]$ ps -ef|grep zookeeper

#停止  restart 重启
[root@192 bin]$ ./zkServer.sh stop
ookeeper-3.5.6-bin.tar.gz
[root@192 install]$ tar -zxvf apache-zookeeper-3.5.6-bin.tar.gz

#进入zookeeper核心配置目录
[root@192 install]$ cd apache-zookeeper-3.5.6-bin/conf/

#拷贝 zk 核心配置文件
[root@192 conf]$ cp zoo_sample.cfg zoo.cfg

#创建zooKeeper存储目录
[root@192 conf]$ mkdir ../zkdata

#修改zoo.cfg  dataDir=/itcast/install/apache-zookeeper-3.5.6-bin/zkdata
[root@192 conf]$ vim zoo.cfg 

#启动ZooKeeper
[root@192 conf]$ cd ../bin/
[root@192 bin]$ ./zkServer.sh start

#查看ZooKeeper状态, 如果存在：not running 启动失败
[root@192 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /itcast/install/apache-zookeeper-3.5.6-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: standalone


#查看进程获取端口监听都可以监测zk 启动是否成功
[root@192 bin]$ netstat -ant|grep 2181

[root@192 bin]$ ps -ef|grep zookeeper

```



#### 3.3 示例代码（重要）

**实现步骤：**

   ① 创建服务提供者Provider模块

   ② 创建服务消费者Consumer模块 

   ③ 在服务提供者模块编写 UserServiceImpl 提供服务

   ④ 在服务消费者中的 UserController 远程调用 UserServiceImpl 提供的服务 

   ⑤ 分别启动两个服务，测试



##### dubbo-interface 模块

- 负责定义项目所有Service接口，供其他模块导入使用

###### UserService.java

```java
package com.itheima.service;

public interface UserService {

    public String sayHello();
    
}
```



##### dubbo-service 模块

- dubbo 服务提供者，启动时立即将自身所在电脑ip、端口、UserServiceImpl 等信息注册到zookeeper注册中心

- @Service 为 dubbo提供非 spring

###### UserServiceImpl.java

```java
package com.itheima.service.impl;

import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Service;


//@Service//将该类的对象创建出来，放到Spring的IOC容器中  bean定义

@Service//将这个类提供的方法（服务）对外发布。将访问的地址 ip，端口，路径注册到注册中心中
public class UserServiceImpl implements UserService {

    public String sayHello() {
        return "hello dubbo hello!~";
    }
}
```

###### applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xmlns:dubbo="http://dubbo.apache.org/schema/dubbo" xmlns:context="http://www.springframework.org/schema/context"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">


	<!--<context:component-scan base-package="com.itheima.service" />-->

	<!--dubbo的配置-->
	<!--1.配置项目的名称,唯一-->
	<dubbo:application name="dubbo-service"/>
	<!--2.配置注册中心的地址-->
	<dubbo:registry address="zookeeper://127.0.0.1:2181"/>
<!--	<dubbo:registry address="zookeeper://192.168.149.135:2181"/>-->
	<!--3.配置dubbo包扫描-->
	<dubbo:annotation package="com.itheima.service.impl" />

</beans>
```

###### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         version="2.5">

	<!-- spring -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:spring/applicationContext*.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

</web-app>
```

###### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>dubbo-service</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <spring.version>5.1.9.RELEASE</spring.version>
        <dubbo.version>2.7.4.1</dubbo.version>
        <zookeeper.version>4.0.0</zookeeper.version>

    </properties>

    <dependencies>
        <!-- servlet3.0规范的坐标 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
        <!--spring的坐标-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--springmvc的坐标-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!--日志-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.21</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.21</version>
        </dependency>

        <!--Dubbo的起步依赖，版本2.7之后统一为rg.apache.dubb -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>${dubbo.version}</version>
        </dependency>
        <!--ZooKeeper客户端实现 -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>${zookeeper.version}</version>
        </dependency>
        <!--ZooKeeper客户端实现 -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>${zookeeper.version}</version>
        </dependency>


        <!--依赖公共的接口模块-->
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>dubbo-interface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <!--tomcat插件-->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.1</version>
                <configuration>
                    <port>9000</port>
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

##### dubbo-web 模块

- dubbo服务器消费者，只有在用到dubbo @Reference时，才会加载zookeeper相关信息，默认启动不加载

###### UserController.java

```java
package com.itheima.controller;

import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/user")
public class UserController {

    //注入Service
    //@Autowired//本地注入

    /*
        1. 从zookeeper注册中心获取userService的访问url
        2. 进行远程调用RPC
        3. 将结果封装为一个代理对象。给变量赋值
     */

    @Reference//远程注入
    private UserService userService;

    @RequestMapping("/sayHello")
    public String sayHello(){
        return userService.sayHello();
    }

}
```

###### springmvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
         http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">


    <mvc:annotation-driven/>
    <context:component-scan base-package="com.itheima.controller"/>

    <!--dubbo的配置-->
    <!--1.配置项目的名称,唯一-->
    <dubbo:application name="dubbo-web" >
        <dubbo:parameter key="qos.port" value="33333"/>
    </dubbo:application>
    <!--2.配置注册中心的地址-->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
    <!--3.配置dubbo包扫描-->
    <dubbo:annotation package="com.itheima.controller" />

</beans>
```

###### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         version="2.5">
		 
	<!-- Springmvc -->	 
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 指定加载的配置文件 ，通过参数contextConfigLocation加载-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/springmvc.xml</param-value>
        </init-param>
    </servlet>

    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

###### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>dubbo-web</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <spring.version>5.1.9.RELEASE</spring.version>
        <dubbo.version>2.7.4.1</dubbo.version>
        <zookeeper.version>4.0.0</zookeeper.version>

    </properties>

    <dependencies>
        <!-- servlet3.0规范的坐标 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
        <!--spring的坐标-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--springmvc的坐标-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!--日志-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.21</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.21</version>
        </dependency>

        <!--Dubbo的起步依赖，版本2.7之后统一为rg.apache.dubb -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>${dubbo.version}</version>
        </dependency>
        <!--ZooKeeper客户端实现 -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>${zookeeper.version}</version>
        </dependency>
        <!--ZooKeeper客户端实现 -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>${zookeeper.version}</version>
        </dependency>

        <!--依赖公共的接口模块-->
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>dubbo-interface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!--依赖service模块-->
       <!-- <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>dubbo-service</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>-->

    </dependencies>

    <build>
        <plugins>
            <!--tomcat插件-->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.1</version>
                <configuration>
                    <port>8000</port>
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>


</project>
```



### 4. 高级特性（重要）

**注意**：该篇会介绍很多 @Service 和 @Reference 配置

​		**1）**@Service用于Provider做配置，@Reference 用于Consumer做配置

​		**2）**@Service 和 @Reference 很多配置存在重复性功能

​		**3）**dubbo官方推荐尽可能多去用provider做配置，因为Consumer 端 @Reference 会默认使用 @Service中的配置

​		**4）**Consumer之所以能读取到Provider配置信息，是因为Provider 启动时，会将配置信息注册到 zookeeper中,  Consumer 在发起调用时，会去zookeeper中获取配置信息



#### 4.1 dubbo-admin

-  dubbo-admin 管理平台，图形化的页面 方便我们对服务进行管理
- 管理的服务有： 提供者/ 消费者、路由规则、动态配置、服务降级、访问控制、权重调整、 负载均衡等管理功能 
- dobbo-admin 需要我们下载安装后通过浏览器打开进行使用，由java语言编写，运行需要java环境

##### 安装

- dubbo-tomcat-8.5.zip 解压缩就能用
- dubbo-admin源码已放到tomcat 8.5 webapps/ROOT 下
- 运行：dubbo-tomcat-8.5/bin/startup.bat 即可
- 访问：http://localhost:8888/ ， 默认：账号/密码   root/root

##### 注意

<font color=red>dubbo 默认去zookeeper  server 中 读取数据并做可视化页面展示，所以dubbo-admin 需要配置 zookeeper 的 地址，修改：dubbo-tomcat-8.5\webapps\ROOT\WEB-INF\dubbo.properties 中 zookeeper ip 即可，协议和默认端口没有必要变更。</font>



#### 4.2 序列化

- 生产者 和 消费者 之间网络传递的对象， 要想正常序列化和反序列化，需注意如下事项：
  - javabean 要实现 Serializable 接口，其他对象不考虑
  - javabean在 生产者 和 消费者两个项目中，**包名.类名** 要完全一致， 一般会定义一个公共的pojo模块，让生产者和消费者都依赖 该模块。
  - **注意**：序列化 和 反序列化是 dubbo底层做的事，我们做好自己该做的，剩下的就不用操心了。
-  dubbo采用默认的hessian序列方式，可以兼容消费端与服务端使用的bean属性个数的不同 

##### dubbo-pojo 模块

###### User.java

```java
package com.itheima.pojo;

import java.io.Serializable;

/**
 * 注意！！！
 *  将来所有的pojo类都需要实现Serializable接口
 */
public class User implements Serializable {
    private int id;
    private String username;
    private String password;

    public User() {
    }

    public User(int id, String username, String password) {
        this.id = id;
        this.username = username;
        this.password = password;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

```

##### dubbo-service 模块

- 生产者模块
- 如下：已删减无参考价值的代码

###### UserServiceImpl.java

- 该模块负责将 User 序列化成 流 相应给 消费者

```java
package com.itheima.service.impl;

import com.itheima.pojo.User;
import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Service;

//@Service//将该类的对象创建出来，放到Spring的IOC容器中  bean定义
//将这个类提供的方法（服务）对外发布。将访问的地址 ip，端口，路径注册到注册中心中
@Service
public class UserServiceImpl implements UserService {

    public User findUserById(int id) {
        System.out.println("3...");
        //查询User对象
        User user = new User(1,"zhangsan","123");
        return user;
    }
}

```

##### dubbo-web 模块

- 消费者模块
- 如下：已删减无参考价值的代码

###### UserController.java

- 消费者 使用同样 User对象 接收 dubbo 响应流数据，从流数据到User 反序列化过程，由dubbo底层完成

```java
package com.itheima.controller;

import com.itheima.pojo.User;
import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/user")
public class UserController {

    @Reference//远程注入
    private UserService userService;

    /**
     * 根据id查询用户信息
     * @param id
     * @return
     */
    @RequestMapping("/find")
    public User find(int id)
        return userService.findUserById(id);
    }

}
```



#### 4.3 地址缓存

- 注册中心挂了，服务是否可以正常访问？
  -  可以，因为dubbo服务消费者在第一次调用时， 会将服务提供方地址缓存到本地，以后在调用则 不会访问注册中心。 
  -  当服务提供者地址发生变化时，注册中心会通知 服务消费者。

- **演示**：启动zk, dubbo-service, dubbo-web

    1）先访问dubbo-web 中的controller，只有第一次访问时，才会缓存

    2）停掉zk, 刷新页面，看看是否能够正常获取dubbo-service 返回的数据，可以证明已缓存

- **注意:**  zk 停机了，是不正常的，所以虽然能用，但是要确保zk server 是不能够出现问题的



#### 4.4 超时与重试

- **超时** 和 **重试** 在服务提供者 和 消费者  都可以配置，消费者配置优先级要高于服务提供者
- 服务提供者：@Service(timeout = 3000,retries = 2)，优先级低
- 服务消费者：@Reference(timeout = 3000, retries=2 )，优先级高
- 消费者请求提供者时，当发生了超时或者异常，重试会被触发

**超时**

- 服务消费者在调用服务提供者的时候发生了阻塞、等待的情形，这个时候，服务消费者会一直等待下去。
- 在某个峰值时刻，大量的请求都在同时请求服务消费者，会造成线程的大量堆积，势必会造成雪崩。
-  dubbo 利用超时机制来解决这个问题，设置一个超时时间，在这个时间段内，无法完成服务访问，则自动断开连接。
- 使用timeout属性配置超时时间，默认值1000，单位毫秒。

**重试**

- 如果出现网络抖动，则这一次请求就会失败。
- Dubbo 提供重试机制来避免类似问题的发生。
- 通过 retries 属性来设置重试次数。默认为2 次。

**示例代码**

##### dubbo-service

- @Service(timeout = 3000,retries = 2)//当前服务3秒超时,重试2次，一共3次

```java
package com.itheima.service.impl;

import com.itheima.pojo.User;
import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Service;


//@Service//将该类的对象创建出来，放到Spring的IOC容器中  bean定义
//将这个类提供的方法（服务）对外发布。将访问的地址 ip，端口，路径注册到注册中心中
@Service(timeout = 3000,retries = 2)//当前服务3秒超时,重试2次，一共3次
public class UserServiceImpl implements UserService {

    int i = 1;
  
    public User findUserById(int id) {
        System.out.println("服务被调用了："+i++);
        //查询User对象
        User user = new User(1,"zhangsan","123");
        //数据库查询很慢，查了5秒

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return user;
    }
}
```



##### dubbo-web

- @Reference(timeout = 3000, retries=2 ) //当前服务3秒超时,重试2次，一共3次

```java
package com.itheima.controller;

import com.itheima.pojo.User;
import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/user")
public class UserController {

    /*
        远程注入
        1. timeout 超时
        2. retries 重试
     */
    @Reference(timeout = 3000, retries=2)
    private UserService userService;
    
    /**
     * 根据id查询用户信息
     * @param id
     * @return
     */
    @RequestMapping("/find")
    public User find(int id){
        return userService.findUserById(id);
    }

}
```



#### 4.5 多版本

- 灰度发布：当出现新功能时，会让一部 分用户先使用新功能，用户反馈没问题 时，再将所有用户迁移到新功能。 
- dubbo 中使用version属性来设置和调 用同一个接口的不同版本



##### 示例代码

###### 服务提供者

UserServiceImplV1.java

```java
package com.itheima.service.impl;

import com.itheima.pojo.User;
import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Service;


//@Service//将该类的对象创建出来，放到Spring的IOC容器中  bean定义
//将这个类提供的方法（服务）对外发布。将访问的地址 ip，端口，路径注册到注册中心中
@Service(version = "v1.0")
public class UserServiceImplV1 implements UserService {

    public User findUserById(int id) {
        System.out.println("old...");
        //查询User对象
        User user = new User(1,"zhangsan","123");

        return user;
    }
}
```

UserServiceImplV2.java

```java
package com.itheima.service.impl;

import com.itheima.pojo.User;
import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Service;


//@Service//将该类的对象创建出来，放到Spring的IOC容器中  bean定义
//将这个类提供的方法（服务）对外发布。将访问的地址 ip，端口，路径注册到注册中心中
@Service(version = "v2.0")
public class UserServiceImplV2 implements UserService {

    public User findUserById(int id) {
        System.out.println("old...");
        //查询User对象
        User user = new User(1,"zhangsan","123");

        return user;
    }
}
```

###### 服务消费者

UserController.java

```java
package com.itheima.controller;

import com.itheima.pojo.User;
import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/user")
public class UserController {

    @Reference(version = "v2.0")//远程注入v2.0 版本代理类
    private UserService userService;

    /**
     * 根据id查询用户信息
     * @param id
     * @return
     */
    @RequestMapping("/find")
    public User find(int id){
        return userService.findUserById(id);
    }

}
```



##### 注意事项

  <font color=red>如果Provider 存在v1.0  v2.0 两个版本， 而 Consumer 端 @Reference 没有指定版本或者指定v3.0 版本，则Consumer在发起调用时，发生异常。</font>



#### 4.6 负载均衡

- 负载均衡前提：将dubbo-service 部署多次后，为集群状态，同一台电脑启动，注意避免端口冲突。
- 当dubbo-service处于集群状态时，消费者dubbo-web 就存在选址策略了，策略一共分为4种。
- 只有一致性hash负载策略，不受权重影响，其他三种权重都有影响，权重值越高，概率或者参与度越高



##### **Random** 

- • **Random** ：**权重随机数, 权重高, 随机概率大**

- **官方解释：**

  -  加权随机算法的具体实现，它的算法思想很简单。假设我们有一组服务器 servers = [A, B, C]，他们对应的权重为 weights = [5, 3, 2]，权重总和为10。现在把这些权重值平铺在一维坐标值上，[0, 5) 区间属于服务器 A，[5, 8) 区间属于服务器 B，[8, 10) 区间属于服务器 C。接下来通过随机数生成器生成一个范围在 [0, 10) 之间的随机数，然后计算这个随机数会落到哪个区间上。比如数字3会落到服务器 A 对应的区间上，此时返回服务器 A 即可。权重越大的机器，在坐标轴上对应的区间范围就越大，因此随机数生成器生成的数字就会有更大的概率落到此区间内。只要随机数生成器产生的随机数分布性很好，在经过多次选择后，每个服务器被选中的次数比例接近其权重比例。比如，经过一万次选择后，服务器 A 被选中的次数大约为5000次，服务器 B 被选中的次数约为3000次，服务器 C 被选中的次数约为2000次。
  -  RandomLoadBalance 的算法思想比较简单，在经过多次请求后，能够将调用请求按照权重值进行“均匀”分配。当然 RandomLoadBalance 也存在一定的缺点，当调用次数比较少时，Random 产生的随机数可能会比较集中，此时多数请求会落到同一台服务器上。这个缺点并不是很严重，多数情况下可以忽略。RandomLoadBalance 是一个简单，高效的负载均衡实现，因此 Dubbo 选择它作为缺省实现。 

  ​      

##### RoundRobin

- • **RoundRobin** ：**权重轮询, 权重高, 优先轮询, 参与度高。**
- **官方解释：**
  -  这里从最简单的轮询开始讲起，所谓轮询是指将请求轮流分配给每台服务器。举个例子，我们有三台服务器 A、B、C。我们将第一个请求分配给服务器 A，第二个请求分配给服务器 B，第三个请求分配给服务器 C，第四个请求再次分配给服务器 A。这个过程就叫做轮询。轮询是一种无状态负载均衡算法，实现简单，适用于每台服务器性能相近的场景下。但现实情况下，我们并不能保证每台服务器性能均相近。如果我们将等量的请求分配给性能较差的服务器，这显然是不合理的。因此，这个时候我们需要对轮询过程进行加权，以调控每台服务器的负载。经过加权后，每台服务器能够得到的请求数比例，接近或等于他们的权重比。比如服务器 A、B、C 权重比为 5:2:1。那么在8次请求中，服务器 A 将收到其中的5次请求，服务器 B 会收到其中的2次请求，服务器 C 则收到其中的1次请求。 

​       

##### LeastActive

- • **LeastActive**：**权重最少连接数, 调用次数越少, 被调用概率越大, 连接数一致时, 权重越高, 概率越大。**
- **官方解释：**
  - 最小活跃数负载均衡。活跃调用数越小，表明该服务提供者效率越高，单位时间内可处理更多的请求。此时应优先将请求分配给该服务提供者。在具体实现中，每个服务提供者对应一个活跃数 active。初始情况下，所有服务提供者活跃数均为0。每收到一个请求，活跃数加1，完成请求后则将活跃数减1。在服务运行一段时间后，性能好的服务提供者处理请求的速度更快，因此活跃数下降的也越快，此时这样的服务提供者能够优先获取到新的服务请求、这就是最小活跃数负载均衡算法的基本思想。除了最小活跃数，LeastActiveLoadBalance 在实现上还引入了权重值。所以准确的来说，LeastActiveLoadBalance 是基于加权最小活跃数算法实现的。举个例子说明一下，在一个服务提供者集群中，有两个性能优异的服务提供者。某一时刻它们的活跃数相同，此时 Dubbo 会根据它们的权重去分配请求，权重越大，获取到新请求的概率就越大。如果两个服务提供者权重相同，此时随机选择一个即可 

##### ConsistentHash

- • **ConsistentHash：一致性hash, 相同的请求参数, 永远请求到同一个服务提供者** 
- **官方解释：**
  - 一致性 hash 算法由麻省理工学院的 Karger 及其合作者于1997年提出的，算法提出之初是用于大规模缓存系统的负载均衡。它的工作过程是这样的，首先根据 ip 或者其他的信息为缓存节点生成一个 hash，并将这个 hash 投射到 [0, 232 - 1] 的圆环上。当有查询或写入请求时，则为缓存项的 key 生成一个 hash 值。然后查找第一个大于或等于该 hash 值的缓存节点，并到这个节点中查询或写入缓存项。如果当前节点挂了，则在下一次查询或写入缓存时，为缓存项查找另一个大于其 hash 值的缓存节点即可 



##### 代码配置

- Provider 和 Consumer 中都可以进行配置，推荐配置在Provider中

###### dubbo-service 配置

```java
//权重随机数, 权重高, 随机概率大
@Service(weight = 20, loadbalance = "random")

//权重轮询, 权重高, 优先轮询, 参与度高
@Service(weight = 20, loadbalance = "roundrobin")

//权重最少连接数, 调用次数越少, 被调用概率越大, 连接数一致时, 权重越高, 概率越大
@Service(weight = 20, loadbalance = "leastactive")

//一致性hash, 相同的请求参数, 永远请求到同一个服务提供者
@Service(loadbalance = "consistethash")
```

######  dubbo-web 配置

- 消费者端配置，**不推荐**，**无法配置 权重**

```java
//权重随机数, 权重高, 随机概率大
@Reference(loadbalance = "random")

//权重轮询, 权重高, 优先轮询, 参与度高
@Reference(loadbalance = "roundrobin")

//权重最少连接数, 调用次数越少, 被调用概率越大, 连接数一致时, 权重越高, 概率越大
@Reference(loadbalance = "leastactive")

//一致性hash, 相同的请求参数, 永远请求到同一个服务提供者
@Reference(loadbalance = "consistethash")
```

 

#### 4.7 集群容错

- 集群容错模式： 
  - • Failover Cluster：失败重试。**默认值**。当出现失败，重试其它 服务器 ，默认重试2次，使用retries 配置。一般用于读操作 
  - • Failfast Cluster ：快速失败，只发起一次调用，失败立即报错。 通常用于写操作。
  - • Failsafe Cluster ：失败安全，出现异常时，直接忽略。返回一 个空结果。 
  - • Failback Cluster ：失败自动恢复，后台记录失败请求，定时 重发。通常用于消息通知操作。 
  - • Forking Cluster ：并行调用多个服务器，只要一个成功即返回。
  -  • Broadcast  Cluster ：广播调用所有提供者，逐个调用，任意 一台报错则报错。

##### dubbo-service 配置

```java
//Failover、 Failfast、 Failsafe、 Failback、 Forking、 Broadcast  
//failover: 重试其它 服务器, retries = 2 重试两次，默认值也是2
@Service(cluster = "failover", retries = 2)
public class UserServiceImpl implements UserService {
```

#####  dubbo-web 配置

- 消费者端配置，**不推荐**，**无法配置 权重**

```java
//failover: 重试其它 服务器, retries = 2 重试两次，默认值也是2
@Reference(cluster = "failover",  retries = 2 )
private UserService userService;
```



#### 4.8 服务降级

- http://dubbo.apache.org/zh-cn/docs/user/demos/service-downgrade.html

- `mock=force:return null` 表示消费方对该服务的方法调用都直接返回 null 值，不发起远程调用。用来屏蔽不重要服务提供者不可用时对调用方消费者的影响。
-  `mock=fail:return null` 表示消费方对该服务的方法调用在失败后，再返回 null 值，不抛异常。用来容忍不重要服务不稳定时对调用方的影响。

##### 自动降级

###### dubbo-service 配置

- 服务提供者不支持配置  mock, 虽然编写代码可以设定，但是会发生异常

```java
@Service(mock = "force:return null")
public class UserServiceImpl implements UserService {}

//异常如下：不支持Provider 端配置 服务降级
org.springframework.beans.MethodInvocationException: Property 'mock' threw exception; nested exception is java.lang.IllegalArgumentException: mock doesn't support on provider side
       
```

######  dubbo-web 配置

- 消费者端配置，**推荐**

```java
/** 服务降级 **/
//force:return null，不发起调用provider，一般用于：得知 提供者不可用时，消费者主动放弃调用它
//fail:return null ，发起调用执行失败，不发生异常，一般用于： 不希望调用不重要服务提供者发生异常时，影响程序继续往下运行
@Reference(mock = "force:return null")
private UserService userService;
```



##### 手动降级

dubbo-admin 管理平台也可以对消费者Consumer配置：

   **屏蔽：force.mock** （即：屏蔽请求，直接返回某个值，如上面的字符串，*mock="return null"*）;

   **容错：fail.mock** （即：允许请求，在请求失败的时候，再返回某个值，如：*mock="fail:return null"*）;



### 5. 扩展

#### 超时&重试

- dubbo 超时和重试配置有三种方式，我们只讲了一种最常用方式：**接口级配置**
  
- 全局配置， 接口级配置，方法级配置
  
- 覆盖规则：

  - 方法级的配置的优先级>接口级配置的优先级>全局级配置的优先级，即遵循就近原则，以此为基础；

  - 在同级情况下，消费者的优先级大于提供者的优先级；

  - 优先级高的会将优先级低的覆盖；

      

- 服务提供者配置方法级别超时：

  ```java
  package com.itheima.service.impl;
  
  import com.itheima.pojo.User;
  import com.itheima.service.UserService;
  import org.apache.dubbo.config.annotation.Method;
  import org.apache.dubbo.config.annotation.Service;
  
  //findUserById 方法超时时间为：3000 毫秒
  @Service(methods = {@Method(name = "findUserById", timeout = 3000)})
  public class UserServiceImpl implements UserService {
  
      public User findUserById(int id) {
          System.out.println("3...");
          //查询User对象
          User user = new User(1,"zhangsan","123");
          return user;
      }
  }
  ```

- 服务提供者全局配置超时：

  ```xml
  <dubbo:provider timeout="3000"></dubbo:provider>
  ```

#### 服务接口设计原则

**接口配置：**

dubbo推荐在Provider上尽量多配置Consumer端属性，原因如下：

- 让provider开发人员一开始就思考编写的服务提供者特点、服务质量等问题

- 作为服务的提供者，比服务方更清楚服务性能的参数，如调用时间，合理的重试次数等，所以这些参数应尽量配置在服务的提供者方；
- 共性的配置不应该在多个Consumer端进行配置，例如: 超时时间 是 消费端都要考虑的配置，应该尽可能的配置在Provider中，这样便于统一管理，反之是不合理的。
- Consumer 的配置多用于配置 Provider 缺省的配置项，这些配置项往往是各个客户端差异性配置

**接口粒度：**

- 服务接口尽可能大粒度，每个服务方法应代表一个功能，而不是某一个功能其中一个步骤，否则将面临分布式事务问题，dubbo暂不支持分布式事务，同时可以减少网络IO
- 服务接口建议以业务场景为单位进行划分，并对相近业务做抽象，防止接口数量爆炸
- 不建议使用过于抽象的通用的接口，如：Map query(Map map), 这样的接口没有明确语意，会给后期的维护带来不便



#### Dubbo服务子系统划分

**服务化目标**：基础服务为上游核心业务服务实现功能提供支持，基础服务本身无状态，可随着系统的负荷灵活伸缩来提供服务能力

**服务子系统数量把控**：

- 过多：可能划分过细，破坏业务子系统的独立性，如：支付订单，退款订单，用户，账户
  - 部署维护工作量大，独立进程占用内存多
- 过少：没能很好的解耦，开发维护不好分工，升级维护影响面答



**服务子系统间避免出现循环依赖调用**

**服务子系统间的依赖关系链不要过长**

**尽量避免分布式事务**

**服务子系统的划分是一个不断优化的过程**