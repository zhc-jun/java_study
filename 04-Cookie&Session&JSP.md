# Cookie&Session&JSP

### 今日目标

- 掌握Cookie和Session两种会话
- 基于JSP完成学生管理系统
- JSP基础知识中，重点看看指令和注释，其他语法每天都会被替代掉，看看就可以了。



### 1. Cookie（重点）

#### 1.1 会话介绍

- **会话**：现实生活中，会话用来描述一次活动从**开始**到**中间过程**再到**结束**。

- **会话生命周期**：

  1）从浏览器第一次访问服务器开始，会话就已经建立了，用户不断的操作页面，浏览器和服务器之间不断的请求与响应，当用户关闭浏览器后，会话也就结束了。

-  **Cookie会话技术**：

  1）浏览器和服务器建立会话后，可以利用Cookie技术，将部分业务数据保存在浏览器中，当某些业务需要时，可以去取。

  2）当Cookie将数据存储在浏览器中后，接下来的每次请求都会在**请求报文的头**中携带该数据，从而实现多次请求的数据共享。

  3）值得一提的是，浏览器存储的共享数据具体内容，Java程序员编写代码来设定。

  4）Cookie中的数据都是key value 形式存放的。

  

#### 1.2 Cookie 属性

- 重点name  value path maxAge

| 属性名  | 作用                   | 是否重要 |
| ------- | ---------------------- | -------- |
| name    | Cookie的名称           | 必须属性 |
| value   | Cookie的值(不支持中文) | 必须属性 |
| path    | Cookie的路径           | 重要     |
| domain  | Cookie的域名           | 重要     |
| maxAge  | Cookie的存活时间       | 重要     |
| version | Cookie的版本号         | 不重要   |
| comment | Cookie的描述           | 不重要   |

#### 1.3 Cookie 对象

- java中Cookie对象只是临时存储一下key  value, 最终数据要交给response对象设置在响应报文的响应头中交给浏览器解析并存储。

| 构造方法                         | 作用                             |
| -------------------------------- | -------------------------------- |
| Cookie(String name,String value) | 创建对象，并设置Cookie存储的数据 |
| 属性对应的set和get方法           | 赋值和获取值                     |



#### 1.4 Cookie 添加和获取

- 添加：指的是将Cookie对象中的数据交给response对象，借助response对象，将数据添加到响应报文的响应头中，交给浏览器解析并存储。
- 获取：当数据被浏览器存储后，接下来的每次浏览器发起的请求头中都会携带该数据，这里说的获取，是指Java如何利用request对象从请求报文头中获取该数据。

<font color=red>添加：HttpServletResponse</font>

| 返回值 | 方法名                   | 说明               |
| ------ | ------------------------ | ------------------ |
| void   | addCookie(Cookie cookie) | 向客户端添加Cookie |

<font color=red>获取：HttpServletRequest</font>

| 返回值   | 方法名       | 说明             |
| -------- | ------------ | ---------------- |
| Cookie[] | getCookies() | 获取所有的Cookie |



#### 1.5 Cookie 的使用 （理解）

- 需求说明

  通过 Cookie 记录最后访问时间，并在浏览器上显示出来。 

- 最终目的 

  掌握 Cookie 的基本使用，从创建到添加客户端，再到从服务器端获取。

- 实现步骤 

  1）通过响应对象写出一个提示信息。 

  2）创建 Cookie 对象，指定name 和 value。 

  3）设置 Cookie 最大存活时间。 

  4）通过响应对象将Cookie 对象添加到客户端。 

  5）通过请求对象获取Cookie 对象。 

  6）将 Cookie 对象中的访问时间写出。

##### 示例代码

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.text.SimpleDateFormat;
import java.util.Date;

/*
    Cookie的使用
    1、cookie怎么创建的？
    new Cookie(name,value)  在服务器端的Java代码（Servlet）中创建的
    2、cookie存储在哪？ 通过响应resp写入到浏览器端的  resp.addCookie()      
    存储在浏览器端的
    3、cookie如何获取？  通过请求req携带到服务器端   req.getCookies();
    在服务器端获取和操作cookie中的数据。
    4、cookie什么时候删除？
    过期删除。。。。
    
    
 */
@WebServlet("/servletDemo01")
public class ServletDemo01 extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.通过响应对象写出提示信息
        resp.setContentType("text/html;charset=UTF-8");
        PrintWriter pw = resp.getWriter();
        pw.write("欢迎访问本网站，您的最后访问时间为：<br>");

        //2.创建Cookie对象，用于记录最后访问时间
        Cookie cookie = new Cookie("time",System.currentTimeMillis()+"");

        //3.设置最大存活时间
        cookie.setMaxAge(3600);
//        cookie.setMaxAge(0);    // 立即清除

        //4.将cookie对象添加到客户端
        resp.addCookie(cookie);

        //5.获取cookie
        Cookie[] arr = req.getCookies();
        for(Cookie c : arr) {
            if("time".equals(c.getName())) {
                //6.获取cookie对象中的value，进行写出
                String value = c.getValue();
                SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                pw.write(sdf.format(new Date(Long.parseLong(value))));
            }
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```



#### 1.6 Cookie 的细节

- 数量限制 

  > 每个网站最多只能有20 个 Cookie，且大小不能超过4KB。所有网站的Cookie 总数不能超过300 个。

- 名称限制 

  > Cookie 的名称只能包含ASCCI 码表中的字母、数字字符。不能包含逗号、分号、空格，不能以$ 开头。 Cookie 的值不支持中文。 

- 存活时间限制 

  > setMaxAge() 方法接收数字: 
  >
  > 1）负整数（默认）：在内存中存储，没有过期时间，浏览器关闭伴随着内存清空而被清除。  会话级别的cookie
  >
  > 2）0：告知浏览器立即清除。  
  >
  > 3）正整数：以秒为单位设置存活时间。  持久级别的cookie

- Cookie共享范围(path):  一般企业都是采用默认范围

  > 1）默认路径：在当前项目虚拟目录下的资源，都可以访问。
  >
  > 2）设置路径：setPath() 
  >
  > - setPath("/")   当前tomcat容器部署的所有java项目都可以共享
  > - setpath("/project1")  当前tomcat 虚拟目录project1下所有servlet都可以共享
  > - setPath("/project1/userLogin"); 当前tomcat 虚拟目录project1下url-partten 为 userLogin 的Servlet才能得到该cookie中数据

cookie的path的范围 >= 请求的url地址的路径范围  此时，该cookie会随着请求携带过去！！

domain属性：



### 2. Session（重点）

#### 2.1 HttpSession 介绍

- HttpSession：服务器端会话管理技术，区别于Cookie，它是将数据已对象的形式存储在Tomcat开辟的jvm内存中。
- session 就是借助域方法来将数据进行存储，区别于request 和 servletContext，它可以对数据设置过期时间。
- session原理要依赖cookie对当前浏览器做一个标识，用来识别每一个客户端浏览器
- Tomcat会对每一个客户端浏览器，都创建一个session对象。
- 同一个浏览器发起的多次请求给不同的servlet，这些servlet可以实现seesion域中的数据，所以，session域的共享范围要大于request。
- session域中数据无法对 多个浏览器对servlet发起的请求，提供数据共享，所以域共享范围要小于 ServletContext。



#### 2.2 Servlet 域对象比较

- 共享范围:  ServletContext  > Session > request

| 域对象         | 功能   | 作用                                                       |
| -------------- | ------ | ---------------------------------------------------------- |
| ServletContext | 应用域 | 多个客户端浏览器，多次请求间可以共享域中数据               |
| HttpSession    | 会话域 | 同一个浏览器多次请求间，可以共享域中数据                   |
| ServletRequest | 请求域 | 一次请求间，转发到多个servlet时，所有servlet可以实现域共享 |



#### 2.3 HttpSession 获取

- HttpSession 实现类对象是通过HttpServletRequest 对象来获取。

| 返回值      | 方法名                      | 说明                                       |
| ----------- | --------------------------- | ------------------------------------------ |
| HttpSession | getSession()                | 获取HttpSession对象                        |
| HttpSession | request.getSession(true)：  | 若存在会话则返回该会话，否则新建一个会话。 |
| HttpSession | request.getSession(false)： | 若存在会话则返回该会话，否则返回NULL       |



#### 2.4 Session 常用方法

- 域操作三个方法
- Invalidate() 方法多用于网页退出/注销功能的实现。

| 返回值 | 方法名                                 | 说明              |
| ------ | -------------------------------------- | ----------------- |
| void   | setAttribute(String name,Object value) | 设置域共享数据    |
| Object | getAttribute(String name)              | 获取域共享数据    |
| void   | removeAttribute(String name)           | 移除域共享数据    |
| String | getId()                                | 获取唯一标识名称  |
| void   | Invalidate()                           | 让session立即失效 |

#### 2.5 Session 的使用（理解）

- 需求说明 

  > 通过第一个 Servlet 设置共享数据用户名，并在第二个Servlet 获取到。

- 最终目的 

  > 掌握 HttpSession 的基本使用，如何获取和使用。

- 实现步骤 

  1）在第一个 Servlet 中获取请求的用户名。 

  2）获取 HttpSession 对象。 

  3）将用户名设置到共享数据中。 

  4）在第二个 Servlet 中获取 HttpSession 对象。 

  5）获取共享数据用户名。 

  6）将获取到用户名响应给客户端浏览器。

##### 示例代码

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

/*
    Session的基本使用
 */
@WebServlet("/servletDemo01")
public class ServletDemo01 extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.获取请求的用户名
        String username = req.getParameter("username");

        //2.获取HttpSession的对象
        HttpSession session = req.getSession();
        /*
        req.getSession() == req.getSession(true);
        // 一定会返回session对象！ 有则返回，无则创建。
        
        req.getSession(false);
        // 找到则返回，找不到就返回null
        
        */
        
        
        
        System.out.println(session);
        System.out.println(session.getId());

        //3.将用户名信息添加到共享数据中
        session.setAttribute("username",username);

        //4.制作超链接,点击后请求servletDemo02
        resp.getWriter().write("<a href='http://localhost:8080"+req.getContextPath()+"/servletDemo02'>go servletDemo02</a>");

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

/*
    Session的基本使用
 */
@WebServlet("/servletDemo02")
public class ServletDemo02 extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.获取HttpSession对象
        HttpSession session = req.getSession();
        System.out.println(session);
        System.out.println(session.getId());

        //2.获取共享数据
        Object username = session.getAttribute("username");

        //3.将数据响应给浏览器
        resp.getWriter().write(username+"");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```



<font color=red>注意：Session是基于Cookie对浏览器身份的识别，如果浏览器禁用了Cookie，那么Session将非正常工作，一般也没人回去禁用Cookie。</font>

#### 2.6 HttpSession 的细节

- 浏览器禁用 Cookie 

  > 方式一：通过提示信息告知用户，大部分网站采用的解决方式。(推荐) 
  >
  > 方式二：访问时在URL中拼接jsessionid 标识 (了解)

- 钝化和活化

  > 1）服务器正常关闭时，tomcat会将内存中的session对象持久化到磁盘
  >
  > 2）再次启动时，会从磁盘文件将session对象在加载进内存。
  >
  > 3）碍于idea工作原理，我们在idea中无法演示钝化和活化。
  >
  > 4）钝化和活化是Tomcat自己的机制，我们知道就好



### 3. JSP （了解）

#### 3.1 JSP 介绍

- JSP(Java Server Pages)：是一种动态网页技术标准。 
- JSP 部署在服务器上，可以处理客户端发送的请求，并根据请求内容动态的生成HTML、XML 或其他格式文 档的 Web 网页，然后再响应给客户端。 
- JSP 是基于 Java 语言的，它的本质就是Servlet。

| 类别       | 使用场景                               |
| ---------- | -------------------------------------- |
| HTML       | 开发静态资源，无法添加动态资源         |
| CSS        | 美化页面                               |
| JavaScript | 给网页添加一些动态效果                 |
| Servlet    | 编写Java代码，实现后台功能处理         |
| JSP        | 包含了显示页面技术，也具备Java代码功能 |



#### 3.2 JSP 快速入门

1）创建一个 web 项目。 

2）在 web 目录下创建一个 index.jsp 文件。 

3）在文件中写一句内容为：这是我的第一个jsp。 

4）部署并启动项目。 

5）通过浏览器测试。

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>JSP</title>
  </head>
  <body>
    <h1>这是我的第一个jsp</h1>
  </body>
</html>

```

#### 3.2 JSP 文件内容介绍

- 生成的 java 文件目录

  > 1） 查看Idea控制台启动日志
  >
  > Using CATALINA_BASE:   "C:\Users\ZJW\.IntelliJIdea2019.1\system\tomcat\Unnamed_cookie_demo"
  >
  > 
  >
  > 2） 进入如下目录
  >
  > C:\Users\ZJW\.IntelliJIdea2019.1\system\tomcat\Unnamed_cookie_demo\work\Catalina\localhost\jsp_demo_Web_exploded\org\apache\jsp
  >
  > 3）index.jsp 转换为  index_jsp.java代码如下：(为了展示-删减很多代码)
  >
  > ```java
  > /*
  >  * Generated by the Jasper component of Apache Tomcat
  >  * Version: Apache Tomcat/8.5.47
  >  * Generated at: 2020-07-06 07:17:46 UTC
  >  * Note: The last modified time of this file was set to
  >  *       the last modified time of the source file after
  >  *       generation to assist with modification tracking.
  >  */
  > package org.apache.jsp;
  > 
  > import javax.servlet.*;
  > import javax.servlet.http.*;
  > import javax.servlet.jsp.*;
  > 
  > public final class index_jsp extends HttpJspBase
  >     implements JspSourceDependent, JspSourceImports {
  > 
  >   public void _jspService(final request, final response)
  >       throws IOException, ServletException {
  >     try {
  >       response.setContentType("text/html;charset=UTF-8");
  >       pageContext = _jspxFactory.getPageContext(this, request, response,
  >       			null, true, 8192, true);
  >       _jspx_page_context = pageContext;
  >       application = pageContext.getServletContext();
  >       config = pageContext.getServletConfig();
  >       session = pageContext.getSession();
  >       out = pageContext.getOut();
  >       _jspx_out = out;
  > 
  >       out.write("\n");
  >       out.write("<html>\n");
  >       out.write("  <head>\n");
  >       out.write("    <title>JSP</title>\n");
  >       out.write("  </head>\n");
  >       out.write("  <body>\n");
  >       out.write("    <h1>这是我的第一个jsp</h1>\n");
  >       out.write("  </body>\n");
  >       out.write("</html>\n");
  >     } catch (java.lang.Throwable t) {
  >     
  >     } finally {
  >       _jspxFactory.releasePageContext(_jspx_page_context);
  >     }
  >   }
  > }
  > 
  > ```
  >
  > 

- 验证：JSP 本质就是一个Servlet



#### 3.3 JSP 语法

- <%-- 注释的内容--%>；  

  > 快键键ctrl + /

- <% Java代码 %>

  > 在service方法中编写java代码！！！
  >
  > 定义的java代码，在service方法中。service方法中可以定义什么，该脚本中就可以定义什么。

- <%=  Java代码 %>

  > 在service中编写out.println(    Java代码    );
  >
  > 该表达式背后由JspWriter 做支撑，定义的java代码，会输出到页面上。

- <%! Java代码 %>

  > 在servlet类中定义成员变量和方法
  >
  > 定义的java代码，在jsp转换后的java类的成员位置, 可以是属性也可以是方法。



1、<%! Java代码 %>

定义一个变量  Date date = new Date();

定义一个方法   public String sayHello(){ return "hello jsp";}

2、<% Java代码 %>

调用sayHello方法，获取返回值。通过out对象输出到页面

3、<%=  Java代码 %> 将刚刚定义的 date输出。



##### jsp语法.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>jsp语法</title>
</head>
<body>
    <%--
        1. 这是注释
    --%>

    <%--
        2.java代码块
        System.out.println("Hello JSP"); 普通输出语句，输出在控制台
        out.println("Hello JSP");
            out是JspWriter对象，输出在页面上
    --%>
    <%
        System.out.println("Hello JSP");
        out.println("Hello JSP<br>");
        String str = "hello<br>";
        out.println(str);
    %>

    <%--
        3.jsp表达式
        <%="Hello"%>  相当于 out.println("Hello");
    --%>
    <%="Hello<br>"%>

    <%--
        4.jsp中的声明(变量或方法)
        如果加!  代表的是声明的是成员变量
        如果不加!  代表的是声明的是局部变量
    --%>
    <%! String s = "abc";%>
    <% String s = "def";%>
    <%=s%>
    <%! public void getSum(){}%>
    <%--<% public void getSum2(){}%>--%>
</body>
</html>

```

##### jsp语法_jsp.java

```java
/*
 * Generated by the Jasper component of Apache Tomcat
 * Version: Apache Tomcat/8.5.47
 * Generated at: 2020-07-06 07:33:38 UTC
 * Note: The last modified time of this file was set to
 *       the last modified time of the source file after
 *       generation to assist with modification tracking.
 */

public final class jsp语法_jsp extends org.apache.jasper.runtime.HttpJspBase
    implements org.apache.jasper.runtime.JspSourceDependent,
                 org.apache.jasper.runtime.JspSourceImports {

 String s = "abc";
                     
 public void getSum(){}
                     
                     
 void _jspDestroy() {
  }

  public void _jspService(final request, final response)
      throws IOException, ServletException {

    try {
      response.setContentType("text/html;charset=UTF-8");
      pageContext = _jspxFactory.getPageContext(this, request, response,
      			null, true, 8192, true);
      _jspx_page_context = pageContext;
      application = pageContext.getServletContext();
      config = pageContext.getServletConfig();
      session = pageContext.getSession();
      out = pageContext.getOut();
      _jspx_out = out;

      out.write("\r\n");
      out.write("<html>\r\n");
      out.write("<head>\r\n");
      out.write("    <title>jsp语法</title>\r\n");
      out.write("</head>\r\n");
      out.write("<body>\r\n");
      out.write("    ");
      out.write("\r\n");
      out.write("\r\n");
      out.write("    ");
      out.write("\r\n");
      out.write("    ");

        System.out.println("Hello JSP");
        out.println("Hello JSP<br>");
        String str = "hello<br>";
        out.println(str);
    
      out.write("\r\n");
      out.write("\r\n");
      out.write("    ");
      out.write("\r\n");
      out.write("    ");
      out.print("Hello<br>");
      out.write("\r\n");
      out.write("\r\n");
      out.write("    ");
      out.write("\r\n");
      out.write("    ");
      out.write("\r\n");
      out.write("    ");
 String s = "def";
      out.write("\r\n");
      out.write("    ");
      out.print(s);
      out.write("\r\n");
      out.write("    ");
      out.write("\r\n");
      out.write("    ");
      out.write("\r\n");
      out.write("</body>\r\n");
      out.write("</html>\r\n");
    } 
  }
}

```



#### 3.4 JSP 指令

##### page 指令 

> <%@ page 属性名=属性值属性名=属性值… %> 

如下指令属性，能够掌握：contentType   import  errorPage即可。

| 属性名          | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| contentType     | 响应正文支持的类型和设置编码格式                             |
| language        | 使用的语言，默认是java                                       |
| errorPage       | 当前页面出现异常后跳转的页面                                 |
| isErrorPage     | 是否抓住异常，如果为true则页面中就可以使用异常对象，默认是false |
| import          | 导包 import=“java.util.ArrayList”                            |
| **session**     | 是否创建HttpSession对象，默认是true                          |
| buffer          | 设定JspWriter输出jsp内容缓存的大小，默认8kb                  |
| pageEncoding    | 翻译jsp时所用的编码格式                                      |
| **isELIgnored** | 是否忽略EL表达式，默认是false                                |

##### include 指令：

- ##### 可以包含其他页面

  > <%@ include file=包含的页面%>



##### taglib 指令：

- 可以引入外部标签库

  > <%@ taglib uri=标签库的地址prefix=前缀名称%>



##### 示例代码

###### includ.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>被包含的页面</title>
</head>
<body>
    <% String s = "Hello"; %>
</body>
</html>
```

###### error.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>自定义错误页面</title>
</head>
<body>
    不好意思，出错了~~~
</body>
</html>

```

###### jsp指令.jsp

```jsp
<%--
    1.page指令
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" errorPage="/error.jsp" %>

<%--
    2.include指令
--%>
<%@ include file="/include.jsp"%>
<html>
<head>
    <title>jsp指令</title>
</head>
<body>
    
    <%-- 演示page指令的errorPage --%>
    <%--<% int result = 1 / 0; %>--%>
    
    <%-- 输出includ.jsp 中定义的变量 s --%>
    <%= s     %>
    <% out.println("aa"); %>
    
</body>
</html>

```

#### 3.5 JSP 细节

#####  九大隐式对象 

- 重点：request response  session  application  **pageContext**

- out 字符输出流对象。可以将数据输出到页面上。和response.getWriter()类似

  > response.getWriter()和out.write()的区别：
  >
  > 1）在tomcat服务器真正给客户端做出响应之前，会先找response缓冲区数据，再找out缓冲区数据。
  >
  > 2）response.getWriter()数据输出永远在out.write()之前			

| 隐式对象名称 | 代表实际对象                           |
| ------------ | -------------------------------------- |
| request      | javax.servlet.http.HttpServletRequest  |
| response     | javax.servlet.http.HttpServletResponse |
| session      | javax.servlet.http.HttpSession         |
| application  | javax.servlet.ServletContext           |
| page         | java.lang.Object                       |
| config       | javax.servlet.ServletConfig            |
| exception    | java.lang.Throwable                    |
| out          | javax.servlet.jsp.JspWriter            |
| pageContext  | javax.servlet.jsp.PageContext          |

#####  PageContext 对象 

- 是 JSP 独有的，Servlet 中没有。 

- 是四大域对象之一的页面域对象，还可以操作其他三个域对象中的属性。 

- **还可以获取其他八个隐式对象**。

  > 一般配合EL表达式，在jsp页面中动态获取虚拟目录，如下：
  >
  > ```html
  > <form action="${pageContext.request.contextPath}/addStudentServlet">
  >     
  > </form>    
  > ```
  >
  > 

- 生命周期是随着JSP 的创建而存在，随着JSP 的结束而消失。每个JSP 页面都有一个 PageContext对象。

#### 3.6 MVC 模型（重点）

- M(Model)：模型。

  > 1）数据的业务处理
  >
  > 2）数据的获取，比如从数据库Mysql中获取
  >
  > 3）数据的封装，将数据封装到javabean中 

- V(View)：视图。用于显示数据，动态资源用JSP 页面，静态资源用HTML 页面！ 

- C(Controller)：控制器。用于处理请求和响应，例如Servlet！



### 4. 案例 学生管理系统（重点）

#### 4.1 案例实现-登录功能

1）创建一个 web 项目。 

2）在 web 目录下创建一个 index.jsp。 

3）在页面中获取会话域中的用户名，获取到了就显示添加和查看功能的超链接，没获取到就显示登录功能的超链接。 

4）在 web 目录下创建一个 login.jsp。实现登录页面。 

5）创建 LoginServlet，获取用户名和密码。 

6）如果用户名为空，则重定向到登录页面。 

7）如果不为空，将用户名添加到会话域中，再重定向到首页。



##### login.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>学生登录</title>
</head>
<body>
<form action="/stu/loginStudentServlet" method="get" autocomplete="off">
<%--<form action="/<%=request.getContextPath()%>/loginStudentServlet" method="get" autocomplete="off">--%>
    姓名：<input type="text" name="username"> <br>
    密码：<input type="password" name="password"> <br>
    <button type="submit">登录</button>
</form>
</body>
</html>

```

##### LoginStudentServlet.java

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/*
    学生登录
 */
@WebServlet("/loginStudentServlet")
public class LoginStudentServlet extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.获取用户名和密码
        String username = req.getParameter("username");
        String password = req.getParameter("password");

        //2.判断用户名
        if(username == null || "".equals(username)) {
            //2.1用户名为空 重定向到登录页面
            resp.sendRedirect("/stu/login.jsp");
            return;
        }

        //2.2用户名不为空 将用户名存入会话域中
        req.getSession().setAttribute("username",username);

        //3.重定向到首页index.jsp
        resp.sendRedirect("/stu/index.jsp");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```



#### 4.2 案例实现-添加功能

1）在 web 目录下创建一个 addStudent.jsp，实现添加学生的表单项。 

2）创建 AddStudentServlet，获取学生信息并保存到文件中。 

3）通过定时刷新功能2 秒后跳转到首页。



##### addStudent.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>添加学生</title>
</head>
<body>
<form action="/stu/addStudentServlet" method="get" autocomplete="off">
    学生姓名：<input type="text" name="username"> <br>
    学生年龄：<input type="number" name="age"> <br>
    学生成绩：<input type="number" name="score"> <br>
    <button type="submit">保存</button>
</form>
</body>
</html>

```

##### AddStudentServlet.java

```java
package com.itheima.servlet;

import com.itheima.bean.Student;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;

/*
    实现添加功能
 */
@WebServlet("/addStudentServlet")
public class AddStudentServlet extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.获取表单中的数据
        String username = req.getParameter("username");
        String age = req.getParameter("age");
        String score = req.getParameter("score");

        //2.创建学生对象并赋值
        Student stu = new Student();
        stu.setUsername(username);
        stu.setAge(Integer.parseInt(age));
        stu.setScore(Integer.parseInt(score));

        //3.将学生对象的数据保存到d:\\stu.txt文件中
        BufferedWriter bw = new BufferedWriter(new FileWriter("d:\\stu.txt",true));
        bw.write(stu.getUsername() + "," + stu.getAge() + "," + stu.getScore());
        bw.newLine();
        bw.close();

        //4.通过定时刷新功能响应给浏览器
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().write("添加成功。2秒后自动跳转到首页...");
        resp.setHeader("Refresh","2;URL=/stu/index.jsp");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

#### 4.3 案例实现-查看功能

1）创建 ListStudentServlet，读取文件中的学生信息到集合中。 

2）将集合添加到会话域中。 

3）重定向到 listStudent.jsp 页面上。 

4）在 web 目录下创建一个 listStudent.jsp。 

5）定义表格标签。在表格中获取会话域的集合数据，将数据显示在页面上。



##### listStudent.jsp

```jsp
<%@ page import="com.itheima.bean.Student" %>
<%@ page import="java.util.ArrayList" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>查看学生</title>
</head>
<body>
    <table width="600px" border="1px">
        <tr>
            <th>学生姓名</th>
            <th>学生年龄</th>
            <th>学生成绩</th>
        </tr>


        <%--
            1. out内置对象是 JspWriter 类的实例，该类专门针对JSP 页面做输出
            2. out 的优先级没有response输出流优先级高，会后于response做出响应，
            3. 所以在jsp页面中，我们为了保持样式不会因为输出优先级造成混乱，都使用out对象来做响应。

        --%>
       <%-- <%
            ArrayList<Student> students = (ArrayList<Student>) session.getAttribute("students");
            for(Student stu : students) {

                out.write("<tr align=\"center\">");
                    out.write("<td>"+stu.getUsername()+"</td>");
                    out.write("<td>"+stu.getAge()+"</td>");
                    out.write("<td>"+stu.getScore()+"</td>");
                out.write("</tr>");

            }
        %>--%>

        <%
            ArrayList<Student> students = (ArrayList<Student>) session.getAttribute("students");
            for(Student stu : students) {
        %>
            <tr align="center">
                <td><%=stu.getUsername()%></td>
                <td><%=stu.getAge()%></td>
                <td><%=stu.getScore()%></td>
            </tr>
        <%}%>

    </table>
</body>
</html>

```

##### ListStudentServlet.java

```java
package com.itheima.servlet;

import com.itheima.bean.Student;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;

/*
    实现查看功能
 */
@WebServlet("/listStudentServlet")
public class ListStudentServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.创建字符输入流对象，关联读取的文件
        BufferedReader br = new BufferedReader(new FileReader("d:\\stu.txt"));

        //2.创建集合对象，用于保存Student对象
        ArrayList<Student> list = new ArrayList<>();

        //3.循环读取文件中的数据，将数据封装到Student对象中。再把多个学生对象添加到集合中
        String line;
        while((line = br.readLine()) != null) {
            //张三,23,95
            Student stu = new Student();
            String[] arr = line.split(",");
            stu.setUsername(arr[0]);
            stu.setAge(Integer.parseInt(arr[1]));
            stu.setScore(Integer.parseInt(arr[2]));
            list.add(stu);
        }

        //4.将集合对象存入会话域中
        req.getSession().setAttribute("students",list);

        //5.重定向到学生列表页面
        resp.sendRedirect("/stu/listStudent.jsp");

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

##### index.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>学生管理系统首页</title>
</head>
<body>
    <%--
        获取会话域中的数据
        如果获取到了则显示添加和查看功能的超链接
        如果没获取到则显示登录功能的超链接
    --%>
    <% Object username = session.getAttribute("username");
        if(username == null) {
    %>
        <a href="/stu/login.jsp">请登录</a>
    <%} else {%>
        <a href="/stu/addStudent.jsp">添加学生</a>
        <a href="/stu/listStudentServlet">查看学生</a>
    <%}%>
</body>
</html>

```



#### 5. 三层架构

- 三层架构分为：

  > 表示层：页面渲染 + 控制器
  >
  > 业务逻辑层：针对不同的业务需求，编写逻辑代码
  >
  > 数据访问层：用来访问数据库mysql

- 回顾MVC：

  > 1）V：jsp/html 页面渲染
  >
  > 2）C：Servlet，处理请求和响应
  >
  > 3）M：数据的业务处理，数据的获取（数据库：mysql）， 数据的封装javabean

- MVC思想之上，我们可以引入三层架构

  > 1）表示层：V（视图）只做数据展示； C（控制器）只接受请求和响应，不做业务逻辑处理。
  >
  > 2）业务逻辑层service：M（模型）只做业务逻辑
  >
  > 3）数据访问层dao：M（模型）只访问数据库，不做业务处理。

- 三者关系：

  1）Servlet 中创建 service 对象调用其方法

  2）Service中创建dao的对象调用其方法

  3）有返回值则顺次返回