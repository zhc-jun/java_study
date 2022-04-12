# Tomcat和HTTP协议

### 今日重点

- tomcat可安装，启动，关闭，集成到idea中
- 通过idea创建javaweb项目，简单实现servlet可以接收客户端请求。
- 说出HTTP协议请求组成部分，并说出请求方式有哪些，请求头有哪些，什么含义？
- 说出HTTP协议响应组成部分，状态码：200  404  500 302分别是什么意思？, 常见的响应头有哪些？
- 熟知Tomcat软件对项目关键文件夹和文件的要求事项。
- 能够知道war包是什么。
- 动手实操，linux安装JDK 和 Tomcat，将web项目发布上去。（之前的CSS案例也可以发布）
- 准确说出什么是：B/S 架构 和  C/S 架构。
- 知道虚拟目录的作用，并且会在idea中进行设置。
- 知道javaweb项目如果有三方依赖jar，必须要放在那个目录下

### 1. 企业开发简介

#### 1.1 JavaEE 规范-(了解)

- JavaEE(Java Enterprise Edition)：Java 企业版。 
- 它是由 SUN 公司领导、各个厂家共同制定并得到广泛认可的工业标准。
- JavaEE 早期叫做 J2EE，但是没有继续采用其命名规则。J2EE 的版本从1.0 开始到 1.4 结束。而 JavaEE 版本是从 JavaEE 5 版本开始，目前最新的版本是JavaEE 8。 
- JavaEE 规范是很多 Java 开发技术的总称。这些技术规范都是沿用自J2EE 的。一共包括了13 个技术规范。
- 包括：JDBC，JNDI，EJB，RMI，IDL/CORBA，JSP，Servlet，XML，JMS，JTA，JTS，JavaMail，JAF。



#### 1.2 WEB 概述

- WEB 在计算机领域中代表的是网络。 
- 像我们之前所用的WWW，它是World Wide Web 三个单词的缩写，称为：万维网。
- 网络相关技术的出现都是为了让我们在网络的世界中获取资源，这些资源的存放之处，我们把它叫做网站。
- 我们通过输入网站的地址(网址)，就可以访问网站中提供的资源(不区分局域网或广域网)。

#### 1.3 资源分类

- 静态资源 

  > 由静态网页开发技术实现的资源为静态资源：常见的技术有 HTML、CSS、JavaScript 等。

- 动态资源 

  > 由动态网页开发技术实现的资源为动态资源，常见的技术有  JSP、Servlet php, python 等, 我们重点掌握jsp + servlet ;

#### 1.4 系统结构

在我们前面课程的学习中，开发的都是Java 工程。这些工程在企业中称之为项目或者产品。它都是有系统架构的！ 

- 基础结构划分： C/S 结构 B/S 结构 
- 技术选型划分： Model1 模型 Model2 模型 MVC 模型 三层架构+ MVC 模型 
- 部署方式划分： 一体化结构 垂直拆分结构 分布式结构 微服务结构

##### CS 结构：

- (Client Server) 客户端+服务器的方式；QQ，LOL，桌面版爱奇艺....
- 特点：通过安装包安装客户端程序 (Client), 如果没有网络没法正常使用，就证明它存在服务器端（Server），这类软件都是C/S 架构的。

#####  BS 结构：

- (Browser Server) 浏览器+服务器的方式； 所有用浏览器打开的网站都是B/S 架构的网站，反之都不是。



### 2. Tomcat 服务器（重点）

#### 2.1 服务器的介绍

- 服务器是计算机的一种，它比普通计算机运行更快、负载更高、价格更贵。
- 服务器在网络中为其它客户机(如PC 机、智 能设备等)提供计算或者应用服务。
- 服务器具有高速的CPU 运算能力、长时间的可靠运行、强大的I/O 外部数据吞吐能 力以及更好的扩展性。 
- 在企业中我们所说的服务器，一般泛指web 服务器，它的特点是：在上述计算机中**安装**一款可以处理web请求和响应的**软件**后，就是我们所说的**web服务器。**

#### 2.2  常用的应用服务器

| 服务器名称                     | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| weblogic                       | 实现了javaEE规范，重量级服务器，又称为javaEE容器。           |
| websphereAS                    | 实现了javaEE规范，重量级服务器。                             |
| JBOSSAS                        | 实现了JavaEE规范，重量级服务器，免费的。                     |
| <font color=red>Tomcat </font> | <font color=red>实现了jsp/servlet规范，是一个轻量级服务器，开源免费。</font> |
| Jetty                          | 比Tomcat更轻量，开源免费。                                   |

#### 2.3 Tomcat 的介绍

- Tomcat 是 Apache 软件基金会的 Jakarta 项目组中的一个核心项目，由Apache、Sun 和其他一些公司及个人共同开 发而成。
- 由于有了Sun 公司的参与和支持，最新的Servlet、JSP 规范总是能在Tomcat 中得到体现。因为Tomcat 技 术先进、性能稳定，而且免费，所以深受Java 爱好者的喜爱并得到了部分软件开发商的认可，成为目前比较流行的 Web 应用服务器。 
- Tomcat 官网：https://tomcat.apache.org/



#### 2.4 Tomcat 的下载和安装

- 下载：https://tomcat.apache.org/ 

- 安装：直接解压即

- 目录介绍：bin    webapps   logs

  > bin : 启动文件在该目录
  >
  > conf：tomcat核心配置文件
  >
  > lib：tomcat自身所需jar包目录
  >
  > logs：tomcat启动和运行时日志目录
  >
  > tmp：临时文件目录
  >
  > work：tomcat自身工作目录
  >
  > webapps：Javaweb项目部署的目录

#### 2.5 Tomcat 的基本使用

- 启动 : 

  >  bin目录下执行：
  >
  > ​    startup.bat   //windows下启动执行文件
  >
  > ​    startup.sh    //linux下启动执行文件 

- 停止： 

  > bing目录下执行：
  >
  > ​     shutdown.bat   //windows下关闭执行文件 
  >
  > ​     shutdown.sh     //linux下关闭执行文件 

- 启动问题 

  1）启动窗口一闪而过：没有配置jdk 环境变量, 变量名必须叫 JAVA_HOME 

   2）java.net.BindException：端口8080 被占用 

- 部署自己的项目 

  > ① 在 webapps 目录下创建一个文件夹html
  >
  > ② 将资源放到该文件夹里 
  >
  > ③ 启动 tomcat，输入正确路径: localhost:8080/html/案例二/04案例二：登录页面.html

#### 2.6 Idea 与 Tomcat集成

- 参考博客：https://blog.csdn.net/m0_37508531/article/details/90510087

- 也可参考当天PDF教程进行配置，或者跟着老师一起做。



#### 2.7 项目组成详解 

- src：存放源代码的 

- web：存放项目相关资源的(html、css、js、jsp、图片等

- WEB-INF：存放相关配置的(web.xml等)

  > <font color=red>如果项目用到了三方框架，比如Mybatis，该框架的jar包必须放在WEB-INF/lib/ 目录下，lib需要自己创建，理由：tomcat只会去该目录下加载三方jar进内存，规定死的。</font>

- WEB-INF下还有一个文件夹classes，用来存放所有.java文件编译成.class 后存放目录，classes文件夹的命名和位置，也是Tomcat强制要求的，我们不可以更改，idea会帮我们将java代码编译后放在该目录下，无需我们操心。

#### 2.8 war文件的介绍

- war 是一种压缩文件格式，在企业中，我们喜欢将javaweb项目打成war包，然后发布到Tomcat安装目录下的webapps文件夹下，启动tomcat进行项目的发布。

- java命令生成war包

  > 1）idea 右键java项目下WEB-INF文件夹 --> Show in Explorer ---> 右键Git Bash -- > 输入命令：jar -cvf myweb.war .
  >
  > 2）可以在当前目录下，查看到myweb.war
  >
  > 3）如果没有安装git, 也可以按住Shift + 鼠标右键空白处，找到：“在此处 打开 PowerShell”, 输入命令即可。



#### 2.9 Tomcat 核心配置文件&虚拟目录

##### server.xml 文件

- Tomcat默认安装目录下conf文件夹/server.xml 就是 tomcat核心配置文件，tomcat默认端口为8080就是在该文件中，可以进行更改。
- server.xml 文件在企业中不会随意更改，如果一旦手误修改错误，将会引起tomcat启动不起来，总之: 不要更改该文件。

##### 虚拟目录

- tomcat是支持部署多个java项目的，虚拟目录的作用就是在URL访问时，来区分访问的是项目A还是项目B。

- 举例：

  1）projec_A.war   project_B.war 上传到 tomcat/webapps/目录下户

  2）启动tomcat

  3）访问projec_A项目下的index.jsp: localhost:8080/project_A/index.jsp

  4）访问projec_B项目下的index.jsp: localhost:8080/project_B/index.jsp

- <font color=red>可以发现：war包的名字就是当前项目的虚拟目录。</font>

- idea 中也可以为项目设置虚拟目录



#### 2.10 虚拟主机

- 这东西实际就是一个字符串到ip的映射

- 本地ip地址为: 127.0.0.1, window默认的映射字符串为: "localhost"

- 我们也可以继续增加字符串到ip的映射：C:\WINDOWS\system32\drivers\etc

- 编辑hosts 文件, 增加如下内容

  > 127.0.0.1 itcast

- <font color=red>此时我们访问 http://itcast:8080  等同于 http://127.0.0.1:8080  </font>



### 3. HTTP 协议 （了解）

#### 3.1 HTTP 协议的介绍

- HTTP(Hyper Text Transfer Protocol)：超文本传输协议。 
- HTTP 协议是基于 TCP/IP 协议的。 
- 超文本：比普通文本更加强大。 
- 传输协议：客户端和服务器端的通信规则。
- HTTP是基于TCP的上层协议，TCP是传输层协议，HTTP是应用层协议。

**特点**

```shell
1. 基于TCP/IP的高级协议
2. 默认端口号:80
3. 基于请求/响应模型的:一次请求对应一次响应

* 历史版本：
    * 1.0：每一次请求响应都会建立新的连接
    * 1.1：复用连接, 效率更高
```



#### 3.2 HTTP 协议的请求报文结构

- 请求行
- 请求头
- 请求空行 （请求方式为POST时才会有）
- 请求体（请求方式为POST时才会有）

```html
<!-- 
* 请求消息数据格式
	1. 请求行
		请求方式 请求url 请求协议/版本
		GET /login.html	HTTP/1.1

		* 请求方式：
			* HTTP协议有7中请求方式，常用的有2种
				* GET：
					1. 请求参数在请求行中，在url后。
					2. 请求的url长度有限制的
					3. 不太安全
				* POST：
					1. 请求参数在请求体中
					2. 请求的url长度没有限制的
					3. 相对安全
	2. 请求头：客户端浏览器告诉服务器一些信息
		请求头名称: 请求头值
		* 常见的请求头：
			1. User-Agent：浏览器告诉服务器，我访问你使用的浏览器版本信息
				* 可以在服务器端获取该头的信息，解决浏览器的兼容性问题

			2. Referer：http://localhost/login.html
				* 告诉服务器，我(当前请求)从哪里来？
					* 作用：
						1. 统计工作：
	3. 请求空行
		空行，就是用于分割POST请求的请求头，和请求体的。
	4. 请求体(正文)：
		* 封装POST请求消息的请求参数的

	* 字符串格式：
		POST /login.html	HTTP/1.1
		Host: localhost
		User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:60.0) Gecko/20100101 Firefox/60.0
		Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
		Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
		Accept-Encoding: gzip, deflate
		Referer: http://localhost/login.html
		Connection: keep-alive
		Upgrade-Insecure-Requests: 1
		
		username=zhangsan
-->        
```

#### 3.3 HTTP 协议响应报文结构

- 响应行
- 响应头
- 响应空行 
- 响应体 

```html
<!--	
	响应消息：服务器端发送给客户端的数据
	* 数据格式：
		1. 响应行
			1. 组成：协议/版本 响应状态码 状态码描述
			2. 响应状态码：服务器告诉客户端浏览器本次请求和响应的一个状态。
				1. 状态码都是3位数字 
				2. 分类：
					1. 1xx：服务器接收客户端消息，但没有接受完成，等待一段时间后，发送1xx多状态码
					2. 2xx：成功。代表：200
					3. 3xx：重定向。代表：302(重定向)，304(访问缓存)
					4. 4xx：客户端错误。
						* 代表：
							* 404（请求路径没有对应的资源） 
							* 405：请求方式没有对应的doXxx方法
					5. 5xx：服务器端错误。代表：500(服务器内部出现异常)
					
	     2. 响应头：
			1. 格式：头名称： 值
			2. 常见的响应头：
				1. Content-Type：服务器告诉客户端本次响应体数据格式以及编码格式
				2. Content-disposition：服务器告诉客户端以什么格式打开响应体数据
					* 值：
						* in-line:默认值,在当前页面内打开
						* attachment;filename=xxx：以附件形式打开响应体。文件下载
         3. 响应空行
         4. 响应体:传输的数据


	* 响应字符串格式
		HTTP/1.1 200 OK
		Content-Type: text/html;charset=UTF-8
		Content-Length: 101
		Date: Wed, 06 Jun 2018 07:08:42 GMT

		<html>
		  <head>
		    <title>$Title$</title>
		  </head>
		  <body>
		  hello , response
		  </body>
		</html>
-->
```





### 4. 案例 发布静态资源 （了解）

#### 4.1实现步骤

1）创建一个 JavaWEB 项目。 

2）将静态页面所需资源导入到项目的web 目录下。 

3）修改 web.xml 配置文件，修改默认主页。 

4）将项目部署到 tomcat 中。 

5）启动 tomcat 服务。 

6）打开浏览器测试查看页面。

**web.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <welcome-file-list>
        <welcome-file>/news/news.html</welcome-file>
    </welcome-file-list>
</web-app>
```



### 5. 案例 发布动态资源 （多练，必须看懂）

#### 5.1 Servlet 的介绍

- Servlet 是Tomcat 为我们提供的一个框架，它可以帮我们实现在java代码中接收和响应来自客户端基于HTTP 协议的报文数据。
- 如果想实现 Servlet 的功能，可以通过实现javax.servlet.Servlet接口或者继承它的实现类。 
- 核心方法：service()，任何客户端的请求都会经过该方法。 



#### 5.2 实现步骤

1）创建一个 JavaWEB 项目。 

2）将静态页面所需资源导入到项目的web 目录下。 

3）修改 web.xml 配置文件，修改默认主页。 

4）在项目的 src 路径下编写一个类，实现Servlet 接口。 

5）重写 service 方法，输出一句话即可。 

6）修改 web.xml 配置文件，配置 servlet 相关资源。 

7）将项目部署到 tomcat 中。 

8）启动 tomcat 服务。 

9）打开浏览器测试功能。



##### StudentServlet.java

```java
package com.itheima.servlet;

import javax.servlet.*;
import java.io.IOException;

/*
    Servlet快速入门
 */
public class StudentServlet implements Servlet{

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {

    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    /*
        客户端所有请求都会经过service方法
     */
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("这是我的第一个servlet入门案例...");
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {

    }
}

```

##### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <!--修改默认主页-->
    <welcome-file-list>
        <welcome-file>/html/frame.html</welcome-file>
    </welcome-file-list>

    <!--配置Servlet-->
    <servlet>
        <servlet-name>studentServlet</servlet-name>
        <servlet-class>com.itheima.servlet.StudentServlet</servlet-class>
    </servlet>

    <!--配置Servlet映射-->
    <servlet-mapping>
        <servlet-name>studentServlet</servlet-name>
        <url-pattern>/studentServlet</url-pattern>
    </servlet-mapping>
</web-app>
```

#### 5.3 执行流程

- 回顾网络通信三要素：ip  端口 协议
- 回顾Tomcat虚拟目录：tomcat启动时可以发布多个java项目，虚拟目录用来做URL访问时项目之间的区别，当只有一个项目发布时，虚拟目录可以设置为：/

**浏览器发起请求**：http://localhost:8080/crm/studentServlet

- http 是协议
- localhost 是对127.0.0.1 该ip的映射，找到这台计算机
- 8080是tomcat默认端口，找到tomcat软件
- crm是tomcat发布java项目的虚拟目录，找到该项目
- studentServlet是crm项目下可以被浏览器访问的资源，该项目下的html  css  js  png 等文件都是可以被访问的资源，这里可以明确知道，浏览器找的是studentServlet 背后的那个.java文件，com.itheima.servlet.StudentServlet
- tomcat默认会执行service()方法，规定死的，这也就是为什么我们要实现Servlet接口，你不实现，tomcat怎么去调用你的service()方法，不掉用该方法，tomcat怎么在接收到浏览器URL请求后，执行你在service()中编写的业务代码。

##### 解惑：

- 正因为我们java项目交给tomcat去执行，所以我们给tomcat叫做容器，准确说java项目容器。
- 正因为tomcat接收请求后，会去调用service方法，该方法用Servlet接口进行了规范，所以我们要学习Servlet框架，我们要想接收浏览器发来的数据，就得实现Servlet接口，在service()方法中，编写代码 接受数据 和 对客户端做出响应（后面会讲具体代码），也正因为这样，我们给自己实现的Servlet叫做web程序执行入口。同时Servlet也是java语言服务器端开发必不可少的技术，主要负责和客户端交互数据，也就是所谓的：**请求 和 响应** （有来有回）。

