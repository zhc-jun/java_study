# AJAX

### 1. AJAX 快速入门

**本节重点**：能够使用JQuery  $.get()  /  $.post()  /  $.ajax() 三个方法分别给服务器发送 HTTP 协议的 GET 和 POST请求方式。

#### 1.0 在线参考网页

- JavaScript AJAX :  https://www.runoob.com/ajax/ajax-xmlhttprequest-onreadystatechange.html
- jQuery  AJAX: https://www.runoob.com/jquery/jquery-ref-ajax.html



#### 1.1 AJAX 介绍

- AJAX(Asynchronous JavaScript And XML)：<font color=red>异步</font>的JavaScript 和 XML。 
- 本身不是一种新技术，而是多个技术综合。用于快速创建动态网页的技术。
- 一般的网页如果需要更新内容，必需重新加载个页面。
- 而 AJAX 通过浏览器与服务器进行少量数据交换，就可以使网页实现异步更新。也就是在不重新加载整个页 面的情况下，对网页的部分内容进行<font color=red>局部更新。</font>

**综上所述：**AJAX就是一种客户端浏览器可以通过JavaScript代码与服务器tomcat实现异步网络通信的一种技术。

**优点**

1）异步发起请求，用户可以继续操作页面，无需等待响应结果，用户体验更好。

2）ajax可以实现整张页面的局部刷新，这样可以不仅可以提高网络通信的效率，浏览器也不用重新加载整张页面，同样带来更好的用户体验。

**异步和同步**

- 异步：当浏览器发起ajax请求为异步时，用户可以继续操作页面。
- 同步:   当浏览器发起ajax请求为同步时，用户将无法操作页面。
- 在使用ajax时，虽然可以设置为同步，但是我们不用这么做。



#### 1.2 JavaScript实现Ajax (了解)

-  核心对象：XMLHttpRequest 用于在后台与服务器交换数据。

  > 可以在不重新加载整个网页的情况下，对网页的某部分进行更新。 

- 打开链接：open(method,url,async) 

  > method：请求的类型 GET 或 POST。 
  >
  > url：请求资源的路径。 
  >
  > async：true(异步) 或 false(同步)。 

- 发送请求：send(String params) 

  > params：请求的参数(POST 专用)。 GET请求无需提供参数，在URL后？拼接即可。

-  处理响应：onreadystatechange 

  > readyState：0-请求未初始化，1-服务器连接已建立，2-请求已接收，3-请求处理中，4-请求已完成，且响应已就绪。 
  >
  > status：200-响应已全部OK。 

- 获得响应数据形式 

  > responseText：获得字符串形式的响应数据。 
  >
  > responseXML：获得 XML 形式的响应数据。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>用户注册</title>
</head>
<body>
    <form autocomplete="off">
        姓名：<input type="text" id="username">
        <span id="uSpan"></span>
        <br>
        密码：<input type="password" id="password">
        <br>
        <input type="submit" value="注册">
    </form>
</body>
<script>
    //1.为姓名绑定失去焦点事件
    document.getElementById("username").onblur = function() {
        //2.创建XMLHttpRequest核心对象
        let xmlHttp = new XMLHttpRequest();

        //3.打开链接
        let username = document.getElementById("username").value;
        //true异步；false同步
        xmlHttp.open("GET","userServlet?username="+username,true);
        //xmlHttp.open("GET","userServlet?username="+username,false);

        //4.发送请求
        xmlHttp.send();

        //5.处理响应
        xmlHttp.onreadystatechange = function() {
            //判断请求和响应是否成功
            if(xmlHttp.readyState == 4 && xmlHttp.status == 200) {
                //将响应的数据显示到span标签
                document.getElementById("uSpan").innerHTML = xmlHttp.responseText;
            }
        }
    }
</script>
</html>
```



#### 1.3 JQuery 实现 AJAX GET 方式 (重点)

- 核心语法：data  callback type 都是可选参数，按照自己需求设定。

  > $.get(url,[data],[callback],[type]); 
  >
  > url：请求的资源路径。 
  >
  > data：发送给服务器端的请求参数，格式可以是key=value，也可以是js 对象。 callback：当请求成功后的回调函数，可以在函数中编写我们的逻辑代码。 type：预期的返回数据的类型，取值可以是xml, html, js, json, text等。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>ajax</title>
</head>
<body>

<script src="jquery-3.3.1.js"></script>
<script>

    /**
     * 注意事项:
     *
     * 由于$.post() 和 $.get() 默认是 异步请求，如果需要同步请求，则可以进行如下使用：
       在$.post()前把ajax设置为同步：$.ajaxSettings.async = false;
       在$.post()后把ajax改回为异步：$.ajaxSettings.async = true;
     *
     * */
    
    //入口函数
    $(function () {

        $("#btnGet").click(function(){

            //方式一
            /*var url = "/ajax/ajaxServlet?method=GET&msg=send get";
            $.get(url, function(resultData, status){
                alert('状态码:' + status + ", 响应报文: " + resultData);
            });*/

            //方式二
            var url = "/ajax/ajaxServlet";
            var data = {"method": "GET", "msg": "send get"};
            $.get(url, data, function(resultData, status){
                alert('状态码:' + status + ", 响应报文: " + resultData);
            });

         });

    });


</script>

<input type="button" value="发送GET" id="btnGet">

</body>
</html>
```



#### 1.4  JQuery 实现 AJAX GET 方式 (重点)

- 核心语法：data  callback type 都是可选参数，按照自己需求设定。

  > $.get(url,[data],[callback],[type]); 
  >
  > url：请求的资源路径。 
  >
  > data：发送给服务器端的请求参数，格式可以是key=value，也可以是js 对象。 callback：当请求成功后的回调函数，可以在函数中编写我们的逻辑代码。 type：预期的返回数据的类型，取值可以是xml, html, js, json, text等

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>ajax</title>
</head>
<body>

<script src="jquery-3.3.1.js"></script>
<script>

    /**
     * 注意事项:
     *
     * 由于$.post() 和 $.get() 默认是 异步请求，如果需要同步请求，则可以进行如下使用 (不推荐)：
       在$.post()前把ajax设置为同步：$.ajaxSettings.async = false;
       在$.post()后把ajax改回为异步：$.ajaxSettings.async = true;
     *
     * */

    //入口函数
    $(function () {

        $("#btnPost").click(function(){

            var url = "/ajax/ajaxServlet";
            var data = {"method": "POST", "msg": "send post"};

            $.post(url, data, function(resultData, status){
                alert('状态码:' + status + ", 响应报文: " + resultData);
            });
        });

    });


</script>

<input type="button" value="发送POST" id="btnPost">

</body>
</html>
```



#### 1.5 JQuery 实现 AJAX 通用请求方式 (重点)

- 核心语法：

  > $.ajax({name:value,name:value,…}); 
  >
  > url：请求的资源路径。 
  >
  > async：是否异步请求，true-是，false-否(默认是 true)。 
  >
  > data：发送到服务器的数据，可以是键值对形式，也可以是js 对象形式。 type：请求方式，POST 或 GET (默认是 GET)。 
  >
  > dataType：预期的返回数据的类型，取值可以是xml, html, js, json, text等。 success：请求成功时调用的回调函数。 
  >
  > error：请求失败时调用的回调函数。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>ajax</title>
</head>
<body>

<script src="jquery-3.3.1.js"></script>
<script>

    /**
     * 注意事项:
     *   
     * data 的两种格式:
     *     data: {"method": "PUT", "msg": "send put"},
     *     data: "method=GET&msg=send get",
     *
     */

    //入口函数
    $(function () {

        $("#btnAjax").click(function(){

            var url = "/ajax/ajaxServlet";
            
            $.ajax({
                Url: url,
                type: 'GET',
                //data: {"method": "PUT", "msg": "send put"},
                data: "method=GET&msg=send get",
                
                // async : true, // 默认true 为异步请求，false 为同步请求
                dataType:"text", //设置接受到的响应数据的格式
                success:function (resultData, status) { 
                    //响应成功后的回调函数
                    alert('状态码:'+status+", 响应报文:" + resultData);
                },
                error:function (data) { 
                    //表示如果请求响应出现错误，会执行的回调函数
                    alert("出错啦..." + data)
                }

            });

        });

    });


</script>

<input type="button" value="通用请求方式" id="btnAjax">

</body>
</html>
```



#### 1.6 AJAX 快速入门小结

- AJAX(Asynchronous JavaScript And XML)：异步的JavaScript 和 XML。

- 通过浏览器与服务器进行少量数据交换，就可以使网页实现异步更新。也就是在不重新加载整个页面的情况下，对网页的部 分内容进行局部更新。 

- 同步和异步 

  >  同步：服务器端在处理过程中，无法进行其他操作。 
  >
  > 异步：服务器端在处理过程中，可以进行其他操作。 

- GET 方式实现：$.get(); 

- POST 方式实现：$.post(); 

  > url：请求的资源路径。 
  >
  > data(可选参数)：发送给服务器端的请求参数，格式可以是key=value，也可以是js 对象。 
  >
  > callback(可选参数)：当请求成功后的回调函数，可以在函数中编写我们的逻辑代码。 
  >
  > type(可选参数)：预期的返回数据的类型，取值可以是xml, html, js, json, text等。

- 通用方式实现：$.ajax(); 

  > url：请求的资源路径。 
  >
  > async：是否异步请求，true-是，false-否(默认是 true)。 
  >
  > data：发送到服务器的数据，可以是键值对形式，也可以是js 对象形式。 type：请求方式，POST 或 GET (默认是 GET)。 
  >
  > dataType：预期的返回数据的类型，取值可以是xml, html, js, json, text等。 success：请求成功时调用的回调函数。 
  >
  > error：请求失败时调用的回调函数。



### 2. JSON 的处理

#### 2.1 JavaScript 中 JSON 回顾

- JSON(JavaScript Object Notation)：是一种轻量级的数据交换格式。 
- 它是基于 ECMAScript 规范的一个子集，采用完全独立于编程语言的文本格式来存储和表示数据。
- 简洁和清晰的层次结构使得JSON 成为理想的数据交换语言。易于人阅读和编写，同时也易于计算机解析和生成，并有效的 提升网络传输效率。

**JS 中创建格式**

| 类型           | 语法                                         | 说明                                                         |
| -------------- | -------------------------------------------- | ------------------------------------------------------------ |
| 对象类型       | {name:value,name:value,...}                  |                                                              |
| 数组/集合类 型 | [{name:value,...},{name:value,...}]          | key是字符串类型，value可以是任意类 型, 推荐value也是string类型 |
| 混合类型       | {name: [{name:value,...},{name:value,...}] } |                                                              |

**常用方法**

| 成员方法        | 说明                           |
| --------------- | ------------------------------ |
| stringify(对象) | 将指定对象转换为json格式字符串 |
| parse(字符串)   | 将指定json格式字符串解析成对象 |



#### 2.2 Java中的JSON转换工具

- 我们除了可以在JavaScript 中来使用 JSON 以外，在 JAVA 中同样也可以使用JSON。 

-  JSON 的转换工具是通过JAVA 封装好的一些 JAR 工具包。

- 可以将 JAVA 对象或集合转换成JSON 格式的字符串，也可以将JSON 格式的字符串转成JAVA 对象。 

- Jackson：开源免费的 JSON 转换工具，SpringMVC转换默认使用 Jackson。 

  > 1. 导入 jar 包。 2. 创建核心对象。 3. 调用方法完成转换。

**常用类**

| 类名          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| ObjectMapper  | 是Jackson工具包的核心类, 它提供一些方法来实现 JSON 字符串和对象之间的转 换 |
| TypeReference | 对集合泛型的反序列化，使用TypeReference可以明确的指定反序列化的对象类 |

**该类常用方法**

| 方法名                                                   | 说明                       |
| -------------------------------------------------------- | -------------------------- |
| String writeValueAsString(Object obj)                    | 将Java对象转换成JSON字符串 |
| <T> T readValue(String json, Class<T> valueType)         | 将JSON字符串转换成Java对象 |
| <T> T readValue(String json, TypeReference valueTypeRef) | 将JSON字符串转换成Java对象 |

#### 2.3 JSON 转换练习

- 导入jar包 

  > jackson-annotations-2.2.3.jar
  >
  > jackson-core-2.2.3.jar
  >
  > jackson-databind-2.2.3.jar

##### 1）对象转 JSON, JSON 转对象。 

```java
private ObjectMapper mapper = new ObjectMapper();
/*
    1.User对象转json, json转User对象
    json字符串 = {"name":"张三","age":23}
    user对象 = User{name='张三', age=23}
*/

@Test
public void test01() throws Exception{
    //User对象转json
    User user = new User("张三",23);
    String json = mapper.writeValueAsString(user);
    System.out.println("json字符串：" + json);

    //json转User对象
    User user2 = mapper.readValue(json, User.class);
    System.out.println("java对象：" + user2);
}
```

##### 2）Map<String,String>转JSON, JSON 转 Map<String,String>。 

```java
private ObjectMapper mapper = new ObjectMapper();

/*
    2.map<String,String>转json, json转map<String,String>
    json字符串 = {"姓名":"张三","性别":"男"}
    map对象 = {姓名=张三, 性别=男}
*/
@Test
public void test02() throws Exception{
    //map<String,String>转json
    HashMap<String,String> map = new HashMap<>();
    map.put("姓名","张三");
    map.put("性别","男");
    String json = mapper.writeValueAsString(map);
    System.out.println("json字符串：" + json);

    //json转map<String,String>
    HashMap<String,String> map2 = mapper.readValue(json, HashMap.class);
    System.out.println("java对象：" + map2);
}
```



##### 3）Map<String,User>转JSON, JSON 转 Map<String,User>。 

```java 
/*
        3.map<String,User>转json, json转map<String,User>
          json字符串 = {"黑马一班":{"name":"张三","age":23},"黑马二班":{"name":"李四","age":24}}
          map对象 = {黑马一班=User{name='张三', age=23}, 黑马二班=User{name='李四', age=24}}
*/
@Test
public void test03() throws Exception{
    //map<String,User>转json
    HashMap<String,User> map = new HashMap<>();
    map.put("黑马一班",new User("张三",23));
    map.put("黑马二班",new User("李四",24));
    String json = mapper.writeValueAsString(map);
    System.out.println("json字符串：" + json);

    //json转map<String,User>
    HashMap<String,User> map2 = mapper.readValue(json,new TypeReference<HashMap<String,User>>(){});
    System.out.println("java对象：" + map2);
}

```



##### 4） List-String转JSON, JSON 转 List-String。 

```java
/*
        4.List<String>转json, json转 List<String>
          json字符串 = ["张三","李四"]
          list对象 = [张三, 李四]
*/
@Test
public void test04() throws Exception{
    //List<String>转json
    ArrayList<String> list = new ArrayList<>();
    list.add("张三");
    list.add("李四");
    String json = mapper.writeValueAsString(list);
    System.out.println("json字符串：" + json);

    //json转 List<String>
    ArrayList<String> list2 = mapper.readValue(json,ArrayList.class);
    System.out.println("java对象：" + list2);
}
```

##### 5）List-User转JSON, JSON 转 List-User。

```java
/*
        5.List<User>转json, json转List<User>
          json字符串 = [{"name":"张三","age":23},{"name":"李四","age":24}]
          list对象 = [User{name='张三', age=23}, User{name='李四', age=24}]
*/
@Test
public void test05() throws Exception{
    //List<User>转json
    ArrayList<User> list = new ArrayList<>();
    list.add(new User("张三",23));
    list.add(new User("李四",24));
    String json = mapper.writeValueAsString(list);
    System.out.println("json字符串：" + json);

    //json转List<User>
    ArrayList<User> list2 = mapper.readValue(json,new TypeReference<ArrayList<User>>(){});
    System.out.println("java对象：" + list2);
}
```





### 3. 综合案例 搜索联想



#### 3.1 数据准备

```mysql
-- 创建db10数据库
CREATE DATABASE db10;

-- 使用db10数据库
USE db10;

-- 创建user表
CREATE TABLE USER(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 主键id
	NAME VARCHAR(20),			-- 姓名
	age INT,				-- 年龄
	search_count INT                        -- 搜索数量
);

-- 插入测试数据
INSERT INTO USER VALUES (NULL,'张三',23,25),(NULL,'李四',24,5),(NULL,'王五',25,3)
,(NULL,'赵六',26,7),(NULL,'张三丰',93,20),(NULL,'张无忌',18,23),(NULL,'张强',33,21),(NULL,'张果老',65,6);
```



#### 3.2 案例分析和实现

- 页面 

  > 1）为用户名输入框绑定鼠标点击事件。 
  >
  > 2）获取输入的用户名数据。 
  >
  > 3）判断用户名是否为空。 
  >
  > 4）如果为空，则将联想提示框隐藏。 
  >
  > 5）如果不为空，则发送AJAX 请求，并将响应的数据显示到联想框。 

- 控制层 

  > 1） 获取请求参数。 
  >
  > 2）调用业务层的模糊查询方法。 
  >
  > 3）将返回的数据转成JSON，并响应给客户端。



#### 3.3 前端代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>用户搜索</title>
    <style type="text/css">
        .content {
            width: 643px;
            margin: 100px auto;
            text-align: center;
        }

        input[type='text'] {
            width: 530px;
            height: 40px;
            font-size: 14px;
        }

        input[type='button'] {
            width: 100px;
            height: 46px;
            background: #38f;
            border: 0;
            color: #fff;
            font-size: 15px
        }
        
        .show {
            position: absolute;
            width: 535px;
            height: 100px;
            border: 1px solid #999;
            border-top: 0;
            display: none;
        }
    </style>
</head>
<body>
<form autocomplete="off">
    <div class="content">
        <img src="img/logo.jpg">
        <br/><br/>
        <input type="text" id="username">
        <input type="button" value="搜索一下">
        <!--用于显示联想的数据-->
        <div id="show" class="show"></div>
    </div>
</form>
</body>
<script src="js/jquery-3.3.1.min.js"></script>
<script>
    //1.为用户名输入框绑定鼠标点击事件
    $("#username").keyup(function () {
    // $("#username").mousedown(function () {
        //2.获取输入的用户名
        let username = $("#username").val();

        //3.判断用户名是否为空
        if(username == null || username == "") {
            //4.如果为空，将联想框隐藏
            $("#show").hide();
            return;
        }

        //5.如果不为空，发送AJAX请求。并将数据显示到联想框
        $.ajax({
            //请求的资源路径
            url:"userServlet",
            //请求参数
            data:{"username":username},
            //请求方式
            type:"POST",
            //响应数据形式
            dataType:"json",
            //请求成功后的回调函数
            success:function (data) {
                //将返回的数据显示到show的div
                let names = "";
                for(let i = 0; i < data.length; i++) {
                    names += "<div>"+data[i].name+"</div>";
                }
                $("#show").html(names);
                $("#show").show();
            }
        });
    });


</script>
</html>
```



#### 3.4 Servlet代码

```java
package com.itheima.controller;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.itheima.bean.User;
import com.itheima.service.UserService;
import com.itheima.service.impl.UserServiceImpl;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@WebServlet("/userServlet")
public class UserServlet extends HttpServlet {
    
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //设置请求和响应的编码
        req.setCharacterEncoding("UTF-8");
        resp.setContentType("text/html;charset=UTF-8");

        //1.获取请求参数
        String username = req.getParameter("username");

        //2.调用业务层的模糊查询方法得到数据
        UserService service = new UserServiceImpl();
        List<User> users = service.selectLike(username);

        //3.将数据转成JSON，响应到客户端
        ObjectMapper mapper = new ObjectMapper();
        String json = mapper.writeValueAsString(users);
        resp.getWriter().write(json);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req,resp);
    }
}

```



#### 3.5 Service代码

```java
package com.itheima.service.impl;

import com.itheima.bean.User;
import com.itheima.mapper.UserMapper;
import com.itheima.service.UserService;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class UserServiceImpl implements UserService {
    @Override
    public List<User> selectLike(String username) {
        List<User> users = null;
        SqlSession sqlSession = null;
        InputStream is = null;
        try{
            //1.加载核心配置文件
            is = Resources.getResourceAsStream("MyBatisConfig.xml");
            //2.获取SqlSession工厂对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
            //3.通过SqlSession工厂对象获取SqlSession对象
            sqlSession = sqlSessionFactory.openSession(true);
            //4.获取UserMapper接口的实现类对象
            UserMapper mapper = sqlSession.getMapper(UserMapper.class);
            //5.调用实现类对象的模糊查询方法
            users = mapper.selectLike(username);
        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            //6.释放资源
            if(sqlSession != null) {
                sqlSession.close();
            }
            if(is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        //7.返回结果到控制层
        return users;
    }
}

```



#### 3.6 Mapper代码

```java
package com.itheima.mapper;

import com.itheima.bean.User;
import org.apache.ibatis.annotations.Select;

import java.util.List;

public interface UserMapper {
    /*
        模糊查询  CONCAT('%',#{username},'%') 函数是 mysql 用来做字符串拼接的, 最终会拼接成 '%张%'
     */
    @Select("SELECT * FROM user WHERE name LIKE CONCAT('%',#{username},'%') ORDER BY search_count DESC LIMIT 0,4")
    public abstract List<User> selectLike(String username);
}

```





### 4. 综合案例 分页 （重点）

#### 4.1 环境准备

```mysql
-- 创建db11数据库
CREATE DATABASE db11;

-- 使用db11数据库
USE db11;

-- 创建数据表
CREATE TABLE news(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 主键id
	title VARCHAR(999)                      -- 新闻标题
);

-- 插入数据
DELIMITER $$
CREATE PROCEDURE create_data()		
BEGIN
DECLARE i INT;		
SET i=1;
WHILE i<=100 DO	
	INSERT INTO news VALUES (NULL,CONCAT('奥巴马罕见介入美国2020大选，警告民主党参选人须“基于现实”',i));	
SET i=i+1;		
END WHILE;
END 
$$

-- 调用存储过程
CALL create_data();
```



#### 4.2 瀑布流分页-案例分析和实现

- 如何确定当前显示的数据已经浏览完毕？ **注意**:  文档高度随着文字长度而变化，所以 滚动条上下距离也随之而变化，公式是没有问题的。

  > 公式：(滚动条距底部的距离+ 滚动条上下滚动的距离+ 当前窗口的高度) >= 当前文档的高度
  >
  >  
  >
  > 1) 当前文档高度：存储10条数据，100px。 
  >
  > 2) 滚动条距底部的距离：1px。 
  >
  > 3) 当前窗口的高度：80px。 
  >
  > 4) 滚动条上下滚动的距离：>=19px。 

**前置知识**

- $(function (){})  入口函数，不同于window.onload, 它可以定义多个
- $(js对象)，将JavaScript对象转成JQuery对象，目的是 调用jQuery的方法。

|         功能          |         说明          |
| :-------------------: | :-------------------: |
|    $(function(){})    |     页面加载事件      |
|       $(window)       |   获取当前窗口对象    |
|       scroll()        | jqueryd的鼠标滚动事件 |
|  $(window).height()   |    当前窗口的高度     |
| $(window).scrollTop() | 滚动条上下滚动的距离  |
| $(document).height()  |    当前文档的高度     |

##### 4.2.1 前端代码实现

- 页面 

  > 1）定义发送请求标记。let  issend=false。 默认false表示没有发送。
  >
  > 2）定义当前页码和每页显示的条数。 
  >
  > 3）定义滚动条距底部的距离。 
  >
  > 4）设置页面加载事件。 
  >
  > 5） 为当前窗口绑定滚动条滚动事件。 
  >
  > 6）获取必要信息(当前窗口的高度,滚动条上下滚动的距离,当前文档的高度)。 
  >
  > 7）计算当前展示数据是否浏览完毕。 
  >
  > 8）判断请求标记是否为true。 
  >
  > 9）将请求标记置为false，当前异步操作完成前，不能重新发起请求。 
  >
  > 10）根据当前页和每页显示的条数来请求查询分页数据。 
  >
  > 11）当前页码+1。



```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>网站首页</title>
    <link rel="stylesheet" href="css/tt.css">
</head>
<body>
<div class="top">
    <span class="top-left">下载APP</span>
    <span class="top-left"> 北京         晴天</span>
    <span class="top-right">更多产品</span>
</div>

<div class="container">

    <div class="left">
        <a>
            <img src="img/logo.png"><br/>
        </a>

        <ul>
            <li>
                <a class="channel-item active" href="#">
                    <span>
                        推荐
                    </span>
                </a>
            </li>

            <li><a class="channel-item" href="#">
                <span>
                    视频
                </span>
            </a></li>

            <li><a class="channel-item" href="#">
                <span>
                    热点
                </span>
            </a></li>

            <li><a class="channel-item" href="#">
                <span>
                    直播
                </span>
            </a></li>

            <li><a class="channel-item" href="#">
                <span>
                    图片
                </span>
            </a></li>

            <li><a class="channel-item" href="#">
                <span>
                    娱乐
                </span>
            </a></li>

            <li><a class="channel-item" href="#">
                <span>
                    游戏
                </span>
            </a></li>

            <li><a class="channel-item" href="#">
                <span>
                    体育
                </span>
            </a></li>

        </ul>

    </div>
    <div class="center">
        <ul class="news_list">
            <li>
                <div class="title-box">
                    <a href="#" class="link">
                        奥巴马罕见介入美国2020大选，警告民主党参选人须“基于现实11”
                        <hr>
                    </a>
                </div>
            </li>
            <li>
                <div class="title-box">
                    <a href="#" class="link">
                        奥巴马罕见介入美国2020大选，警告民主党参选人须“基于现实22”
                        <hr>
                    </a>
                </div>
            </li>
            <li>
                <div class="title-box">
                    <a href="#" class="link">
                        奥巴马罕见介入美国2020大选，警告民主党参选人须“基于现实33”
                        <hr>
                    </a>
                </div>
            </li>
            <li>
                <div class="title-box">
                    <a href="#" class="link">
                        奥巴马罕见介入美国2020大选，警告民主党参选人须“基于现实44”
                        <hr>
                    </a>
                </div>
            </li>
            <li>
                <div class="title-box">
                    <a href="#" class="link">
                        奥巴马罕见介入美国2020大选，警告民主党参选人须“基于现实55”
                        <hr>
                    </a>
                </div>
            </li>
            <li>
                <div class="title-box">
                    <a href="#" class="link">
                        奥巴马罕见介入美国2020大选，警告民主党参选人须“基于现实66”
                        <hr>
                    </a>
                </div>
            </li>
            <li>
                <div class="title-box">
                    <a href="#" class="link">
                        奥巴马罕见介入美国2020大选，警告民主党参选人须“基于现实77”
                        <hr>
                    </a>
                </div>
            </li>
            <li>
                <div class="title-box">
                    <a href="#" class="link">
                        奥巴马罕见介入美国2020大选，警告民主党参选人须“基于现实88”
                        <hr>
                    </a>
                </div>
            </li>
            <li>
                <div class="title-box">
                    <a href="#" class="link">
                        奥巴马罕见介入美国2020大选，警告民主党参选人须“基于现实99”
                        <hr>
                    </a>
                </div>
            </li>
            <li>
                <div class="title-box">
                    <a href="#" class="link">
                        奥巴马罕见介入美国2020大选，警告民主党参选人须“基于现实1010”
                        <hr>
                    </a>
                </div>
            </li>
        </ul>

        <!--display: none;默认不显示-->
        <div class="loading" style="text-align: center; height: 80px;display: none;">
            <img src="img/loading2.gif" height="100%">
        </div>

        <div class="content">
            <div class="pagination-holder clearfix">
                <div id="light-pagination" class="pagination"></div>
            </div>
        </div>

        <div id="no" style="text-align: center;color: red;font-size: 20px"></div>
    </div>
</div>
</body>
<script src="js/jquery-3.3.1.min.js"></script>
<script>
    //1.定义发送请求标记, 默认false表示没有发送
    let isSend = false;

    //2.定义当前页码和每页显示的条数
    let start = 1;
    let pageSize = 10;

    //3.定义滚动条距底部的距离
    let bottom = 1;

    //4.设置页面加载事件
    $(function () {
        //5.为当前窗口绑定滚动条滚动事件
        $(window).scroll(function () {
            //6.获取必要信息，用于计算当前展示数据是否浏览完毕
            //当前窗口的高度
            let windowHeight = $(window).height();
            console.log(windowHeight);

            //滚动条从上到下滚动距离
            let scrollTop = $(window).scrollTop();
            console.log(scrollTop);
            //当前文档的高度
            let docHeight = $(document).height();
            console.log(scrollTop);


            //7.计算当前展示数据是否浏览完毕
            //当 滚动条距底部的距离 + 当前滚动条滚动的距离 + 当前窗口的高度 >= 当前文档的高度
            if((bottom + scrollTop + windowHeight) >= docHeight) {
                //8.判断请求标记是否为false
                if(isSend == false) {
                    //9.将请求标记置为true，当前异步操作完成前，不能重新发起请求。
                    isSend = true;
                    //10.根据当前页和每页显示的条数来 请求查询分页数据
                    queryByPage(start,pageSize);
                    //11.当前页码+1
                    start++;
                }
            }
        });
    });

    //定义查询分页数据的函数
    function queryByPage(start,pageSize){
        //加载动图显示
        $(".loading").show();
        //发起AJAX请求
        $.ajax({
            //请求的资源路径
            url:"newsServlet",
            //请求的参数
            data:{"start":start,"pageSize":pageSize},
            //请求的方式
            type:"POST",
            //响应数据形式
            dataType:"json",
            //请求成功后的回调函数
            success:function (data) {
                if(data.length == 0) {
                    $(".loading").hide();
                    $("#no").html("我也是有底线的...");
                    return;
                }
                //加载动图隐藏
                $(".loading").hide();
                //将数据显示
                let titles = "";
                for(let i = 0; i < data.length; i++) {
                    titles += "<li>\n" +
                        "                <div class=\"title-box\">\n" +
                        "                    <a href=\"#\" class=\"link\">\n" +
                                                    data[i].title +
                        "                        <hr>\n" +
                        "                    </a>\n" +
                        "                </div>\n" +
                        "            </li>";
                }

                //显示到页面
                $(".news_list").append(titles);
                //将请求标记设置为false
                isSend = false;
            }
        });
    }

</script>
</html>
```



##### 4.2.2 后端代码实现

- 服务器 

  > 1）获取请求参数(当前页,每页显示的条数)。 
  >
  > 2）根据当前页码和每页显示的条数，调用业务层的方法，得到分页Page 对象。 
  >
  > 3）将得到的数据转为json。 
  >
  > 4）将数据响应给客户端。

##### 4.2.3 Servlet

```java
package com.itheima.controller;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.github.pagehelper.Page;
import com.itheima.bean.News;
import com.itheima.service.NewsService;
import com.itheima.service.impl.NewsServiceImpl;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@WebServlet("/newsServlet")
public class NewsServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //设置请求和响应的编码
        req.setCharacterEncoding("UTF-8");
        resp.setContentType("text/html;charset=UTF-8");

        //1.获取请求参数
        String start = req.getParameter("start");
        String pageSize = req.getParameter("pageSize");

        //2.根据当前页码和每页显示的条数来调用业务层的查询方法，得到分页Page对象
        NewsService service = new NewsServiceImpl();
        List<News> newsList = service.pageQuery(Integer.parseInt(start), Integer.parseInt(pageSize));

        //3.将得到的数据转为JSON
        String json = new ObjectMapper().writeValueAsString(newsList);

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        //4.将数据响应给客户端
        resp.getWriter().write(json);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req,resp);
    }
}

```



##### 4.2.4 Service

```java
package com.itheima.service.impl;

import com.github.pagehelper.Page;
import com.github.pagehelper.PageHelper;
import com.itheima.bean.News;
import com.itheima.mapper.NewsMapper;
import com.itheima.service.NewsService;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class NewsServiceImpl implements NewsService {

    @Override
    public List<News> pageQuery(Integer start, Integer pageSize) {
        InputStream is = null;
        SqlSession sqlSession = null;
        List<News> newsList = null;
        try{
            //1.加载核心配置文件
            is = Resources.getResourceAsStream("MyBatisConfig.xml");

            //2.获取SqlSession工厂对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

            //3.通过SqlSession工厂对象获取SqlSession对象
            sqlSession = sqlSessionFactory.openSession(true);

            //4.获取NewsMapper接口的实现类对象
            NewsMapper mapper = sqlSession.getMapper(NewsMapper.class);

            //5.封装Page对象   start:当前页码   pageSize：每页显示的条数
            PageHelper.startPage(start,pageSize);

            //6.调用实现类对象的查询全部方法，此时底层执行的就是MySQL的limit分页查询语句
            newsList= mapper.selectAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //7.释放资源
            if(sqlSession != null) {
                sqlSession.close();
            }
            if(is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        //8.返回List<News>对象到控制层
        return newsList;
    }
}

```



##### 4.2.5 Mapper

```java
package com.itheima.mapper;

import com.itheima.bean.News;
import org.apache.ibatis.annotations.Select;

import java.util.List;

public interface NewsMapper {
    /*
        查询全部
     */
    @Select("SELECT * FROM news")
    public abstract List<News> selectAll();
}

```



#### 4.3 按钮分页-分析和实现

- ```html
  <!-- Jquery 分页插件 css文件 -->
  <link rel="stylesheet" href="css/simplePagination.css">
  
  <!-- Jquery 分页插件 js文件 -->
  <script src="js/jquery.simplePagination.js"></script>
  ```

- 页面 

  > 1）引入分页插件的样式文件和js 文件。
  >
  >  2）定义当前页码和每页显示的条数。 
  >
  > 3）调用查询数据的函数。 
  >
  > 4）定义请求查询分页数据的函数，发起AJAX 异步请求。 
  >
  > 5）为分页按钮区域设置页数参数(总页数和当前页)。 
  >
  > 6）为分页按钮绑定单击事件,完成上一页下一页查询功能。



- 服务器 

  > 1）获取请求参数。 
  >
  > 2）根据当前页码和每页显示的条数，调用业务层的方法，得到分页Page 对象。 
  >
  > 3）封装 PageInfo 对象。 
  >
  > 4） 将得到的数据转为json。 
  >
  > 5）将数据响应给客户端。



##### 4.3.1 前端代码实现

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>网站首页</title>
    <link rel="stylesheet" href="css/tt.css">
    <link rel="stylesheet" href="css/simplePagination.css">
</head>
<body>
<div class="top">
    <span class="top-left">下载APP</span>
    <span class="top-left"> 北京         晴天</span>
    <span class="top-right">更多产品</span>
</div>

<div class="container">

    <div class="left">
        <a>
            <img src="img/logo.png"><br/>
        </a>

        <ul>
            <li>
                <a class="channel-item active" href="#">
                    <span>
                        推荐
                    </span>
                </a>
            </li>

            <li><a class="channel-item" href="#">
                <span>
                    视频
                </span>
            </a></li>

            <li><a class="channel-item" href="#">
                <span>
                    热点
                </span>
            </a></li>

            <li><a class="channel-item" href="#">
                <span>
                    直播
                </span>
            </a></li>

            <li><a class="channel-item" href="#">
                <span>
                    图片
                </span>
            </a></li>

            <li><a class="channel-item" href="#">
                <span>
                    娱乐
                </span>
            </a></li>

            <li><a class="channel-item" href="#">
                <span>
                    游戏
                </span>
            </a></li>

            <li><a class="channel-item" href="#">
                <span>
                    体育
                </span>
            </a></li>
        </ul>

    </div>
    <div class="center">
        <ul class="news_list">

        </ul>

        <div class="content">
            <div class="pagination-holder clearfix">
                <div id="light-pagination" class="pagination"></div>
            </div>
        </div>

    </div>
</div>
</body>
<script src="js/jquery-3.3.1.min.js"></script>
<script src="js/jquery.simplePagination.js"></script>
<script>
    //1.定义当前页码和每页显示的条数
    let start = 1;
    let pageSize = 10;

    //2.调用查询数据的方法
    queryByPage(start,pageSize);

    //3.定义请求查询分页数据的函数，发起AJAX异步请求，将数据显示到页面
    function queryByPage(start,pageSize) {
        $.ajax({
            //请求的资源路径
            url:"newsServlet2",
            //请求的参数
            data:{"start":start,"pageSize":pageSize},
            //请求的方式
            type:"POST",
            //响应数据形式
            dataType:"json",
            //请求成功后的回调函数
            success:function (pageInfo) {
                //将数据显示到页面
                let titles = "";
                for(let i = 0; i < pageInfo.list.length; i++) {
                    titles += "<li>\n" +
                        "                <div class=\"title-box\">\n" +
                        "                    <a href=\"#\" class=\"link\">\n" +
                                                pageInfo.list[i].title +
                        "                        <hr>\n" +
                        "                    </a>\n" +
                        "                </div>\n" +
                        "            </li>";
                }
                $(".news_list").html(titles);

                //4.为分页按钮区域设置页数参数（总页数和当前页）
                $("#light-pagination").pagination({
                    pages:pageInfo.pages,
                    currentPage:pageInfo.pageNum
                });

                //5.为分页按钮绑定单击事件,完成上一页下一页查询功能
                $("#light-pagination .page-link").click(function () {
                    //获取点击按钮的文本内容
                    let page = $(this).html();
                    //如果点击的是Prev，调用查询方法，查询当前页的上一页数据
                    if(page == "Prev") {
                        queryByPage(pageInfo.pageNum - 1,pageSize);
                    }else if (page == "Next") {
                        //如果点击的是Next，调用查询方法，查询当前页的下一页数据
                        queryByPage(pageInfo.pageNum + 1,pageSize);
                    } else {
                        //调用查询方法，查询当前页的数据
                        queryByPage(page,pageSize);
                    }
                });
            }
        });
    }

</script>
</html>
```



##### 4.3.2 后端代码实现-Servlet

```java
package com.itheima.controller;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.github.pagehelper.Page;
import com.github.pagehelper.PageInfo;
import com.itheima.bean.News;
import com.itheima.service.NewsService;
import com.itheima.service.impl.NewsServiceImpl;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@WebServlet("/newsServlet2")
public class NewsServlet2 extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //设置请求和响应的编码
        req.setCharacterEncoding("UTF-8");
        resp.setContentType("text/html;charset=UTF-8");

        //1.获取请求参数
        String start = req.getParameter("start");
        String pageSize = req.getParameter("pageSize");

        //2.根据当前页码和每页显示的条数来调用业务层的查询方法，得到分页Page对象
        NewsService service = new NewsServiceImpl();
        List<News> newsList = service.pageQuery(Integer.parseInt(start), Integer.parseInt(pageSize));

        //3.封装PageInfo对象
        PageInfo<List<News>> info = new PageInfo(newsList);

        //4.将得到的数据转为JSON
        String json = new ObjectMapper().writeValueAsString(info);

        //5.将数据响应给客户端
        resp.getWriter().write(json);

    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
        doPost(req,resp);
    }
}

```



### 5. 扩展

#### 5.1 JQuery 实现AJAX 其他请求方式

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>ajax</title>
</head>
<body>

<script src="jquery-3.3.1.js"></script>
<script>

    //入口函数
    $(function () {

        $("#btnDelete").click(function(){

            var url = "/ajax/ajaxServlet?method=DELETE&msg=send delete";

            //注意不能用data, 传递json 后台接收不到数据
            // var data = {"method": "PUT", "msg": "send put"};

            $.ajax({
                Url: url,
                type: 'DELETE',
                // async : true, // 默认true 为异步请求，false 为同步请求
                dataType:"text", //设置接受到的响应数据的格式
                success:function (resultData, status) { //响应成功后的回调函数
                    alert('状态码:' + status + ", 响应报文: " + resultData);
                },
                error:function (data) { //表示如果请求响应出现错误，会执行的回调函数
                    alert("出错啦..." + data)
                }

            });

        });

        $("#btnPut").click(function(){

            var url = "/ajax/ajaxServlet?method=PUT&msg=send put";

            //注意不能用data, 传递json 后台接收不到数据
            // var data = {"method": "PUT", "msg": "send put"};

            $.ajax({
                type: "PUT",
                url: url,
                async : true, // 默认true 为异步请求，false 为同步请求
                dataType:"text", //设置接受到的响应数据的格式
                success:function (resultData, status) { //响应成功后的回调函数
                    alert('状态码:' + status + ", 响应报文: " + resultData);
                },
                error:function (data) { //表示如果请求响应出现错误，会执行的回调函数
                    alert("出错啦..." + data)
                }
            });

        });


    });


</script>

<input type="button" value="发送DELETE" id="btnDelete">

<input type="button" value="发送PUT" id="btnPut">


</body>
</html>
```

