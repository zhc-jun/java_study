# EL&JSTL&过滤器&监听器

### 今日目标

- EL获取域中数据
- EL运算符: 比较，逻辑，empty运算符
- JSTL掌握if  choose for , 练习要能看懂，并仿写。
- 过滤器基本使用要会，重点注解配置，细节要能够理解。
- 独立完成 ServletContextListener 的代码实现，并在扩展案例中，使用它解决了什么问题。



### 1. EL 表达式

- EL(Expression Language)：表达式语言。
- 在 JSP 2.0 规范中加入的内容，也是Servlet 规范的一部分。 
- 作用：在 JSP 页面中获取**域对象中的数据**。让我们的JSP 脱离 java 代码块和 JSP 表达式。 
- <font color=red>语法：${ 表达式内容 }</font>

#### 1.1 EL 快速入门

1）创建一个 web 项目。 

2）在 web 目录下创建 el01.jsp。 

3）在文件中向域对象添加数据。 

4）使用三种方式获取域对象中的数据(java 代码块、JSP 表达式、EL 表达式)。 

5）部署并启动项目。 

6）通过浏览器测试。

###### 示例代码

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>EL表达式快速入门</title>
</head>
<body>
    <%--1.向域对象中添加数据--%>
    <% request.setAttribute("username","zhangsan"); %>

    <%--2.获取数据--%>
    Java代码块：<% out.println(request.getAttribute("username")); %> <br>

    JSP表达式：<%= request.getAttribute("username")%> <br>

    EL表达式：${username}
</body>
</html>

```

#### 1.2 EL 从域中获取数据（重点）

- 重点看 1） 2）4）
- 2）从对象中获取数据，底层是调用javabean getXXX() 来获取对应属性值。5

1）获取基本数据类型的数据。 

2）获取自定义对象类型的数据。 

3）获取数组类型的数据。 

4）获取 List 集合类型的数据。 

5）获取 Map 集合类型的数据。

```java
<%@ page import="com.itheima.bean.Student" %>
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.util.HashMap" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>EL表达式获取不同类型数据</title>
</head>
<body>
    <%--1.获取基本数据类型--%>
    <% pageContext.setAttribute("num",10); %>
    基本数据类型：${num} <br>

    <%--2.获取自定义对象类型--%>
    <%
        Student stu = new Student("张三",23);
        stu = null;
        pageContext.setAttribute("stu",stu);
    %>
    <%--EL表达式中没有空指针异常--%>
    自定义对象：${stu} <br>
    <%--stu.name 实现原理 getName()--%>
    学生姓名：${stu.name} <br>
    学生年龄：${stu.age} <br>

    <%--3.获取数组类型--%>
    <%
        String[] arr = {"hello","world"};
        pageContext.setAttribute("arr",arr);
    %>
    数组：${arr}  <br>
    0索引元素：${arr[0]} <br>
    1索引元素：${arr[1]} <br>
    <%--EL表达式中没有索引越界异常--%>
    8索引元素：${arr[8]} <br>
    <%--EL表达式中没有字符串拼接--%>
    0索引拼接1索引的元素：${arr[0]} + ${arr[1]} <br>

    <%--4.获取List集合--%>
    <%
        ArrayList<String> list = new ArrayList<>();
        list.add("aaa");
        list.add("bbb");
        pageContext.setAttribute("list",list);
    %>
    List集合：${list} <br>
    0索引元素：${list[0]} <br>

    <%--5.获取Map集合--%>
    <%
        HashMap<String,Student> map = new HashMap<>();
        map.put("hm01",new Student("张三",23));
        map.put("hm02",new Student("李四",24));
        pageContext.setAttribute("map",map);
    %>
    Map集合：${map}  <br>
    第一个学生对象：${map.hm01}  <br>
    第一个学生对象的姓名：${map.hm01.name}

</body>
</html>

```

##### EL 注意事项

- EL 表达式没有空指针异常。 
- EL 表达式没有索引越界异常。
- EL 表达式没有字符串的拼接。



#### 1.4 EL 运算符（重点）

- 后面案例中用到的运算符记住即可，用不到的可以放一放。

##### 关系运算符

| 运算符   | 作用     | 示例                   | 结果  |
| -------- | -------- | ---------------------- | ----- |
| == 或 eq | 等于     | ${5 == 5} 或 ${5 eq 5} | true  |
| != 或 ne | 不等于   | ${5 != 5} 或 ${5 ne 5} | false |
| < 或 lt  | 小于     | ${3 < 5} 或 ${3 lt 5}  | true  |
| > 或 gt  | 大于     | ${3 > 5} 或 ${3 gt 5}  | false |
| <= 或 le | 小于等于 | ${3 <= 5} 或 ${3 le 5} | true  |
| >= 或 ge | 大于等于 | ${3 >= 5} 或 ${3 ge 5} | false |

##### 逻辑运算符

| 运算符     | 作用 | 示例                     | 结果         |
| ---------- | ---- | ------------------------ | ------------ |
| && 或 and  | 并且 | ${A && B} 或 ${A and B}  | true / false |
| \|\| 或 or | 或者 | ${A \|\| B} 或 ${A or B} | true / false |
| ! 或 not   | 取反 | ${ !A } 或 ${ not A }    | true / false |

##### 其他运算符

| 运算符                   | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| empty                    | 1.判断对象是否为null 2.判断字符串是否为空字符串 3.判断容器元素是否为0 |
| 条件 ? 表达式1 : 表达式2 | 三元运算符                                                   |

##### 示例代码

```jsp
<%@ page import="java.util.List" %>
<%@ page import="java.util.ArrayList" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>EL表达式运算符</title>
</head>
<body>

    <h3>比较运算符</h3>
    ${3 == 4}<br>

    <h3>逻辑运算符</h3>
    ${3 > 4  && 3 < 4}<br>
    ${3 > 4  and 3 < 4}<br>

    <h3>empty 运算符</h3>
    <%
        String str1 = null;
        String str2 = "";
        int[] arr = {};

        List list = new ArrayList();
        request.setAttribute("list",list);
    %>

    期待是空: ${empty str1} <br>
    期待不是空: ${not empty str1}<br>

    <hr>

    期待是空: ${empty str2} <br>
    期待是空: ${empty arr} <br>
    期待是空: ${empty list} <br>


    <h3>三元运算符。获取性别的数据，在对应的按钮上进行勾选</h3>
    <% pageContext.setAttribute("gender","women"); %>
    <input type="radio" name="gender" value="men" ${gender == "men" ? "checked" : ""}>男
    <input type="radio" name="gender" value="women" ${gender == "women" ? "checked" : ""}>女
</body>
</html>

```



#### 1.5 EL 使用细节

- EL 表达式能够获取四大域对象的数据，如果没有指定域对象，将根据名称从小到大在域对象中查找。例如：

  ```jsp
  
  <%-- 没有指定域对象名称，将从小到大遍历 --%>
  <% request.setAttribute("num",10); %>
      基本数据类型：${num} <br>
  
  <%-- 指定域对象名称requestScope, 不会遍历 --%>
  <% request.setAttribute("num",10); %>
      基本数据类型：${requestScope.num} <br>
  ```

  

- 还可以使用pageContext获取 JSP 其他八个隐式对象，并调用对象中的方法。

  ```jsp
  <form action="${pageContext.request.contextPath}/addStudentServlet">
   
  </form>   
  ```



#### 1.6 EL 隐式对象

- pageContext 获取JSP八大隐式对象，比如: 获取request对象, 在调用getContextPath() 方法动态获得虚拟目录，

  ```jsp
  ${pageContext.request.contextPath}
  ```

- requestScope 获取request域中数据。

- sessionScope 获取session域中数据。

- applicationScope 获取ServletContext域中数据。

- .........其他了解即可



### 2. JSTL

#### 2.1 JSTL 介绍

- JSTL(Java Server Pages Standarded Tag Library)：**JSP 标准标签库**。 
- 主要提供给开发人员一个标准通用的标签库。 
- 开发人员可以利用这些标签取代JSP 页面上的 Java 代码，从而提高程序的可读性，降低程序的维护难度。

#### 2.2 JSTL 核心标签库

| 标签名称                                         | 功能分类 | 属性       | 作用           |
| ------------------------------------------------ | -------- | ---------- | -------------- |
| <标签名:if>                                      | 流程控制 | 核心标签库 | 用于条件判断   |
| <标签名:choose> <标签名:when> <标签名:otherwise> | 流程控制 | 核心标签库 | 用于多条件判断 |
| <标签名:forEach>                                 | 迭代遍历 | 核心标签库 | 用于循环遍历   |

#### 代码演示（重点）

###### if  choose

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<html>
<head>
    <title>流程控制</title>
</head>
<body>
    <%--向域对象中添加成绩数据--%>
    ${pageContext.setAttribute("score","T")}

    <%--对成绩进行判断--%>
    <c:if test="${score == 'A'}">
        优秀
    </c:if>

    <%--对成绩进行多条件判断--%>
    <c:choose>
        <c:when test="${score == 'A'}">优秀</c:when>
        <c:when test="${score == 'B'}">良好</c:when>
        <c:when test="${score == 'C'}">及格</c:when>
        <c:when test="${score == 'D'}">较差</c:when>
        <c:otherwise>成绩非法</c:otherwise>
    </c:choose>
</body>
</html>

```

###### for

```jsp
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.util.List" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<html>
<head>
    <title>foreach标签</title>
</head>
<body>

<%--

    foreach:相当于java代码的for语句
        1. 完成重复的操作
            for(int i = 0; i < 10; i ++){

            }
            * 属性：
                begin：开始值
                end：结束值
                var：临时变量
                step：步长

        2. 遍历容器
            List<User> list;
            for(User user : list){

            }

            * 属性：
                items:容器(集合对象)对象
                var:容器中元素的临时变量
                varStatus:循环状态对象
                    index:容器中元素的索引，从0开始
                    count:循环次数，从1开始


--%>

    <%

        for (int i = 1; i <= 10; i++) {

        }

    %>

    <c:forEach var="i" begin="1" end="10" step="1" varStatus="vs">
        变量i的值: ${i} <br>
    </c:forEach>

<hr>


    <%

        List<String> list = new ArrayList();
        list.add("aaa");
        list.add("bbb");
        list.add("ccc");

        request.setAttribute("list", list);

        for (String str : list){


        }

     %>

    <c:forEach items="${requestScope.list}" var="str" varStatus="vs">

        变量str的值: ${str} , 索引下标<b>${vs.index}</b> <br>

    </c:forEach>


</body>
</html>

```

###### 练习

* 需求：在request域中有一个存有User对象的List集合。需要使用jstl+el将list集合数据展示到jsp页面的表格table中

```jsp
<%@ page import="java.util.List" %>
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.util.Date" %>
<%@ page import="com.itheima.cn.User" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<html>
<head>
    <title>test</title>
</head>
<body>

<%

    List list = new ArrayList();
    list.add(new User("张三",23,new Date()));
    list.add(new User("李四",24,new Date()));
    list.add(new User("王五",25,new Date()));

    request.setAttribute("list",list);

%>

<table border="1" width="500" align="center">
    <tr>
        <th>编号</th>
        <th>姓名</th>
        <th>年龄</th>
        <th>生日</th>
    </tr>
    <%--数据行--%>
    <c:forEach items="${list}" var="user" varStatus="s">

        <tr >
            <td>${s.count}</td>
            <td>${user.name}</td>
            <td>${user.age}</td>
            <td>${user.birStr}</td>
        </tr>

    </c:forEach>

</table>

</body>
</html>

```



### 3. Filter

#### 3.1 过滤器介绍

- 在程序中访问服务器资源时，当一个请求到来，服务器会判断当前请求是否满足过滤条件，如果满足， 过滤器会将请求拦截下来，执行doFilter()方法，再由过滤器决定是否交给请求资源（servlet、jsp、html....）。如果不满足过滤条件，则像之前 那样直接请求资源了。响应也是类似的！

- 过滤器一般用于完成通用的操作，例如：登录验证、统一编码处理、敏感字符过滤等等~~~

#### 3.2 Filter 介绍

- Filter 是一个接口。如果想实现过滤器的功能，必须实现该接口！ 

- 核心方法

  | 返回值 | 方法名                                                       | 作用                     |
  | ------ | ------------------------------------------------------------ | ------------------------ |
  | void   | init(FilterConfig config)                                    | 初始化后调用方法         |
  | void   | doFilter(ServletRequest request,ServletResponse response,FilterChain chain) | 对请求资源和响应资源过滤 |
  | void   | destroy()                                                    | 销毁时调用方法           |

- 配置方式 
  - 注解方式 
  - 配置文件



#### 3.3 FilterChain 介绍

- FilterChain 是一个接口，代表过滤器链对象。由Servlet 容器提供实现类对象。直接使用即可。 

- 过滤器可以定义多个，就会组成过滤器链。 

- 核心方法

  | 返回值 | 方法名                                                  | 作用     |
  | ------ | ------------------------------------------------------- | -------- |
  | void   | doFilter(ServletRequestrequest,ServletResponseresponse) | 放行方法 |

- <font color=red>如果有多个过滤器，在第一个过滤器中调用下一个过滤器，依次类推。直到到达最终访问资源。 如果只有一个过滤器，放行时，就会直接到达最终访问资源</font>



#### 3.4 过滤器使用 (重点)

- 需求说明 

  > 通过 Filter 过滤器解决多个资源写出中文乱码的问题。

- 最终目的 

  > 通过本需求，最终掌握Filter 过滤器的使用。 

- 实现步骤 

  1） 创建一个 web 项目。 

  2）创建两个 Servlet 功能类，都向客户端写出中文数据。 

  3）创建一个 Filter 过滤器实现类，重写doFilter 核心方法。 

  4）在方法内解决中文乱码，并放行。 

  5）部署并启动项目。 

  6）通过浏览器测试。

##### 注解配置

```java
package com.itheima.filter;

import javax.servlet.*;
import java.io.IOException;

/*
    过滤器基本使用
 */
@WebFilter("/*")
public class FilterDemo01 implements Filter{

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("filterDemo01执行了...");

        //处理乱码
        servletResponse.setContentType("text/html;charset=UTF-8");

        //放行
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void destroy() {

    }
}

```

##### xml配置

- （注解和xml只需一种方式即可）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    
   <!--配置过滤器-->
   <filter>
     <filter-name>filterDemo01</filter-name>
     <filter-class>com.itheima.filter.FilterDemo01</filter-class>
   </filter>
   <filter-mapping>
     <filter-name>filterDemo01</filter-name>
     <url-pattern>/*</url-pattern>
   </filter-mapping>
    
</web-app>
```



#### 3.5 过滤器使用细节 (重点)

- 配置方式 

  1）注解方式 @WebFilter(拦截路径) 

  2）配置文件方式

  ```xml
  <!--配置过滤器-->
  <filter>
   <filter-name>filterDemo01</filter-name>
   <filter-class>com.itheima.filter.FilterDemo01</filter-class>
  </filter>
  
  <filter-mapping>
   <filter-name>filterDemo01</filter-name>
   <url-pattern>/*</url-pattern>
  </filter-mapping>
  ```

- 多个过滤器执行顺序 ：

  1） xml配置方式，谁配置在上面谁先执行

  2）注解配置方式, 按自然顺序比较，谁大谁先执行，自然顺序: 0-9 , a-z.

  ```html
  <!-- 
      Filter_1.java  要比 Filter_2.java 先执行
      Filter_a.java  要比 Filter_b.java 先执行
      AFilter.java   要比 BFilter.java 先执行
  -->    
  ```



#### 3.6 过滤器生命周期

- 创建 当应用加载时实例化对象并执行init 初始化方法。
- 服务 对象提供服务的过程，执行doFilter 方法。 
- 销毁 当应用卸载时或服务器停止时对象销毁。执行destroy 方法。

#### 3.7 FilterConfig 介绍

- FilterConfig 是一个接口。代表过滤器的配置对象，可以加载一些初始化参数。 
- 核心方法

| 返回值              | 方法名                       | 作用               |
| ------------------- | ---------------------------- | ------------------ |
| String              | getFilterName()              | 获取过滤器对象名称 |
| String              | getInitParameter(String key) | 根据key获取value   |
| Enumeration<String> | getInitParameterNames()      | 获取所有参数的key  |
| ServletContext      | getServletContext()          | 获取应用上下文对象 |

#### 3.8 过滤器五种拦截行为

- 五种行为:

  1）请求时  **REQUEST**

  2）请求转发 **FORWARD**

  3）请求包含 **INCLUDE**

  4）全局错误页面 **ERROR**

  5）异步类型  **ASYNC**

- Filter 过滤器默认是**请求时**拦截，  ，但是在实际开发中，我们还有请求转发和请求包含，以及由服务器触发调 用的全局错误页面。默认情况下过滤器是不参与过滤的，要想使用，就需要我们配置。 
- <font color=red>默认的 **REQUEST**    就够用，其他了解即可</font>

##### 代码演示

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
 
    <!--配置过滤器五种拦截方式-->
    <filter>
        <filter-name>filterDemo05</filter-name>
        <filter-class>com.itheima.filter.FilterDemo05</filter-class>
        <!--配置开启异步支持，当dispatcher配置ASYNC时，需要配置此行-->
        <async-supported>true</async-supported>
    </filter>
    <filter-mapping>
        <filter-name>filterDemo05</filter-name>
        <!--<url-pattern>/error.jsp</url-pattern>-->
        <url-pattern>/index.jsp</url-pattern>
        <!--过滤请求：默认值。-->
        <dispatcher>REQUEST</dispatcher>
        <!--过滤全局错误页面：当由服务器调用全局错误页面时，过滤器工作-->
        <dispatcher>ERROR</dispatcher>
        <!--过滤请求转发：当请求转发时，过滤器工作。-->
        <dispatcher>FORWARD</dispatcher>
        <!--过滤请求包含：当请求包含时，过滤器工作。它只能过滤动态包含，jsp的include指令是静态包含，过滤器不会起作用-->
        <dispatcher>INCLUDE</dispatcher>
        <!--过滤异步类型，它要求我们在filter标签中配置开启异步支持-->
        <dispatcher>ASYNC</dispatcher>
    </filter-mapping>

    <!--配置全局错误页面-->
    <error-page>
        <exception-type>java.lang.Exception</exception-type>
        <location>/error.jsp</location>
    </error-page>
    <error-page>
        <error-code>404</error-code>
        <location>/error.jsp</location>
    </error-page>
    
</web-app>
```



### 4. Listener

#### 4.1 监听器介绍

- 观察者设计模式，所有的监听器都是基于观察者设计模式的！

- 三个组成部分 

  1）事件源：触发事件的对象 

  2）事件：触发的动作，封装了事件源 

  3）监听器：当事件源触发事件后，可以完成功能

- 在程序当中，我们可以对：对象的创建销毁、域对象中属性的变化、会话相关内容进行监听。 
- Servlet 规范中共计 8 个监听器，监听器都是以接口形式提供，具体功能需要我们自己来完成。



#### 4.2 ServletContextListener 监听器

- ServletContextListener：主要用于监听ServletContext 对象的创建和销毁 

- <font color=red>ServletContext </font>

  1）**特点是**：在Tomcat启动过程中，会在内存中创建唯一一个ServletContext对象。

  2）我们可以监听它，这样在tomcat启动过程中，可以完成一些业务上数据加载项。

- 核心方法

| 返回值 | 方法名                                      | 作用                 |
| ------ | ------------------------------------------- | -------------------- |
| void   | contextInitialized(ServletContextEvent sce) | 对象创建时执行该方法 |
| void   | contextDestroyed(ServletContextEvent sce)   | 对象销毁时执行该方法 |

##### 注解实现

```java
package com.itheima.listener;

import javax.servlet.ServletContext;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;

/*
    ServletContext对象的创建和销毁的监听器
 */
@WebListener
public class ServletContextListenerDemo implements ServletContextListener{
    /*
        ServletContext对象创建的时候执行此方法
     */
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("监听到了对象的创建...");

        //获取对象
        ServletContext servletContext = sce.getServletContext();
        //System.out.println(servletContext);

        //添加属性
        servletContext.setAttribute("username","zhangsan");

        //替换属性
        servletContext.setAttribute("username","lisi");

        //移除属性
        servletContext.removeAttribute("username");
    }

    /*
        ServletContext对象销毁的时候执行此方法
     */
    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("监听到了对象的销毁...");
    }
}

```

##### xml实现注册监听

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    
<!--配置监听器-->
<listener>
  <listener-class>包名.ServletContextListenerDemo</listener-class>
</listener>
  
</web-app>
```



### 5. 扩展

#### 5.1 案例 学生管理系统

- 引入Filter 对登录进行拦截，未登录跳转到登录页面。
- 敏感词汇过滤；过滤器 + 监听器
- 演示XSS攻击；增强过滤器 + 监听器完成拦截；