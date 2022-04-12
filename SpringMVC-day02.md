##  SpringMVC-day02

### **学习目标**：

​    **1）** 实现Ajax异步处理，前后端数据交互（请求和响应）

​    **2）** 实现拦截器，处理请求--概念和作用

​    **3）**基于注解实现全局异常处理器--了解它的思路

​    **4）**实现文件上传和下载

​    **5）**了解Restful 风格--什么是restful?



### 1. 异步调用

- 异步请求，局部刷新

#### 1.1 JQuery Ajax 回顾

```html 
<a href="javascript:void(0);" id="testAjax">访问controller</a>
<script type="text/javascript" src="/js/jquery-3.3.1.min.js"></script>
<script type="text/javascript">
    $(function(){
        // 为 id="testAjax" 的组件绑定点击事件
        $("#testAjax").click(function(){ 
            $.ajax({ // 发送异步调用
            type:"POST", // 请求方式： POST 请求
            url:"ajaxController", // 请求参数（也就是请求内容）
            data:'ajax message', // 请求参数（也就是请求内容）
            dataType:"text", // 响应正文类型
            contentType:"application/text", // 请求正文的 MIME 类型
            });
        });
    });
</script>
```



#### 1.2 异步请求传参

- @RequestBody  使用该注解，可以将请求体内容封装到指定参数中

##### 方式一

```javascript
//为id="testAjax"的组件绑定点击事件
$("#testAjax").click(function(){
    //发送异步调用
    $.ajax({
        //请求方式：POST请求
        type:"POST",
        //请求的地址
        url:"ajaxController",
        //请求参数（也就是请求内容）
        data:'ajax message',
        //响应正文类型
        dataType:"text",
        //请求正文的MIME类型
        contentType:"application/text",
    });
});
```

```java
@RequestMapping("/ajaxController")
//使用@RequestBody注解，可以将请求体内容封装到指定参数中
public String ajaxController(@RequestBody String message){
    System.out.println("ajax request is running..."+message);
    return "page.jsp";
}
```

##### 方式二

```js
//为id="testAjaxPojo"的组件绑定点击事件
$("#testAjaxPojo").click(function(){
    $.ajax({
        type:"POST",
        url:"ajaxPojoToController",
        data:'{"name":"Jock","age":39}',
        dataType:"text",
        contentType:"application/json",
    });
});
```

```java
@RequestMapping("/ajaxPojoToController")
//如果处理参数是POJO，且页面发送的请求数据格式与POJO中的属性对应，@RequestBody注解可以自动映射对应请求数据到POJO中
//注意：POJO中的属性如果请求数据中没有，属性值为null，POJO中没有的属性如果请求数据中有，不进行映射
public String  ajaxPojoToController(@RequestBody User user){
    System.out.println("controller pojo :"+user);
    return "page.jsp";
}
```

##### 方式三

```js
//为id="testAjaxList"的组件绑定点击事件
$("#testAjaxList").click(function(){
    $.ajax({
        type:"POST",
        url:"ajaxListToController",
        data:'[{"name":"Jock","age":39},{"name":"Jockme","age":40}]',
        dataType:"text",
        contentType:"application/json",
    });
});
```

```java
@RequestMapping("/ajaxListToController")
//如果处理参数是List集合且封装了POJO，且页面发送的数据是JSON格式的对象数组，数据将自动映射到集合参数中
public String  ajaxListToController(@RequestBody List<User> userList){
    System.out.println("controller list :"+userList);
    return "page.jsp";
}
```

##### 方式四

```js
//为id="testAjaxReturnString"的组件绑定点击事件
$("#testAjaxReturnString").click(function(){
//发送异步调用
$.ajax({
        type:"POST",
        url:"ajaxReturnString",
        //回调函数
        success:function(data){
            //打印返回结果
            alert(data);
        }
    });
});
```

```java
//使用注解@ResponseBody可以将返回的页面不进行解析，直接返回字符串，该注解可以添加到方法上方或返回值前面
@RequestMapping("/ajaxReturnString")
//    @ResponseBody
public @ResponseBody String ajaxReturnString(){
    System.out.println("controller return string ...");
    return "page.jsp";
}
```

##### 方式五

```js
 //为id="testAjaxReturnJson"的组件绑定点击事件
 $("#testAjaxReturnJson").click(function(){
     //发送异步调用
     $.ajax({
         type:"POST",
         url:"ajaxReturnJson",
         //回调函数
         success:function(data){
         alert(data);
         alert(data['name']+" ,  "+data['age']);
         }
     });
 });
```

```Java
@RequestMapping("/ajaxReturnJson")
@ResponseBody
//基于jackon技术，使用@ResponseBody注解可以将返回的POJO对象转成json格式数据
public User ajaxReturnJson(){
    System.out.println("controller return json pojo...");
    User user = new User();
    user.setName("Jockme");
    user.setAge(39);
    return user;
}
```

##### 方式六

```js
//为id="testAjaxReturnJsonList"的组件绑定点击事件
$("#testAjaxReturnJsonList").click(function(){
    //发送异步调用
    $.ajax({
        type:"POST",
        url:"ajaxReturnJsonList",
        //回调函数
        success:function(data){
            alert(data);
            alert(data.length);
            alert(data[0]["name"]);
            alert(data[1]["age"]);
        }
    });
});
```

```java

@RequestMapping("/ajaxReturnJsonList")
@ResponseBody
//基于jackon技术，使用@ResponseBody注解可以将返回的保存POJO对象的集合转成json数组格式数据
public List ajaxReturnJsonList(){
    System.out.println("controller return json list...");
    User user1 = new User();
    user1.setName("Tom");
    user1.setAge(3);

    User user2 = new User();
    user2.setName("Jerry");
    user2.setAge(5);

    ArrayList al = new ArrayList();
    al.add(user1);
    al.add(user2);

    return al;
}
```

##### 跨域访问

- @CrossOrigin 可以修饰类，也可以修饰方法

```js
//为id="testCross"的组件绑定点击事件
$("#testCross").click(function(){
    //发送异步调用
    $.ajax({
        type:"POST",
        url:"http://www.jock.com/cross",
        //回调函数
        success:function(data){
        	alert("跨域调用信息反馈："+data['name']+" ,  "+data['age']);
        }
    });
});
```

```java
@RequestMapping("/cross")
@ResponseBody
//使用@CrossOrigin开启跨域访问
//标注在处理器方法上方表示该方法支持跨域访问
//标注在处理器类上方表示该处理器类中的所有处理器方法均支持跨域访问
@CrossOrigin
public User cross(HttpServletRequest request){
    System.out.println("controller cross..."+request.getRequestURL());
    User user = new User();
    user.setName("Jockme");
    user.setAge(39);
    return user;
}
```



### 2. 拦截器

- 拦截器 （Intercepoer）是一种动态拦截方法调用机制

- 作用：

    1）在指定方法调用前后执行预先设定后的代码

     2）阻止原始方法的执行

- 核心原理：AOP思想
- 拦截器链：多个拦截器按照一定的顺序，对原始被调用功能进行增强

**拦截器VS过滤器**

- 归属不同：Interceptor拦截器归属SpringMvc，Filter过滤器归属Servlet
- Filter对所有访问增强，Interceptor 只对基于SpringMvc 的请求拦截
- Filter的执行顺序在Interceptor之前 



#### 2.1 自定义拦截器与配置

- request 请求对象
- response 响应对象
- handler 被调用的处理器对象，本质上是一个方法对象，对反射中的Method对象进行了再包装
- 返回值：为false，被拦截器的处理器将不执行
- 后置处理方法：原始方法运行后运行，如果原始方法被拦截，将不执行
- <font color=red>◆ preHandler  必定执行</font>   ◆ postHandler 未必执行   ◆ afterCompletion   未必执行 

##### spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <mvc:annotation-driven/>

    <context:component-scan base-package="com.itheima"/>

    <!--开启拦截器使用-->
    <mvc:interceptors>
        <!--开启具体的拦截器的使用，可以配置多个-->
        <mvc:interceptor>
            <!--设置拦截器的拦截路径，支持*通配-->
            <!--/**         仅表示拦截根路径以及子路径下任意名称 -->
            <!--/*          仅表示根路径下任意名称，不在往下匹配路径 -->
            <!--/           表示拦截根映射 -->
            <!--/user/*     表示拦截所有/user/开头的映射-->
            <!--/user/add*  表示拦截所有/user/开头，且具体映射名称以add开头的映射-->
            <!--/user/*All  表示拦截所有/user/开头，且具体映射名称以All结尾的映射-->
            <mvc:mapping path="/*"/>
            <mvc:mapping path="/**"/>
            <mvc:mapping path="/handleRun*"/>
            <!--设置拦截排除的路径，配置/**或/*，达到快速配置的目的-->
            <mvc:exclude-mapping path="/b*"/>
            <!--指定具体的拦截器类 只能配置一个 -->
            <bean class="com.itheima.interceptor.MyInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>


</beans>
```

##### MyInterceptor.java

```java
package com.itheima.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
//自定义拦截器需要实现HandleInterceptor接口
public class MyInterceptor implements HandlerInterceptor {

    //处理器运行之前执行
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("前置运行----a1");
        //返回值为false将拦截原始处理器的运行
        //如果配置多拦截器，返回值为false将终止当前拦截器后面配置的拦截器的运行
        return true;
        
        //判断是否登录
        /*Object user = request.getSession().getAttribute(Constants.SESSION_USER );
        if (user == null) {
            response.sendRedirect(request.getContextPath()+"/user/login/toLogin");
            return false;
        }
        return true;*/
    }

    //处理器运行之后执行
    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler,
                           ModelAndView modelAndView) throws Exception {
        System.out.println("后置运行----b1");
    }

    //所有拦截器的后置执行全部结束后，执行该操作
    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler,
                                Exception ex) throws Exception {
        System.out.println("完成运行----c1");
    }

    //三个方法的运行顺序为    preHandle -> postHandle -> afterCompletion
    //如果preHandle返回值为false，三个方法仅运行preHandle
}
```

##### InterceptorController.java

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Controller
public class InterceptorController {

    @RequestMapping("/handleRun")
    public String handleRun() {
        System.out.println("业务处理器运行------------main");
        return "page.jsp";
    }
}
```



#### 2.2 多拦截器配置

- 当配置多个拦截器时，形成拦截器链
- 拦截器链的运行顺序参照配置的先后顺序
- 当拦截器中出现对原始处理器的拦截，后面的拦截器均终止运行
- 当拦截器运行中断，仅运行配置再前面拦截器的afterCompletion操作

##### spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <mvc:annotation-driven/>

    <context:component-scan base-package="com.itheima"/>

    <!--开启拦截器使用-->
    <mvc:interceptors>
        <!--开启具体的拦截器的使用，可以配置多个-->
        <mvc:interceptor>
            <!--设置拦截器的拦截路径，支持*通配-->
            <!--/**         仅表示拦截根路径以及子路径下任意名称 -->
            <!--/*          仅表示根路径下任意名称，不在往下匹配路径 -->
            <!--/           表示拦截根映射 -->
            <!--/user/*     表示拦截所有/user/开头的映射-->
            <!--/user/add*  表示拦截所有/user/开头，且具体映射名称以add开头的映射-->
            <!--/user/*All  表示拦截所有/user/开头，且具体映射名称以All结尾的映射-->
            <mvc:mapping path="/*"/>
            <mvc:mapping path="/**"/>
            <mvc:mapping path="/handleRun*"/>
            <!--设置拦截排除的路径，配置/**或/*，达到快速配置的目的-->
            <mvc:exclude-mapping path="/b*"/>
            <!--指定具体的拦截器类 只能配置一个 -->
            <bean class="com.itheima.interceptor.MyInterceptor"/>
        </mvc:interceptor>
<!--        配置多个拦截器，配置顺序即为最终运行顺序 -->
        <mvc:interceptor>
            <mvc:mapping path="/*"/>
            <bean class="com.itheima.interceptor.MyInterceptor2"/>
        </mvc:interceptor>
        <mvc:interceptor>
            <mvc:mapping path="/*"/>
            <bean class="com.itheima.interceptor.MyInterceptor3"/>
        </mvc:interceptor>
    </mvc:interceptors>

</beans>
```

##### MyInterceptor.java

```java
package com.itheima.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
//自定义拦截器需要实现HandleInterceptor接口
public class MyInterceptor implements HandlerInterceptor {

    //处理器运行之前执行
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("前置运行----a1");
        //返回值为false将拦截原始处理器的运行
        //如果配置多拦截器，返回值为false将终止当前拦截器后面配置的拦截器的运行
        return true;
    }

    //处理器运行之后执行
    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler,
                           ModelAndView modelAndView) throws Exception {
        System.out.println("后置运行----b1");
    }

    //所有拦截器的后置执行全部结束后，执行该操作
    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler,
                                Exception ex) throws Exception {
        System.out.println("完成运行----c1");
    }

    //三个方法的运行顺序为    preHandle -> postHandle -> afterCompletion
    //如果preHandle返回值为false，三个方法仅运行preHandle
}
```

##### MyInterceptor2.java

```java
package com.itheima.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyInterceptor2 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("前置运行------a2");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler,
                           ModelAndView modelAndView) throws Exception {
        System.out.println("后置运行------b2");
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler,
                                Exception ex) throws Exception {
        System.out.println("完成运行------c2");
    }
}
```

##### MyInterceptor3.java

```java
package com.itheima.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyInterceptor3 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("前置运行--------a3");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler,
                           ModelAndView modelAndView) throws Exception {
        System.out.println("后置运行--------b3");
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler,
                                Exception ex) throws Exception {
        System.out.println("完成运行--------c3");
    }
}
```

#### 2.3 SpringMvc开放静态资源

- webapp/ 下静态资源：图片，css,  js 等需要特殊配置才能访问

- 说明：

    1）location：本地资源路径，注意必须是webapp根目录下的路径。 
    2）mapping:值在img文件下的所有文件(**代表所有文件，包括子路径，即接多个/） 
    3）所有意思就是在根目录下img的所有文件不会被DispatcherServlet拦截，直接访问，当做静态资源交个Servlet处理

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <mvc:annotation-driven/>

    <context:component-scan base-package="com.itheima"/>

    <!--
        SpringMvc 开放静态资源
        1). location:指location指定的目录不要拦截，直接请求，这里指在根目录下的img文件下的所有文件
        2). mapping:值在img文件下的所有文件(**代表所有文件)
        3). 所有意思就是在根目录下img的所有文件不会被DispatcherServlet拦截，直接访问，当做静态资源交个Servlet处理
    -->
    <mvc:resources mapping="/img/**" location="/img/" />

</beans>
```



### 3. 异常处理器

-  通过 **异常处理器** 我们可以对 所有 @Controller 注解修饰控制器类进行统一的异常管理。 

-  推荐使用：  `@ControllerAdvice` + `@ExceptionHandler` 进行全局的 `Controller` 层异常处理。只要设计得当，就再也不用在 `Controller` 层进行 `try-catch` 了！ 

- `@ExceptionHandler`：用于捕获所有**控制器类**里面的异常，并进行处理。

  

#### 3.1 HandlerExceptionResolver

ExceptionResolver.java

```java
package com.itheima.exception;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class ExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request,
                                         HttpServletResponse response,
                                         Object handler,
                                         Exception ex) {
        System.out.println("my exception is running ...."+ex);
        ModelAndView modelAndView = new ModelAndView();
        if( ex instanceof NullPointerException){
            modelAndView.addObject("msg","空指针异常");
        }else if ( ex instanceof  ArithmeticException){
            modelAndView.addObject("msg","算数运算异常");
        }else{
            modelAndView.addObject("msg","未知的异常");
        }
        modelAndView.setViewName("error.jsp");
        return modelAndView;
    }
}
```

#### 3.2 @ControllerAdvice 

- 类注解，设置当前类为：异常处理器类
- `@ExceptionHandler`：用于捕获所有控制器里面的异常，并进行处理。
-  `@ControllerAdvice` + `@ExceptionHandler` 进行全局的 `Controller` 层异常处理。只要设计得当，就再也不用在 `Controller` 层进行 `try-catch` 了！ 
- 可配合@ResponseBody 注解修饰在方法上，返回json数据，如果整个异常处理器中所有方法都返回json可以使用 `@ControllerAdvice`  替代 `@ControllerAdvice` + `@ExceptionHandler` 

##### ProjectExceptionAdvice.java

```java
package com.itheima.exception;

import org.springframework.stereotype.Component;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

@Component
@ControllerAdvice
public class ProjectExceptionAdvice {

    @ExceptionHandler(BusinessException.class)
    public String doBusinessException(Exception ex, Model m){
        //使用参数Model将要保存的数据传递到页面上，功能等同于ModelAndView
        //业务异常出现的消息要发送给用户查看
        m.addAttribute("msg",ex.getMessage());
        return "error.jsp";
    }

    @ExceptionHandler(SystemException.class)
    public String doSystemException(Exception ex, Model m){
        //系统异常出现的消息不要发送给用户查看，发送统一的信息给用户看
        m.addAttribute("msg","服务器出现问题，请联系管理员！");
        //实际的问题现象应该传递给redis服务器，运维人员通过后台系统查看
        //实际的问题显现更应该传递给redis服务器，运维人员通过后台系统查看
        return "error.jsp";
    }

    @ExceptionHandler(Exception.class)
    public String doException(Exception ex, Model m){
        m.addAttribute("msg",ex.getMessage());
        //将ex对象保存起来
        return "error.jsp";
    }

}
```

##### SystemException.java

```java
package com.itheima.exception;
//自定义异常继承RuntimeException，覆盖父类所有的构造方法
public class SystemException extends RuntimeException {
    public SystemException() {
    }

    public SystemException(String message) {
        super(message);
    }

    public SystemException(String message, Throwable cause) {
        super(message, cause);
    }

    public SystemException(Throwable cause) {
        super(cause);
    }

    public SystemException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```

##### BusinessException.java

```java
package com.itheima.exception;
//自定义异常继承RuntimeException，覆盖父类所有的构造方法
public class BusinessException extends RuntimeException {
    public BusinessException() {
    }

    public BusinessException(String message) {
        super(message);
    }

    public BusinessException(String message, Throwable cause) {
        super(message, cause);
    }

    public BusinessException(Throwable cause) {
        super(cause);
    }

    public BusinessException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```

#### 3.3 二者对比

- 注解处理器可以拦截到入参类型转换异常
- 非注解无法拦截到到入参类型转换异常 
- **推荐**使用注解简洁方便



#### 3.4 异常处理器开发规范

- 异常分类

    1）业务异常：用户执行过程中没有按照要求执行时产生的异常

    2）系统异常：系统运行过程中，可预计无法避免的异常，如果网络IO方面

    3）其他异常：编程人员未预期到的异常

- 异常处理方案：

    1）业务异常：发送对应提示数据给用户，提醒规范操作

    2）系统异常&其他异常：

  ​             （1） 发送固定提示数据给用户，安抚用户

  ​             （2）发送特定消息给运维人员，提醒维护

  ​             （3）记录日志

  

### 4 文件上传&下载

**注意事项：**

-  防止文件覆盖的现象发生，产生一个唯一的文件名 ，可使用UUID：

  ```java
  UUID.randomUUID().toString().replace("-", "").toUpperCase();
  ```

- 要限制上传文件的最大值, 文件过大响应上传速度，容易受网络影响上传失败。
- 可以限制上传文件的类型，在收到上传文件名时，判断后缀名是否合法。

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>springmvc-fileupload</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- servlet3.0规范的坐标 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
        <!--jsp坐标-->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.1</version>
            <scope>provided</scope>
        </dependency>
        <!--spring的坐标-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <!--springmvc的坐标-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>

        <!--文件上传下载-->
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.4</version>
        </dependency>
        
    </dependencies>

    <build>
        <!--设置插件-->
        <plugins>
            <!--具体的插件配置-->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.1</version>
                <configuration>
                    <port>80</port>
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

#### fileupload.jsp

```jsp
<%@page pageEncoding="UTF-8" language="java" contentType="text/html;UTF-8" %>

<form action="${pageContext.request.contextPath}/fileupload" method="post" enctype="multipart/form-data">
    <%--文件上传表单的name属性值一定要与controller处理器中方法的参数对应，否则无法实现文件上传--%>
    上传LOGO：<input type="file" name="file"/><br/>
    上传照片：<input type="file" name="file1"/><br/>
    上传任意文件：<input type="file" name="file2"/><br/>
    <input type="submit" value="上传"/>
</form>
```

#### FileUtil.java

```java
package com.itheima.controller;

import org.springframework.util.StringUtils;
import org.springframework.web.multipart.MultipartFile;
import sun.misc.BASE64Encoder;

import javax.activation.MimetypesFileTypeMap;
import java.io.*;
import java.net.URLEncoder;

/**
 * 文件处理工具类
 * @author ZJW
 * @date 2019-9-10
 */
public final class FileUtil {

    /**
     * 上传文件保存路径根路径
     */
    public static final String FILE_PATH ="D:/itheima/leyou/image/";


    /**
     * 文件保存在本地
     *
     * @param fileName
     * @param bytes
     * @return
     */
    public static String saveLocal(String fileName, byte[] bytes) {
        String localPath = FILE_PATH + fileName;
        BufferedOutputStream bos = null;
        FileOutputStream fos = null;
        try {
            File dir = new File(localPath.substring(0, localPath.lastIndexOf("/") + 1));
            if (!dir.exists()) {//判断文件目录是否存在
                dir.mkdirs();
            }
            File savaFile = new File(localPath);
            fos = new FileOutputStream(savaFile);
            bos = new BufferedOutputStream(fos);
            bos.write(bytes);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (bos != null) {
                try {
                    bos.close();
                } catch (IOException e1) {
                    e1.printStackTrace();
                }
            }
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e1) {
                    e1.printStackTrace();
                }
            }
        }
        return FILE_PATH + fileName;
    }


    /**
     * 文件保存在本地
     *
     * @param fileName
     * @param is
     * @return
     */
    public static String saveLocal(String fileName, InputStream is) {

        String localPath = FILE_PATH + fileName;
        BufferedOutputStream bos = null;
        FileOutputStream fos = null;
        try {
            File dir = new File(localPath.substring(0, localPath.lastIndexOf("/") + 1));
            if (!dir.exists()) {//判断文件目录是否存在
                dir.mkdirs();
            }
            File savaFile = new File(localPath);
            fos = new FileOutputStream(savaFile);
            bos = new BufferedOutputStream(fos);
            //获取字节数组并写入到本地
            bos.write(getStreamBytes(is));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (bos != null) {
                try {
                    bos.close();
                } catch (IOException e1) {
                    e1.printStackTrace();
                }
            }
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e1) {
                    e1.printStackTrace();
                }
            }
        }
        return FILE_PATH + fileName;
    }

    /**
     * 文件保存在本地
     * @return
     */
    public static String saveLocal(MultipartFile file) {

        // 获取原文件名
        String fileName = file.getOriginalFilename();

        // 写入文件
        try {
            // 判断文件是否为空，空则返回失败页面
            if (file.isEmpty()) {
                return "failed";
            }
            // 创建文件实例
            File filePath = new File(FILE_PATH, fileName);
            // 如果文件目录不存在，创建目录
            if (!filePath.getParentFile().exists()) {
                filePath.getParentFile().mkdirs();
            }
            //将文件写到本地磁盘
            file.transferTo(filePath);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return FILE_PATH + fileName;
    }

    //删除临时文件
    public static void deleteFile(String filePath) {

        try {
            File file = new File(filePath);
            if (file.exists()) {
                file.delete();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 流转字节数组
     * @param is
     * @return
     * @throws Exception
     */
    public static byte[] getStreamBytes(InputStream is) throws Exception {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        byte[] buffer = new byte[1024];
        int len = 0;
        while ((len = is.read(buffer)) != -1) {
            baos.write(buffer, 0, len);
        }
        byte[] b = baos.toByteArray();
        is.close();
        baos.close();
        return b;
    }

    /**
     * 根据文件路径获取字节数组
     * @param filePath 文件路径
     * @return
     * @throws Exception
     */
    public static byte[] getBytesByFilePath(String filePath) throws FileNotFoundException, Exception {

        if(StringUtils.isEmpty(filePath)){
            throw new NullPointerException("filePath 不能为空");
        }

        File file = new File(filePath);
        InputStream in = new FileInputStream(file);
        byte[] fileByte = FileUtil.getStreamBytes(in);
        return fileByte;

    }

    /**
     * 获取文件扩展名
     * @param file
     * @return
     */
    public static String getFileExtension(MultipartFile file) {
        String originalFileName = file.getOriginalFilename();
        return originalFileName.substring(originalFileName.lastIndexOf("."));
    }

    /**
     * 解决中文名字文件在不同浏览器下载时，文件名字不被识别问题
     *
     * @param requestUserAgent  request.getHeader("user-agent");
     * @param filename 带有中文的文件名字, 呵呵.jpg
     * @return String 编码后的文件名字
     * @throws UnsupportedEncodingException
     */
    public static String downloadChineseFileName(String requestUserAgent, String filename) throws UnsupportedEncodingException {

        if (requestUserAgent.contains("MSIE")) {
            // IE浏览器
            filename = URLEncoder.encode(filename, "utf-8");
            filename = filename.replace("+", " ");
        } else if (requestUserAgent.contains("Firefox")) {
            // 火狐浏览器
            BASE64Encoder base64Encoder = new BASE64Encoder();
            filename = "=?utf-8?B?" + base64Encoder.encode(filename.getBytes("utf-8")) + "?=";
        } else {
            // 其它浏览器
            filename = URLEncoder.encode(filename, "utf-8");
        }
        return filename;
    }

    /**
     * 校验文件是否为图片
     * @param fileName
     * @return
     */
    public static boolean ImageCheck(String fileName){

        File file = new File(FILE_PATH, fileName);
        MimetypesFileTypeMap mtftp = new MimetypesFileTypeMap();
        mtftp.addMimeTypes("png jpg jpeg bmp");
        String mimetype= mtftp.getContentType(file);
        String type = mimetype.split("/")[0];
        return type.equals("image");
    }


}
```

#### FileUploadController.java

```java 
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServletRequest;
import java.io.File;
import java.io.IOException;

@Controller
public class FileUploadController {

    @RequestMapping(value = "/fileupload")
    //参数中定义MultipartFile参数，用于接收页面提交的type=file类型的表单，要求表单名称与参数名相同
    public String fileupload(MultipartFile file,MultipartFile file1,MultipartFile file2, HttpServletRequest request) throws IOException {
        System.out.println("file upload is running ..."+file);
//        MultipartFile参数中封装了上传的文件的相关信息
//        System.out.println(file.getSize());
//        System.out.println(file.getBytes().length);
//        System.out.println(file.getContentType());
//        System.out.println(file.getName());
//        System.out.println(file.getOriginalFilename());
//        System.out.println(file.isEmpty());
        //首先判断是否是空文件，也就是存储空间占用为0的文件
        if(!file.isEmpty()){
            //如果大小在范围要求内正常处理，否则抛出自定义异常告知用户（未实现）
            //获取原始上传的文件名，可以作为当前文件的真实名称保存到数据库中备用
            String fileName = file.getOriginalFilename();
            //设置保存的路径
            String realPath = request.getServletContext().getRealPath("/images");
            //保存文件的方法，指定保存的位置和文件名即可，通常文件名使用随机生成策略产生，避免文件名冲突问题
//            file.transferTo(new File(realPath,file.getOriginalFilename()));


            /*************************** 工具类测试 ***********************************/
            //工具类方式一
//            FileUtil.saveLocal(fileName, file.getBytes());

            //工具类方式二
//            FileUtil.saveLocal(file);

            //工具类方式三
            FileUtil.saveLocal(fileName, file.getInputStream());

        }
        //测试一次性上传多个文件
        if(!file1.isEmpty()){
            String fileName = file1.getOriginalFilename();
            //可以根据需要，对不同种类的文件做不同的存储路径的区分，修改对应的保存位置即可
            String realPath = request.getServletContext().getRealPath("/images");
            file1.transferTo(new File(realPath,file1.getOriginalFilename()));
        }
        if(!file2.isEmpty()){
            String fileName = file2.getOriginalFilename();
            String realPath = request.getServletContext().getRealPath("/images");
            file2.transferTo(new File(realPath,file2.getOriginalFilename()));
        }
        return "page.jsp";
    }
}
```

#### DownFileController.java

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.*;

/**
 * 文件下载
 *
 * @Author: ZJ
 * @Date: 2019/9/2 16:12
 */
@Controller
@RequestMapping("/common/down/")
public class DownFileController {

    /**
     * 文件下载
     *
     * http://localhost:8080/springmvc_fileupload/common/down/downFile?fileName=logo.png
     *
     * http://localhost:8080/springmvc_fileupload/common/down/downFile?fileName=新文件.xlsx
     *
     */
    @GetMapping(value = "/downFile")
    public void down(HttpServletRequest request, HttpServletResponse response) throws IOException{

        try {

            String fileName = request.getParameter("fileName");
            if(StringUtils.isEmpty(fileName)){
                response.getWriter().write("文件名字 fileName 不能为空!");
                return;
            }
            // 获取工程resources资源目录下的文件, 如 banner.txt, 获取其下其他文件夹下的文件需要调整下面代码路径
//            byte[] fileBytes = FileUtil.getStreamBytes(Thread.currentThread().getContextClassLoader().getResourceAsStream(fileName));

            //获取webapp下 文件
            byte[] fileBytes = FileUtil.getStreamBytes(new FileInputStream(new File(request.getServletContext().getRealPath("/images/" + fileName))));

            //设置响应头
            fileName = FileUtil.downloadChineseFileName(request.getHeader("user-agent"), fileName);
            response.setHeader("content-type", request.getServletContext().getMimeType(fileName));
            response.setHeader("content-disposition", "attachment;filename="+fileName);
            //将文件流输出
            response.getOutputStream().write(fileBytes);
        }catch (FileNotFoundException e1){
            response.getWriter().write("文件不存在!");
        }catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```



### 5 Resultful 风格

#### 概述

- 本质：一种软件架构风格

- 核心：面向资源设计的API

- 传统风格：

  ```
  http://localhost/user/getUser?id=3
  ```

- Rest 风格：

  ```
  http://localhost/user/3
  ```

- Restful 是按照Rest风格访问网络资源

- 优点：

  1）隐藏资源的访问行为，通过地址无法得知做的是何种操作

  2）书写简单

- 缺点：

  1）直观性差，不利于理解和阅读

  2）需编写对应的技术说明文档，为后期人员变更维护做考虑，人力成本高。

#### 行为约定

-  RESTful 与HTTP协议操作无关，但是它是按照HTTP的思想进行设计的，所以有必要知道HTTP 

- 正删改查操作-请求方式：

  1）查询：GET

  2）修改：PUT

  3）删除：DELETE

  4）增加：POST

- 以上为风格，可以自主选择和调整



#### 示例代码

-  在使用Spring MVC来做web前端框架时，需要使用标签<mvc:annotation-driven/>，它是启用MVC注解的钥匙。 

-  如果没有使用这个标签，而仅仅是使用<context:component-scan/>标签扫描并注册了相关的注解类到bean中，那么相关的细节功能并不能使用（[@Controller](https://my.oschina.net/u/1774615) @RequestMapping等基本功能除外），比如返回类型的定义，@RestController等。 

- 那么它跟<context:component-scan/>有什么区别呢？

  1）context:component-scan/>标签是告诉Spring 来扫描指定包下的类，并注册被@Component，[@Controller](https://my.oschina.net/u/1774615)，@Service，@Repository等注解标记的组件。

  2）而<mvc:annotation-driven/>是告知Spring，我们启用注解驱动。然后Spring会自动为我们注册   RequestMappingHandlerMapping和RequestMappingHandlerAdapter到 Bean工厂中，来处理我们的请求。

##### spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <mvc:annotation-driven />

    <context:component-scan base-package="com.itheima"/>

</beans>
```

##### page.jsp

```jsp
<%@page pageEncoding="UTF-8" language="java" contentType="text/html;UTF-8" %>
<h1>restful风格请求表单</h1>
<%--切换请求路径为restful风格--%>
<%--GET请求通过地址栏可以发送，也可以通过设置form的请求方式提交--%>
<%--POST请求必须通过form的请求方式提交--%>
<form action="/user/1" method="post">
    <%--当添加了name为_method的隐藏域时，可以通过设置该隐藏域的值，修改请求的提交方式，切换为PUT请求或DELETE请求，但是form表单的提交方式method属性必须填写post--%>
    <%--该配置需要配合HiddenHttpMethodFilter过滤器使用，单独使用无效，请注意检查web.xml中是否配置了对应过滤器--%>
    <input type="text" name="_method" value="PUT"/>
    <input type="submit"/>
</form>
```

##### UserController.java

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

//@Controller
//@ResponseBody
//设置rest风格的控制器
@RestController
//设置公共访问路径，配合下方访问路径使用
@RequestMapping("/user/")
public class UserController {

    //rest风格访问路径完整书写方式
    @RequestMapping("/user/{id}")
    //@RequestMapping("{id}")
    //使用@PathVariable注解获取路径上配置的具名变量，该配置可以使用多次
    public String restLocation(@PathVariable Integer id){
        System.out.println("restful is running ....");
        return "success.jsp";
    }

    //接收GET请求配置方式
    @RequestMapping(value = "{id}",method = RequestMethod.GET)
    //接收GET请求简化配置方式
    @GetMapping("{id}")
    public String get(@PathVariable Integer id){
        System.out.println("restful is running ....get:"+id);
        return "success.jsp";
    }

    //接收POST请求配置方式
    @RequestMapping(value = "{id}",method = RequestMethod.POST)
    //接收POST请求简化配置方式
    @PostMapping("{id}")
    public String post(@PathVariable Integer id){
        System.out.println("restful is running ....post:"+id);
        return "success.jsp";
    }

    //接收PUT请求简化配置方式
    @RequestMapping(value = "{id}",method = RequestMethod.PUT)
    //接收PUT请求简化配置方式
    @PutMapping("{id}")
    public String put(@PathVariable Integer id){
        System.out.println("restful is running ....put:"+id);
        return "success.jsp";
    }

    //接收DELETE请求简化配置方式
    @RequestMapping(value = "{id}",method = RequestMethod.DELETE)
    //接收DELETE请求简化配置方式
    @DeleteMapping("{id}")
    public String delete(@PathVariable Integer id){
        System.out.println("restful is running ....delete:"+id);
        return "success.jsp";
    }

}
```
