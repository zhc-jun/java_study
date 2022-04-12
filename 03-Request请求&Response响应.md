# Request请求&Response响应

### 今日重点

- 知道request对象都有哪些功能，每个功能背后的方法

  > 1) 获取请求报文中数据，获取请求参数的方法最为重要
  >
  > 2) 域数据的共享
  >
  > 3) 请求转发
  >
  > 4) 中文乱码

- 知道response对象都有哪些功能，每个功能背后的方法

  > 1) response设置响应体两个流对象以及方法
  >
  > 2) 常见的设置响应头，content-type /  Content-Disposition 等

- 准确吸收下载案例，知道每行代码的作用

  > 1) 获取文件路径
  >
  > 2) 获得文件流对象
  >
  > 3) 使用response输出流将文件响应给浏览器
  >
  > 4) 设置响应头，告知浏览器用附件的形式处理，不要去解析展示。

- 能够知道BeanUtils.populater()反射原理，并在使用时注意 Map 中的key 在javabean中 要有对应的setXX() 方法。

- 完成学生管理系统



### 1. 请求对象 （重点）

#### 1.1 请求对象介绍

- 请求对象是指

  > doGet(HttpServletRequest request, HttpServletResponse response)方法中的request对象。

- 该对象的功能是：可以通过调用方法来接收客户端浏览器发来的数据，**所有的数据都在浏览器发送的请求报文中。**

#### 1.2 Request获取浏览器发来数据

- 浏览器作为客户端要先发起请求给服务器，浏览器传输给服务器的所有数据，都在HTTP请求报文中，整个报文分为四个结构：请求行，请求头，请求空行，请求体。

##### 1.2.1 Request获取请求路径

- 重点记住：getContextPath() 

| 返回值       | 方法名           | 说明                |
| ------------ | ---------------- | ------------------- |
| String       | getContextPath() | 获取虚拟目录名称    |
| String       | getServletPath() | 获取Servlet映射路径 |
| String       | getRemoteAddr()  | 获取访问者ip地址    |
| String       | getQueryString() | 获取请求的消息数据  |
| String       | getRequestURI()  | 获取统一资源标识符  |
| StringBuffer | getRequestURL()  | 获取统一资源定位符  |

```java
URI: /虚拟目录/资源路径 从当前web项目下去找
URL: 服务器地址/虚拟目录/资源路径 从指定服务器上发布的指定的web项目下去找



package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/*
    获取路径的相关方法
 */
@WebServlet("/servletDemo01")
public class ServletDemo01 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.获取虚拟目录名称 getContextPath()
        String contextPath = req.getContextPath();
        System.out.println(contextPath);

        //2.获取Servlet映射路径 getServletPath()
        String servletPath = req.getServletPath();
        System.out.println(servletPath);

        //3.获取访问者ip getRemoteAddr()
        String ip = req.getRemoteAddr();
        System.out.println(ip);

        //4.获取请求消息的数据 getQueryString()
        String queryString = req.getQueryString();
        System.out.println(queryString);

        //5.获取统一资源标识符 getRequestURI()    /request/servletDemo01   共和国
        String requestURI = req.getRequestURI();
        System.out.println(requestURI);

        //6.获取统一资源定位符 getRequestURL()    http://localhost:8080/request/servletDemo01  中华人民共和国
        StringBuffer requestURL = req.getRequestURL();
        System.out.println(requestURL);

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

##### 1.2.2 Request获取请求头数据

- 记住该方法即可：getHeader(String name) 

| 返回值              | 方法名                  | 说明                     |
| ------------------- | ----------------------- | ------------------------ |
| String              | getHeader(String name)  | 根据请求头名称获取一个值 |
| Enumeration<String> | getHeaders(String name) | 根据请求头名称获取多个值 |
| Enumeration<String> | getHeaderNames()        | 获取所有请求头名称       |

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Enumeration;

/*
    获取请求头信息的相关方法
 */
@WebServlet("/servletDemo02")
public class ServletDemo02 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.根据请求头名称获取一个值
        String connection = req.getHeader("connection");
        System.out.println(connection);
        System.out.println("--------------");

        //2.根据请求头名称获取多个值
        Enumeration<String> values = req.getHeaders("accept-encoding");
        while(values.hasMoreElements()) {
            String value = values.nextElement();
            System.out.println(value);
        }
        System.out.println("--------------");

        //3.获取所有的请求头名称
        Enumeration<String> names = req.getHeaderNames();
        while(names.hasMoreElements()) {
            String name = names.nextElement();
            String value = req.getHeader(name);
            System.out.println(name + "," + value);
        }

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

##### 1.2.3 Request获取请求参数（重点）  ?username=xxx    请求体

- 如下三个方法重点学习。

- getParameter(String name) ，该方法使用率最高， name参数为：input 标签name属性的value。
- getParameterValues(String name)  多用于获取input标签，CheckBox复选框值
- getParameterMap() 特点是获取key,value。可以配合BeanUtils工具类，简化开发。

| 返回值               | 方法名                          | 说明                 |
| -------------------- | ------------------------------- | -------------------- |
| String               | getParameter(String name)       | 根据名称获取数据     |
| String[]             | getParameterValues(String name) | 根据名称获取所有数据 |
| Enumeration<String>  | getParameterNames()             | 获取所有名称         |
| Map<String,String[]> | getParameterMap()               | 获取所有参数的键值对 |

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>注册页面</title>
</head>
<body>
    <form action="/request/servletDemo03" method="post" autocomplete="off">
        姓名：<input type="text" name="username"> <br>
        密码：<input type="password" name="password"> <br>
        爱好：<input type="checkbox" name="hobby" value="study">学习
              <input type="checkbox" name="hobby" value="game">游戏 <br>
        <button type="submit">注册</button>
    </form>
</body>
</html>
```

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Enumeration;
import java.util.Map;

/*
    获取请求参数信息的相关方法
 */
@WebServlet("/servletDemo03")
public class ServletDemo03 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.根据名称获取数据   getParameter()
        String username = req.getParameter("username");
        System.out.println(username);
        String password = req.getParameter("password");
        System.out.println(password);
        System.out.println("--------------------");

        //2.根据名称获取所有数据 getParameterValues()
        String[] hobbies = req.getParameterValues("hobby");
        for(String hobby : hobbies) {
            System.out.println(hobby);
        }
        System.out.println("--------------------");

        //3.获取所有名称  getParameterNames()
        Enumeration<String> names = req.getParameterNames();
        while(names.hasMoreElements()) {
            String name = names.nextElement();
            System.out.println(name);
        }
        System.out.println("--------------------");

        //4.获取所有参数的键值对 getParameterMap()
        Map<String, String[]> map = req.getParameterMap();
        for(String key : map.keySet()) {
            String[] values = map.get(key);
            System.out.print(key + ":");
            for(String value : values) {
                System.out.print(value + " ");
            }
            System.out.println();
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

##### 1.2.4 获取数据封装到JavaBean

- 掌握手动封装 和 BeanUtils 封装即可



###### 手动封装方式

```java
package com.itheima.servlet;

import com.itheima.bean.Student;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/*
    封装对象-手动方式
 */
@WebServlet("/servletDemo04")
public class ServletDemo04 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.获取所有的数据
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        String[] hobbies = req.getParameterValues("hobby");

        //2.封装学生对象
        Student stu = new Student(username,password,hobbies);
        
        //3.输出对象
        System.out.println(stu);

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

###### 反射封装方式

```java
package com.itheima.servlet;

import com.itheima.bean.Student;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.beans.PropertyDescriptor;
import java.io.IOException;
import java.lang.reflect.Method;
import java.util.Map;

/*
    封装对象-反射方式
 */
@WebServlet("/servletDemo05")
public class ServletDemo05 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.获取所有的数据
        Map<String, String[]> map = req.getParameterMap();

        //2.封装学生对象
        Student stu = new Student();
        //2.1遍历集合
        for(String name : map.keySet()) {
            String[] value = map.get(name);
            try {
                //2.2获取Student对象的属性描述器
                PropertyDescriptor pd = new PropertyDescriptor(name,stu.getClass());
                //2.3获取对应的setXxx方法
                Method writeMethod = pd.getWriteMethod();
                //2.4执行方法
                if(value.length > 1) {
                    writeMethod.invoke(stu,(Object)value);
                }else {
                    writeMethod.invoke(stu,value);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        //3.输出对象
        System.out.println(stu);

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

###### 工具类封装方式

```java
package com.itheima.servlet;

import com.itheima.bean.Student;
import org.apache.commons.beanutils.BeanUtils;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Map;

/*
    封装对象-工具类方式
 */
@WebServlet("/servletDemo06")
public class ServletDemo06 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.获取所有的数据
        Map<String, String[]> map = req.getParameterMap();

        //2.封装学生对象
        Student stu = new Student();
        try {
            BeanUtils.populate(stu,map);
        } catch (Exception e) {
            e.printStackTrace();
        }

        //3.输出对象
        System.out.println(stu);

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

##### 1.2.5 流对象获取请求信息 （了解即可）

- 切记：流对象获取的是：请求报文中的类似表单类型的数据，其他数据不获取。

- getReader() 使用该方法时，客户端发来的请求方式必须是POST

| 返回值             | 方法名           | 说明           |
| ------------------ | ---------------- | -------------- |
| BufferedReader     | getReader()      | 获取字符输入流 |
| ServletInputStream | getInputStream() | 获取字节输入流 |

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.ServletInputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/*
    流对象获取数据
 */
@WebServlet("/servletDemo07")
public class ServletDemo07 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //字符流(必须是post方式)
        /*BufferedReader br = req.getReader();
        String line;
        while((line = br.readLine()) != null) {
            System.out.println(line);
        }*/
        //br.close();

        //字节流
        ServletInputStream is = req.getInputStream();
        byte[] arr = new byte[1024];
        int len;
        while((len = is.read(arr)) != -1) {
            System.out.println(new String(arr,0,len));
        }
        //is.close();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

##### 1.2.6 中文乱码问题（重点）

- 之所以会乱码，一定是编码和解码不匹配，为什么我设置为UTF-8就可以解决乱码? 在html 页面中有这样的一行代码：<meta charset="UTF-8">，浏览器解析到该字符编码时，在发送数据给tomcat时，就会采用UTF-8进行编码，所以，我们要用request对象设置解码为：UTF-8。
- 本质： 编码和解码使用的字符集不匹配
- 现象：乱码   1、鏍稿績锛堝弻鍏冿級（都支持中文）     2、???????(有编码直接不支持中文)
- 支持中文的编码集：UTF-8、GBK

- GET 方式 没有乱码问题。在Tomcat 8.5 版本后已经解决！ 

  ?username=中文    tomcat8以前也是乱码的。ISO-8859-1(不支持中文)  ????。

- POST 方式 有乱码问题。可以通过setCharacterEncoding() 方法来解决！

  tomcat解析请求体中的数据的时候，用的编码集是ISO-8859-1 。
  
  setCharacterEncoding()可以设置编码集！！
  
  
  
  ```java
  package com.itheima.servlet;
  
  import javax.servlet.ServletException;
  import javax.servlet.annotation.WebServlet;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  /*
      中文乱码
   */
  @WebServlet("/servletDemo08")
  public class ServletDemo08 extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //设置编码格式
          req.setCharacterEncoding("UTF-8");
  
          String username = req.getParameter("username");
          System.out.println(username);
      }
  
      @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doGet(req,resp);
      }
  }
  
  ```
  
  

#### 

|      |      |      |
| ---- | ---- | ---- |
|      |      |      |
|      |      |      |
|      |      |      |



#### 1.3 请求转发（重点）

- 请求转发：客户端的一次请求到达后，发现需要借助其他Servlet 来实现功能。 

- 特点： 

  1）浏览器地址栏不变 

  2）域对象数据可以在转发的多个servlet之间共享 

  3）负责转发的 Servlet 转发前后的响应正文会丢失 

  4）由转发的目的地来响应客户端

| 返回值            | 方法名                            | 说明             |
| ----------------- | --------------------------------- | ---------------- |
| RequestDispatcher | getRequestDispatcher(String name) | 获取请求调度对象 |
| void              | forword(request,  response resp)  | 开发转发         |

##### Request对象域

- request对象域的共享范围，只在浏览本次请求中，多个Servlet之间可以实现域中数据的共享，也是作用范围最小域对象
- ServletContext域对象作用范围是最大的。

| 返回值 | 方法名                                 | 说明                   |
| ------ | -------------------------------------- | ---------------------- |
| void   | setAttribute(String key, Object value) | 往域中存储数据         |
| Object | getAttribute(String key )              | 根据key从域中取value   |
| void   | removeAttribute(String key )           | 根据key从域中移除value |

##### 示例代码

- ServletDemo09往域中存储数据后，转发给ServletDemo10，然后从域中通过key获取value。

```java
package com.itheima.servlet;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/*
    请求转发
 */
@WebServlet("/servletDemo09")
public class ServletDemo09 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //设置共享数据
        req.setAttribute("encoding","gbk");

        //获取请求调度对象
        RequestDispatcher rd = req.getRequestDispatcher("/servletDemo10");
        //实现转发功能
        rd.forward(req,resp);
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
import java.io.IOException;

/*
    请求转发
 */
@WebServlet("/servletDemo10")
public class ServletDemo10 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //获取共享数据
        Object encoding = req.getAttribute("encoding");
        System.out.println(encoding);

        System.out.println("servletDemo10执行了...");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```



### 2. 响应对象（重点）

#### 2.1 响应对象介绍

- response对象就是响应对象，我们可以通过该对象调用方法，对客户端浏览器当前本次请求做出回应，将我们想输出的数据，通过网络响应给浏览器。

**常见状态码**

| 状态码 | 说明                         |
| ------ | ---------------------------- |
| 200    | 成功                         |
| 302    | 重定向                       |
| 304    | 请求资源未改变，使用缓存     |
| 400    | 请求错误，常见于请求参数错误 |
| 404    | 请求资源未找到               |
| 405    | 请求方式不支持               |
| 500    | 服务器错误                   |

#### 2.2 response对象设置响应报文

- response对象调用的方法都是在设置响应报文中的数据
- 回顾：响应报文组成为：响应头，响应行，响应体。



##### 2.2.1 设置响应体数据

- 设置响应体数据，存在两个流对象，一个字符流：PrintWriter ，一个字节流 OutputStream。
-  响应字符数据，推荐使用字符流-PrintWriter，下载案例推荐使用字节输入流-OutputStream。
- setContentType() 该方法是在**设置响应头**，有两个功能：告诉tomcat用什么编码方式去处理流，同时响应给浏览器，告诉它用什么“解析引擎”去解析数据，并告诉它解码方式。

| 返回值              | 方法名                                     | 说明                                |
| ------------------- | ------------------------------------------ | ----------------------------------- |
| ServletOutputStream | getOutputStream()                          | 获取响应字节输出流对象              |
| PrintWriter         | getWriter()                                | 获取响应字符输出流对象              |
| void                | setContentType(“text/html;charset =UTF-8”) | 设置响应内容类型，解决 中文乱码问题 |

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/*
    字节流响应消息
 */
@WebServlet("/servletDemo01")
public class ServletDemo01 extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");

        //1.获取字节输出流对象
        ServletOutputStream os = resp.getOutputStream();

        //2.定义一个消息
        String str = "你好";

        //3.通过字节流对象输出
        os.write(str.getBytes("UTF-8"));

        //os.close();
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
import java.io.IOException;
import java.io.PrintWriter;

/*
    字符流响应消息
 */
@WebServlet("/servletDemo02")
public class ServletDemo02 extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");//底层执行了下面的代码
        // resp.setCharacterEncoding("UTF-8");  设置字符流缓冲区的编码集，默认是ISO-8859-1
        //1.获取字符输出流对象
        PrintWriter pw = resp.getWriter();

        //2.准备一个消息
        String str = "你好";
        
        
        resp.setContentType("text/html;charset=UTF-8");
        //3.通过字符流输出
        pw.write(str);
        

        //pw.close();
    }

}

```

###### 响应图片给浏览器

1）创建字节输入流对象，关联读取的图片路径。 

2）通过响应对象获取字节输出流对象。 

3）循环读取和写出图片。

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.IOException;

/*
    响应图片
 */
@WebServlet("/servletDemo03")
public class ServletDemo03 extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //通过文件的相对路径获取绝对路径
        String realPath = getServletContext().getRealPath("/img/hm.png");
        System.out.println(realPath);
        //1.创建字节输入流对象，关联图片路径
        BufferedInputStream bis = new BufferedInputStream(new FileInputStream(realPath));

        //2.通过响应对象获取字节输出流对象
        ServletOutputStream os = resp.getOutputStream();

        //3.循环读写
        byte[] arr = new byte[1024];
        int len;
        while((len = bis.read(arr)) != -1) {
            os.write(arr,0,len);
        }

        //4.释放资源
        bis.close();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

##### 2.2.2 设置响应头数据

###### 设置缓存

- 缓存：对于不经常变化的数据，我们可以设置合理缓存时间，以避免浏览器频繁请求服务器。以此来提高效率！

| 返回值 | 方法名                               | 说明               |
| ------ | ------------------------------------ | ------------------ |
| void   | setDateHeader(String name,long time) | 设置消息头添加缓存 |

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/*
    设置缓存
 */
@WebServlet("/servletDemo04")
public class ServletDemo04 extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String news = "这是一条很火爆的新闻！";

        //设置缓存  1小时的缓存时间
        resp.setDateHeader("Expires",System.currentTimeMillis() + 1*60*60*1000);

        //设置编码格式
        resp.setContentType("text/html;charset=UTF-8");
        //写出数据
        resp.getWriter().write(news);

        System.out.println("aaa");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

###### 定时刷新

- 定时刷新：过了指定时间后，页面自动进行跳转

| 返回值 | 方法名                              | 说明               |
| ------ | ----------------------------------- | ------------------ |
| void   | setHeader(String name,String value) | 设置消息头定时刷新 |

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/*
    定时刷新
 */
@WebServlet("/servletDemo05")
public class ServletDemo05 extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String str = "您的用户名或密码有误，3秒后自动跳转到登录页面~~~";

        //设置编码格式
        resp.setContentType("text/html;charset=UTF-8");
        //写出数据
        resp.getWriter().write(str);

        //定时刷新
        resp.setHeader("Refresh","3;URL=/response/login.html");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

###### 请求重定向（重点）

- 请求重定向：客户端的一次请求到达后，发现需要借助其他Servlet 来实现功能。 
- 特点：浏览器地址栏发生改变

| 返回值 | 方法名                    | 说明       |
| ------ | ------------------------- | ---------- |
| void   | sendRedirect(String name) | 设置重定向 |

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/*
    请求重定向
 */
@WebServlet("/servletDemo06")
public class ServletDemo06 extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //设置重定向
        resp.sendRedirect("/response/servletDemo07");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

###### 重定向&请求转发对比

```java
/*
相同之处:
      都是资源的跳转

不同之处：
     1) 转发是tomcat完成资源的跳转, 当前项目下进行跳转, 路径写法上不需要虚拟目录。
     2）重定向是浏览器完成资源的跳转, 路径需要虚拟目录
     3）转发浏览器发起了一次请求，地址栏不变
     4）重定向客户端浏览器发起了两次请求，地址栏变化。
     5）转发只能完成服务器内部资源的跳转，重定向可以跳转到其他的服务器资源（当然也能实现内部资源的跳转）。
     6）因为转发是一起请求，所以多个servlet拿到request对象是一个，request的域中数据时可以共享的
     7）因为重定向是两次请求，所以存在多个request对象，多个servlet之间不能实现request域中数据的共享。  
     
使用场景：
     1）转发：多用于内部资源的跳转，如果需要使用request域来在多个资源间共享数据，只能用请求转发。
     2）重定向：多用于外部资源的跳转，比如微信公众号“授权认证”，这种跨服务器之间的资源跳转。
*/     
```



#### 2.3 文件下载（重点）

1）创建字节输入流，关联读取的文件。 

2）设置响应消息头支持的类型。 

3）设置响应消息头以下载方式打开资源。 

4）通过响应对象获取字节输出流对象。 

5）循环读写。 

6）释放资源。

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.IOException;
/*
    文件下载
 */
@WebServlet("/servletDemo08")
public class ServletDemo08 extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.创建字节输入流对象，关联读取的文件
        String realPath = getServletContext().getRealPath("/img/hm.png");
        BufferedInputStream bis = new BufferedInputStream(new FileInputStream(realPath));

        //2.设置响应头支持的类型
        /*
            Content-Type 消息头名称  支持的类型 MIME
            application/octet-stream  消息头参数  应用的类型为字节流
         */
        resp.setContentType("text/html");
        resp.setHeader("Content-Type","application/octet-stream");

        //3.设置响应头以下载方式打开附件  强制浏览器必须下载！！！
        /*
            Content-Disposition  消息头名称  处理的形式
            attachment;filename=hm.png  消息头参数  附件形式进行处理   指定下载文件的名称
            浏览器的默认行为：能打开就打开，不能打开则下载！！
         */
        resp.setHeader("Content-Disposition","attachment;filename=hm.png");

        //4.获取字节输出流对象
        ServletOutputStream os = resp.getOutputStream();

        //5.循环读写
        byte[] arr = new byte[1024];
        int len;
        while((len = bis.read(arr)) != -1) {
            os.write(arr,0,len);
        }

        //6.释放资源
        bis.close();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```



### 3. 案例 学生管理系统（重点）

#### 3.1 案例实现-资源准备

1）创建一个 web 项目。 

2）在 web 目录下创建一个 index.html，包含两个超链接标签(添加学生、查看学生)。 

3）在 web 目录下创建一个 addStudent.html，用于实现添加功能的表单页面。

4）在 src 下创建一个Student 类，用于封装学生信息。

#### 3.2 案例实现-添加功能

1）创建 AddStudentServlet 类。继承 HttpServlet。 

2）重写 doGet 和 doPost 方法。 

3）获取表单中的数据。 

4）将获取到的数据封装成Student 对象。 

5）将 Student 对象中的数据保存到d:\\stu.txt 文件中。 

6）通过定时刷新功能完成对浏览器的响应。

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
        resp.setHeader("Refresh","2;URL=/stu/index.html");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

#### 3.3 案例实现-查看功能

1）创建 ListStudentServlet 类。继承 HttpServlet。 

2）重写 doGet 和 doPost 方法。 

3）通过字符输入流读取d:\\stu.txt 文件中的数据。 

4）将读到的数据封装到Student 对象中。 

5）将多个 Student 对象保存到集合中。 

6）遍历集合，将数据响应到浏览器。

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
import java.io.PrintWriter;
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

        //4.遍历集合，将数据响应给浏览器
        resp.setContentType("text/html;charset=UTF-8");
        //获取输出流对象
        PrintWriter pw = resp.getWriter();
        for(Student s : list) {
            pw.write(s.getUsername() + "," + s.getAge() + "," + s.getScore());
            pw.write("<br>");
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```





### 4. 扩展

#### 4.1 重定向&请求转发

**相同点：**

- 都是根据设置的路径跳转到目标资源

**不同点：**

- 请求转发，浏览器发出一次请求，而重定向浏览器要发出来两次请求。
- 请求转发地址栏没有变化，重定向地址栏有变化。

**使用场景**

- 当业务需用用到request域的时候，必须用请求转发，因为重定向是两次请求，那么tomcat会对每次请求都创建一个全新的request对象，所以不能实现request域的共享。
- 如果只是单纯在java项目下跳转到其他资源，用谁都无所谓。
- 重定向更多的应用在跳转其他网站的应用场景中，比如：开发微信公众号时，授权登录功能。



#### 4.2 下载文件中文名-乱码

- 获取请求头user-agent: 识别出浏览器品牌：谷歌、360、火狐......

- 针对不同的浏览器对中文文件名做出编码处理，这里提供一个工具类：DownLoadUtil，能看懂，拿来直接用即可。

- 将处理后的文件名，设置在响应头：

  > resp.setHeader("content-disposition", "attachment;filename="+ fileName);

##### DownloadServlet.java

```java
package com.itheima.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

@WebServlet("/download")
public class DownloadServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        String fileName = req.getParameter("fileName");

        String path = this.getServletContext().getRealPath(fileName);
//        String path = this.getServletContext().getRealPath("download/" + fileName);

        InputStream in = new FileInputStream(new File(path));

        //解决文件中文名下载时 客户端无法识别问题
        fileName = DownLoadUtil.getFileName(req.getHeader("user-agent"), fileName);

        //设置下载响应头
        resp.setHeader("content-disposition", "attachment;filename="+ fileName);

        //每次从文件中读10KB
        int len = 0;
        byte[] bytes = new byte[1024 * 10];
        while((len = in.read(bytes)) != -1){
            //开始输出出去
            resp.getOutputStream().write(bytes, 0, len);
        }

    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }
}

```

##### DownLoadUtil.java

- 能看懂，拿来直接用即可。

```java
package com.itheima.servlet;

import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.Base64;

/**
 * 解决中文名文件，下载后乱码问题
 */
public class DownLoadUtil {

    public static String getFileName(String agent, String filename) throws UnsupportedEncodingException {
        if (agent.contains("MSIE")) {
            // IE浏览器
            filename = URLEncoder.encode(filename, "utf-8");
            filename = filename.replace("+", " ");
        } else if (agent.contains("Firefox")) {
            // 火狐浏览器

            //java 9和以上版本已经处理掉了 BASE64Encoder类
            /*BASE64Encoder base64Encoder = new BASE64Encoder();
            filename = "=?utf-8?B?" + base64Encoder.encode(filename.getBytes("utf-8")) + "?=";*/

            //java9 版本使用如下base64方式进行编码
            filename = "=?utf-8?B?" + Base64.getEncoder().encodeToString(filename.getBytes("utf-8")) + "?=";

        } else {
            // 其它浏览器
            filename = URLEncoder.encode(filename, "utf-8");
        }
        return filename;
    }
}

```



#### 4.3 验证码案例

1）java代码生成验证码图片

2）使用response字节输出流，将图片二进制流响应给客户端浏览器

3）通过JS-Date对象，来防止浏览器对图片进行缓存

##### YzmServlet.servlet

```java
package com.itheima.servlet;

import javax.imageio.ImageIO;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.awt.image.BufferedImage;
import java.io.IOException;

/**
 * 获取验证码
 * @author ZJW
 */
@WebServlet("/yzm")
public class YzmServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {

        //接收随机的验证码code
        //获取制作好的验证码图片对象
        BufferedImage image = YzmUtil.getBufferedImage(request);

        //将图片响应给客户端
        ImageIO.write(image, "JPEG", response.getOutputStream());

    }
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

}

```

##### YzmUtil.java

- 拿来直接用即可

```java
package com.itheima.servlet;

import javax.servlet.http.HttpServletRequest;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.IOException;
import java.util.Random;

/**
 * 验证码图片生成类
 */
public class YzmUtil{

	/**
	 *  获取图片对象
	 *
	 * @param request request对象 用来获取session对象，将验证码存储到session中
	 *
	 * @throws IOException
	 */
	public static BufferedImage getBufferedImage(HttpServletRequest request) throws IOException {

		int width = 150;
		int height = 50;
		BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);

		Graphics g = image.getGraphics();
		g.setColor(getRandColor(200, 250));
		g.fillRect(0, 0, width, height);
		g.setColor(new Color(102, 102, 102));
		g.drawRect(20, 30, width - 20, height - 30);
		g.setFont(new Font("Times New Roman", Font.PLAIN, 40));
		g.setColor(getRandColor(160, 200));
		Random RANDOM = new Random();
		// // 画随机线
		for (int i = 0; i < 155; i++) {
			int x = 5 + RANDOM.nextInt(width - 10);
			int y = 5 + RANDOM.nextInt(height - 10);
			int xl = RANDOM.nextInt(6) + 5;
			int yl = RANDOM.nextInt(12) + 5;
			g.drawLine(x, y, x + xl, y + yl);
		}
		// // 从另一方向画随机线
		for (int i = 0; i < 70; i++) {
			int x = 5 + RANDOM.nextInt(width - 10);
			int y = 5 + RANDOM.nextInt(height - 10);
			int xl = RANDOM.nextInt(12) + 5;
			int yl = RANDOM.nextInt(6) + 5;
			g.drawLine(x, y, x - xl, y - yl);
		}
		// 生成随机数,并将随机数字转换为字母
		String code = "";
		for (int i = 0; i < 4; i++) {
			int itmp = RANDOM.nextInt(26) + 65;
			char ctmp = (char) itmp;
			code += String.valueOf(ctmp);
			g.setColor(new Color(20 + RANDOM.nextInt(110), 20 + RANDOM.nextInt(110), 20 + RANDOM.nextInt(110)));
			g.drawString(String.valueOf(ctmp), 25 * i + 20, 40);
		}
		g.dispose();

		//接收随机生成的code
		request.getSession().setAttribute("yzmCode", code);
		return image;
	}

	/**
	 * 给定范围获得随机颜色
	 * @param fc
	 * @param bc
	 * @return
	 */
	static Color getRandColor(int fc, int bc) {
		Random random = new Random();
		if (fc > 255){
			fc = 255;
		}
		if (bc > 255){
			bc = 255;
		}
		int r = fc + random.nextInt(bc - fc);
		int g = fc + random.nextInt(bc - fc);
		int b = fc + random.nextInt(bc - fc);
		return new Color(r, g, b);
	}

}

```

##### yzm.html

- 注意要处理浏览器对图片的缓存问题

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>验证码</title>
</head>

<script>
    window.onload = function (ev) {
        var yamImg = document.getElementById("yzm");
        yamImg.onclick = changeImg;
        function changeImg(ev) {
            yamImg.src = "/day15_resp/yzm?date="+ new Date().getTime();
        }
    }
</script>
<body>

<img id="yzm" src="/day15_resp/yzm" style="width: 118px;">

<a id="" href="javascript:void(0);">看不清？换一张</a>

</body>

</html>
```



