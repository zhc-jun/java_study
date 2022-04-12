# Servlet

### 今日重点

- 基于注解完成综合案例-学生管理系统
- ServletContext对象创建+全局参数+域功能+文件真实路径涉及到的方法

- 了解：线程安全问题的产生，以及如何避免。
- 了解：servlet创建时机，通过配置修改时机。
- 了解：servlet url-partten的配置方式和注意事项



### 1. Servlet

#### 1.1 Servlet 介绍

- Servlet 是运行在 Java 服务器端的程序，用于接收和响应来自客户端基于HTTP 协议的请求。 
- 如果想实现 Servlet 的功能，可以通过实现javax.servlet.Servlet接口或者继承它的实现类。 
- 核心方法：service()，任何客户端的请求都会经过该方法。 

#### 1.2 Servlet 快速入门

1）创建一个 WEB 项目。 

2）创建一个类继承GenericServlet。 

3）重写 service 方法。 

4）在 web.xml 中配置 Servlet。 

5）部署并启动项目。 

6）通过浏览器测试。

##### ServletDemo01.java

```java
package com.itheima.servlet;

import javax.servlet.GenericServlet;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import java.io.IOException;

/*
    Servlet快速入门1
 */
public class ServletDemo01 extends GenericServlet{

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("service方法执行了...");
    }
}

```

#####  web.xml

- 注意: <url-pattern>/servletDemo01</url-pattern> 中的 / 必须要有。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <!--配置快速入门1Servlet-->
    <servlet>
        <servlet-name>servletDemo01</servlet-name>
        <servlet-class>com.itheima.servlet.ServletDemo01</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>servletDemo01</servlet-name>
        <url-pattern>/servletDemo01</url-pattern>
    </servlet-mapping>
</web-app>
```



1.客户端发送请求给服务器

2.输入URL/虚拟目录/资源路径-- 不同虚拟目录名称用来区分Tomcat上发布的不同Web项目

解析,获取虚拟目录对应的Web项目

3.访问Web项目,解析得到对应的资源路径url-pattern

4.从当前web项目内获取web.xml,配置文件内部存在资源路径(用户要获取的东西)

5.找到对应资源,之后通过service()方法进行响应



#### 1.3 Servlet 实现方式

第一种 

- 实现 Servlet 接口，实现所有的抽象方法。该方式支持最大程度的自定义。 

第二种 

- 继承 GenericServlet 抽象类，必须重写 service 方法，其他方法可选择重写。该方式让我们开发 Servlet 变得简单。 但是这种方式和HTTP 协议无关。

<font color=red>第三种 （重点）</font>

- 继承 HttpServlet 抽象类，需要重写doGet 和 doPost 方法。该方式表示请求和响应都需要和HTTP 协议相关。

##### HttpServlet 快速入门 （重点）

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/*
    Servlet快速入门2
 */
public class ServletDemo02 extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("方法执行了...");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <!--配置快速入门2Servlet-->
    <servlet>
        <servlet-name>servletDemo02</servlet-name>
        <servlet-class>com.itheima.servlet.ServletDemo02</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>servletDemo02</servlet-name>
        <url-pattern>/servletDemo02</url-pattern>
    </servlet-mapping>

</web-app>
```



#### 1.4 Servlet 生命周期

- 对象的生命周期，就是对象从出生到死亡的过程。即：出生-> 活着 -> 死亡。官方说法是对象创建到销毁的过程！ 

- 出生：请求第一次到达Servlet 时，对象就创建出来，并且初始化成功。只出生一次，将对象放到内存中。 

- 活着：服务器提供服务的整个过程中，该对象一直存在，每次浏览器对该servlet发起请求，都是执行service 方法。 

- 死亡：当服务停止时，或者服务器宕机时，对象死亡。

  

<font color=red>注意：Servlet 对象只会创建一次，销毁一次。所以Servlet 对象只有一个实例。</font>PS：如果一个对象实例在应用中是唯一的 存在，那么我们就称它为单例模式。



#### 1.5 Servlet 线程安全问题

- 由于 Servlet 采用的是单例模式，只会创建一个对象，该对象在多个线程之间是线程共享的。所以在定义该对象的属性时，我们要考虑线程安全。

- 解决：

  1）如果定义的是属性，多个线程都是只读，没有重新赋值操作的话，不用考虑线程安全，存在多个线程对该属性有写操作，要考虑线程安全。

  2）方法内定义的变量都是线程私有的，可以将变量 定义到 doGet 或 doPost 方法内

  3）也可以加锁，实现同步功能即可，不推荐，效率太低。
  
  

#### 1.6 Servlet 映射方式 （重点）

<font color=red>第一种 (推荐)</font>

- 具体名称的方式。访问的资源路径必须和映射配置完全相同。 

第二种（了解） 

- / 开头 + 通配符的方式。只要符合目录结构即可，不用考虑结尾是什么。 

第三种 

- 通配符 + 固定格式结尾的方式。只要符合固定结尾格式即可，不用考虑前面的路径。

<font color=red>注意：优先级问题。越是具体的优先级越高，越是模糊通用的优先级越低。第一种-> 第二种 -> 第三种</font>



#### 1.7 Servlet 多路径映射（了解即可）

- 我们可以给一个Servlet 配置多个访问映射，从而根据不同的请求路径来实现不同的功能。 

- 场景分析： 

  1）如果访问的资源路径是/vip 商品价格打9折。 

  2）如果访问的资源路径是/vvip 商品价格打5折。 

  3）如果访问的资源路径是其他 商品价格不打折。

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
/*
    Servlet 多路径映射
 */
public class ServletDemo06 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        int money = 1000;

        //获取访问的资源路径
        String name = req.getRequestURI();
        name = name.substring(name.lastIndexOf("/"));

        if("/vip".equals(name)) {
            //如果访问资源路径是/vip 商品价格为9折
            System.out.println("商品原价为：" + money + "。优惠后是：" + (money*0.9));
        } else if("/vvip".equals(name)) {
            //如果访问资源路径是/vvip 商品价格为5折
            System.out.println("商品原价为：" + money + "。优惠后是：" + (money*0.5));
        } else {
            //如果访问资源路径是其他  商品价格原样显示
            System.out.println("商品价格为：" + money);
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <!--演示Servlet多路径映射-->
    <servlet>
        <servlet-name>vip</servlet-name>
        <servlet-class>com.itheima.servlet.ServletDemo06</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>vip</servlet-name>
        <url-pattern>/vip</url-pattern>
    </servlet-mapping>
    <servlet>
        <servlet-name>vvip</servlet-name>
        <servlet-class>com.itheima.servlet.ServletDemo06</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>vvip</servlet-name>
        <url-pattern>/vvip</url-pattern>
    </servlet-mapping>
    <servlet>
        <servlet-name>other</servlet-name>
        <servlet-class>com.itheima.servlet.ServletDemo06</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>other</servlet-name>
        <url-pattern>/other</url-pattern>
    </servlet-mapping>
</web-app>
```

#### 1.8 Servlet 创建时机 （了解）

1）第一次访问时创建 

- 优势：减少对服务器内存的浪费。提高了服务器启动的效率。 
- 弊端：如果有一些要在应用加载时就做的初始化操作，无法完成。 

2）服务器加载时创建 

- 优势：提前创建好对象，提高了首次执行的效率。可以完成一些应用加载时要做的初始化操作。 
- 弊端：对服务器内存占用较多，影响了服务器启动的效率。

 3）修改 Servlet 创建时机。在<servlet>标签中，添加<load-on-startup>标签。

- 正整数代表服务器加载时创建，值越小、优先级越高。负整数或不写代表第一次访问时创建。



### 2. ServletConfig （玩一玩即可）

#### 2.1 ServletConfig 配置方式

- 在<servlet>标签中，通过<init-param>标签来配置。有两个子标签。
-  <param-name>：代表初始化参数的key。
- <param-value>：代表初始化参数的value。

#### 2.2 ServletConfig 实现步骤

1）定义一个类，继承HttpServlet。 

2）在成员位置，声明一个ServletConfig对象。 

3）重写 init 方法，并为ServletConfig 对象赋值。 

4）重写 doGet 和 doPost 方法。 

5）在 web.xml 文件中进行配置。 

6）在 doGet 方法中获取初始化参数。 

7）部署并启动项目。 

8）通过浏览器测试。

```java
package com.itheima.servlet;

import javax.servlet.ServletConfig;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Enumeration;

/*
    ServletConfig的使用
 */
public class ServletConfigDemo extends HttpServlet {

    //声明ServletConfig配置对象
    private ServletConfig config;

    /*
        通过init方法来为ServletConfig配置对象赋值
     */
    @Override
    public void init(ServletConfig config) throws ServletException {
        this.config = config;
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //根据key获取value
        String encodingValue = config.getInitParameter("encoding");
        System.out.println(encodingValue);

        //获取Servlet的名称
        String servletName = config.getServletName();
        System.out.println(servletName);

        //获取所有的key
        Enumeration<String> names = config.getInitParameterNames();
        //遍历得到的key
        while(names.hasMoreElements()) {
            //获取每一个key
            String name = names.nextElement();
            //通过key获取value
            String value = config.getInitParameter(name);
            System.out.println("name:" + name + ",value:" + value);
        }

        //获取ServletContext对象
        ServletContext context = config.getServletContext();
        System.out.println(context);

        //获取ServletContextDemo设置共享的数据
        Object username = context.getAttribute("username");
        System.out.println(username);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <!--配置Servlet-->
    <servlet>
        <servlet-name>servletConfigDemo</servlet-name>
        <servlet-class>com.itheima.servlet.ServletConfigDemo</servlet-class>
        <!--配置ServletConfig初始化参数-->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>desc</param-name>
            <param-value>This is ServletConfig</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>servletConfigDemo</servlet-name>
        <url-pattern>/servletConfigDemo</url-pattern>
    </servlet-mapping>

</web-app>
```



### 3. ServletContext（重点）

- 域功能方法
- 虚拟目录方法
- 磁盘绝对路径

#### 3.1 ServletContext 介绍

- ServletContext 是应用上下文对象。每一个应用中只有一个ServletContext 对象。 
- 作用：可以获得应用的全局初始化参数和达到Servlet 之间的数据共享。 
- 生命周期：应用一加载则创建，应用被停止则销毁。
- 该对象获取：request对象 和  servlet对象都可以获取

**常用方法**

| 返回值              | 方法名                                 | 说明                         |
| ------------------- | -------------------------------------- | ---------------------------- |
| void                | setAttribute(String key, Object value) | 往域中存储数据               |
| Object              | getAttribute(String key )              | 根据key从域中取value         |
| void                | removeAttribute(String key )           | 根据key从域中移除value       |
| String              | getContextPath()                       | 获取虚拟目录                 |
| String              | getRealPath()                          | 磁盘绝对路径，多用于下载功能 |
| String              | getInitParameter(String name)          | 根据名称获取全局配置的值     |
| Enumeration<String> | getInitParamterNames()                 | 获取全局配置的所有名称       |



#### 3.3 域对象

- 域对象指的是对象有作用域。也就是有作用范围。域对象可以实现数据的共享。不同作用范围的域对 象，共享数据的能力也不一样。 
- 在 Servlet 规范中，一共有4 个域对象。ServletContext就是其中的一个。它也是web 应用中最大 的作用域，大家习惯称为application 域。它可以实现整个应用之间的数据共享！

```java
package com.itheima.servlet;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class ServletContextDemo extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        
        //获取ServletContext对象
        ServletContext context = this.getServletContext();

        //向域对象中存储数据
        context.setAttribute("username","zhangsan");

        //移除域对象中username的数据
        context.removeAttribute("username");

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```



#### 3.4 虚拟目录&磁盘绝对路径&全局参数

- 动态获得虚拟目录可以用在重定向功能中，后续会讲重定向。
- 磁盘绝对路径获取，多用于下载功能案例中。
- 全局参数可以在案例中做一些配置项

```java
package com.itheima.servlet;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class ServletContextDemo extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //获取ServletContext对象
        ServletContext context = getServletContext();

        //获取应用的访问虚拟目录
        String contextPath = context.getContextPath();
        System.out.println(contextPath);

        //根据虚拟目录获取应用部署的磁盘绝对路径
        String realPath = context.getRealPath("/");
        System.out.println(realPath);
        
        //获取全局配置的globalEncoding
     String value = context.getInitParameter("globalEncoding");
        System.out.println(value);
        
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <!--配置ServletContext-->
    <context-param>
        <param-name>globalEncoding</param-name>
        <param-value>UTF-8</param-value>
    </context-param>
    <context-param>
        <param-name>globalDesc</param-name>
        <param-value>This is ServletContext</param-value>
    </context-param>
  
</web-app>
```



### 4. 注解开发 Servlet

#### 4.1 Servlet3.0 规范

- 我们使用的是 Tomcat 9 版本。JavaEE 规范要求是 8 。对应的 Servlet 版本应该是4.x 版本。但是， 在企业开发中，稳定要远比追求新版本要重要。所以我们会降版本使用，用的是Servlet 3.1 版本。 
- 其实我们之前的操作全都是基于Servlet 2.5 版本规范的，也就是借助于配置文件的方式。后来随着软 件开发逐步的演变，基于注解的配置开始流行。而Servlet 3.0 版本也就开始支持注解开发了！ 
- Servlet 3.0 版本既保留了2.5 版本的配置方式，同时又支持了全新的注解配置方式。它可以完全不需 要 web.xml 配置文件，就能实现Servlet 的配置，同时还有一些其他的新特性，我们在后面的课程中 会慢慢学习到。



#### 4.1 注解实现Servelet（重点）

1）创建一个 web 项目。 

2）定义一个类，继承HttpServlet。 

3）重写 doGet 和 doPost 方法。 

4）在类上使用 @WebServlet 注解配置 Servlet。 

5）部署并启动项目。 

6）通过浏览器测试。

##### 示例代码

```java 
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/*
    自动注解配置Servlet
 */
@WebServlet("/servletDemo1")
public class ServletDemo1 extends HttpServlet{

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("servlet执行了...");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```



#### 4.2 手动创建容器的实现步骤 （忘掉吧）

............





### 5. 案例 学生管理系统（重点）

#### 5.1 实现步骤

1）创建一个 web 项目。 

2）创建一个用于保存学生信息的html 文件。 

3）创建一个类，继承HttpServlet。 

4）重写 doGet 和 doPost 方法。 

5）在 web.xml 文件中修改默认主页和配置Servlet。 

6）在 doGet 方法中接收表单数据保存到文件中，并响应给浏览器结果。 

7）部署并启动项目。 

8）通过浏览器测试。

##### StudentServlet.java

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/studentServlet")
public class StudentServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //获取表单中的数据
        String username = req.getParameter("username");
        String age = req.getParameter("age");
        String score = req.getParameter("score");

        //将数据保存到stu.txt文件中
        BufferedWriter bw = new BufferedWriter(new FileWriter("d:\\stu.txt",true));
        bw.write(username + "," + age + "," + score);
        bw.newLine();
        bw.close();

        //给浏览器回应
        PrintWriter pw = resp.getWriter();
        pw.println("Save Success~");
        pw.close();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

