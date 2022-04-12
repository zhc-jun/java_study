##  SpringMVC-day01

### **学习目标**：

​    **1）** 基于**xml**  或者  **注解驱动**  实现 springmvc 接收请求和处理响应

​    **2）** 可以接收请求数据到JavaBean 或者 普通形参类型

​    **3）**可以响应JSON数据，并完成页面跳转 + 转发 + 重定向

#### 注意事项：

- 默认return “” 为**请求转发** 跳转 页面
- 请求转发跳转其他 @RequestMapping修饰方法，需要：return "forward:/showPage2";
- 重定向写法：return "redirect:http://www.itcast.cn";
- 如果整个Controller类返回都是 JSON数据，可以使用 @RestController 替换 @Controller + @ResponseBody, 需要开启注解支持。

### 1. SpringMVC简介

- SpringMVC 是一款基于java （Servlet ）实现MVC模式的轻量级框架。

#### MVC

- MVC  （Model View Controller ），一种用于设计创建Web应用程序表现层的模式
- Model （模型）：数据模型，用于数据的：逻辑处理，数据封装，数据访问
- View（视图）：页面视图，用于展示数据给客户端
- Controller（控制器）：处理用户交互的调度器，用于根据用户需求处理程序逻辑事务前后数据的完整性必须保持一致 

 	        ◆ Servlet

​	         ◆ SpringMVC

​			 ◆ Struts2

​			 ◆ Play

#### 三层架构

**表示层:**

-  负责数据的展示和与客户端做交互（Controller + View）

**不可重复读**:  

- 事物A读到了事物B修改后提交的数据，导致事物A两次执行一模一样的查询SQL得到的数据是不一样的，这种现象称之为： “不可重复读”。

  

**业务层：**

- 负责业务逻辑的处理


**数据层：**

- 负责与数据库打交道，增删改查。

#### SpringMVC优点

- 使用简单
- 性能突出（相比Struts2）框架
- 灵活性强



### 2. 入门案例：

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>spring.base</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <name>spring.base Maven Webapp</name>
    <!-- FIXME change it to the project's website -->
    <url>http://www.example.com</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- servlet3.1规范的坐标 -->
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
        <!--spring web的坐标-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <!--springmvc的坐标-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>

    </dependencies>

    <!--构建-->
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

#### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
		  http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">

    <servlet>
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath*:spring-mvc.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>

```

#### spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    <!--扫描加载所有的控制类类-->
    <context:component-scan base-package="com.itheima"/>

</beans>
```

#### UserController.java

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
//设置当前类为Spring的控制器类
@Controller
public class UserController {
    //设定当前方法的访问映射地址
    @RequestMapping("/save")
    //设置当前方法返回值类型为String，用于指定请求完成后跳转的页面
    public String save(){
        System.out.println("user mvc controller is running ...");
        //设定具体跳转的页面
        return "success.jsp";
    }
}
```

#### success.jsp

```jsp
<%@page pageEncoding="UTF-8" language="java" contentType="text/html;UTF-8" %>
<html>
<body>
<h1>第一个spring-mvc页面</h1>
</body>
</html>
```



### 3. SpringMVC架构图

- DispatcherServlet ：前端控制器，是整体流程控制的中心，由其调用其他组件处理用户的请求，有效的降低了组件间的耦合性

-  HandlerMapping Handler ：处理器映射，负责根据用户请求找到对应具体的Handler处理器 

- Handler：处理器，业务处理的核心类，通常由开发编写，描述 具体的业务

- HandlerAdapter：处理适配器，通过它对处理器进行执行

- View Resolver：视图解析器，将处理结果生成View视图

- View：视图，最终产出结果，常用视图如jsp, html.

  

### 4 接触配置（注解驱动）

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>spring.base.config</artifactId>
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
    </dependencies>

    <!--构建-->
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

#### SpringMVCConfiguration.java

- @EnableWebMVC 注解用来开启Web MVC的配置支持。也就是写Spring MVC时的时候会用到。 
-  不使用@EnableWebMvc注解也是可以实现配置Webmvc，只需要将配置类继承于WebMvcConfigurationSupport类即可 

```java
package com.itheima.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.stereotype.Controller;
import org.springframework.web.servlet.config.annotation.*;

@Configuration
@ComponentScan(value = "com.itheima",includeFilters =
    @ComponentScan.Filter(type=FilterType.ANNOTATION,classes = {Controller.class})
    )

@EnableWebMvc
public class SpringMVCConfiguration implements WebMvcConfigurer{
//public class SpringMVCConfiguration extends WebMvcConfigurationSupport {
    //注解配置放行指定资源格式
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/img/**").addResourceLocations("/img/");
        registry.addResourceHandler("/js/**").addResourceLocations("/js/");
        registry.addResourceHandler("/css/**").addResourceLocations("/css/");
    }

    //注解配置通用放行资源的格式
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();;
    }
}
```

#### **ServletContainersInitConfig**.java

```java 
package com.itheima.config;

import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.support.AnnotationConfigWebApplicationContext;
import org.springframework.web.filter.CharacterEncodingFilter;
import org.springframework.web.servlet.support.AbstractDispatcherServletInitializer;

import javax.servlet.DispatcherType;
import javax.servlet.FilterRegistration;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import java.util.EnumSet;

public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {
    //创建Servlet容器时，使用注解的方式加载SPRINGMVC配置类中的信息，并加载成WEB专用的ApplicationContext对象
    //该对象放入了ServletContext范围，后期在整个WEB容器中可以随时获取调用
    @Override
    protected WebApplicationContext createServletApplicationContext() {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringMVCConfiguration.class);
        return ctx;
    }

    //注解配置映射地址方式，服务于SpringMVC的核心控制器DispatcherServlet
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    //乱码处理作为过滤器，在servlet容器启动时进行配置，相关内容参看Servlet零配置相关课程
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        super.onStartup(servletContext);
        CharacterEncodingFilter cef = new CharacterEncodingFilter();
        cef.setEncoding("UTF-8");
        FilterRegistration.Dynamic registration = servletContext.addFilter("characterEncodingFilter", cef);
        registration.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST,DispatcherType.FORWARD,DispatcherType.INCLUDE),false,"/*");
    }
}
```

#### springmvc.jsp

```jsp
<%@page pageEncoding="UTF-8" language="java" contentType="text/html;UTF-8" %>
<html>
<body>
<img src="img/logo.png">
</body>
</html>
```



### 5 请求参数

#### 5.1 基础配置

##### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

  <filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath*:spring-mvc.xml</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

</web-app>
```

##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>springmvc.request</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <dependencies>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.9.RELEASE</version>
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

##### spring-mvc.xml

- org.springframework.web.servlet.view.InternalResourceViewResolver 可以帮助我们规定j视图文件（jsp/html）所在目录

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

     
    <context:component-scan base-package="com.itheima"/>
    
    <!--    自定义格式转换器案例，配合不带@DateTimeFormat的requestParam11使用-->
    <!--开启注解驱动，加载自定义格式化转换器对应的类型转换服务-->
    <mvc:annotation-driven conversion-service="conversionService"/>
    <!--自定义格式化转换器-->
    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <!--覆盖格式化转换器定义规则，该规则是一个set集合，对格式化转换器来说是追加和替换的思想，而不是覆盖整体格式化转换器-->
        <property name="formatters">
            <set>
                <!--具体的日期格式化转换器-->
                <bean class="org.springframework.format.datetime.DateFormatter">
                    <!--具体的规则，不具有通用性，仅适用于当前的日期格式化转换器-->
                    <property name="pattern" value="yyyy-MM-dd"/>
                </bean>
            </set>
        </property>
    </bean>

</beans>
```



####  5.2 普通类型请求参数

- 方法传递普通类型参数，数量任意，类型必须匹配
- 匹配不对状态码 400

```java
@Controller
@RequestMapping("/user")
public class UserController {
    
    //http://localhost/user/requestParam0/zhangsan?age=23
    //这个URL里面，服务器想获取问题编号101566，因为这个参数直接包含在请求路径部分中，所以代码中用的应该是@PathVariable
    @RequestMapping(value="/requestParam0/{name}")
    public String requestParam0(@PathVariable String name, int age){
        System.out.println(name+","+age);
        return "page.jsp";
    }

    //方法传递普通类型参数，数量任意，类型必须匹配
    //http://localhost/user/requestParam1?name=itheima
    //http://localhost/user/requestParam1?name=itheima&age=14
    @RequestMapping("/requestParam1")
    public String requestParam1(String name,int age){
        System.out.println(name+","+age);
        return "page.jsp";
    }

    //方法传递普通类型参数，使用@RequestParam参数匹配URL传参中的参数名称与方法形参名称
    //http://localhost/user/requestParam2?userName=Jock
    @RequestMapping("/requestParam2")
    public String requestParam2(@RequestParam(value = "userName",required = true) String name){
        System.out.println(name);
        return "page.jsp";
    }
    
}
```

#### 5.3 JavaBean请求参数

- /requestParam3 常用，其他玩法不常用

```java
@Controller
@RequestMapping("/user")
public class UserController {

    //方法传递POJO类型参数，URL地址中的参数作为POJO的属性直接传入对象
    //http://localhost/user/requestParam3?name=Jock&age=39
    @RequestMapping("/requestParam3")
    public String requestParam3(User user){
        System.out.println(user);
        return "page.jsp";
    }

    //当方法参数中具有POJO类型参数与普通类型参数嘶，URL地址传入的参数不仅给POJO对象属性赋值，也给方法的普通类型参数赋值
    //http://localhost/user/requestParam4?name=Jock&age=39
    @RequestMapping("/requestParam4")
    public String requestParam4(User user,int age){
        System.out.println("user="+user+",age="+age);
        return "page.jsp";
    }

    //使用对象属性名.属性名的对象层次结构可以为POJO中的POJO类型参数属性赋值
    //http://localhost/user/requestParam5?address.city=beijing
    @RequestMapping("/requestParam5")
    public String requestParam5(User user){
        System.out.println(user.getAddress().getCity());
        return "page.jsp";
    }
    
}
```

#### 5.4 数组&集合请求参数

- String[] 和 List<String> 多用于批量删除
- /requestParam7 和 /requestParam8 了解即可，用的不多，其他练一练

```java
@Controller
@RequestMapping("/user")
public class UserController {

    //通过URL地址中同名参数，可以为POJO中的集合属性进行赋值，集合属性要求保存简单数据
    //http://localhost/user/requestParam6?nick=Jock1&nick=Jockme&nick=zahc
    @RequestMapping("/requestParam6")
    public String requestParam6(User user){
        System.out.println(user);
        return "page.jsp";
    }

    //POJO中List对象保存POJO的对象属性赋值，使用[数字]的格式指定为集合中第几个对象的属性赋值
    //http://localhost/user/requestParam7?addresses[0].city=beijing&addresses[1].province=hebei
    @RequestMapping("/requestParam7")
    public String requestParam7(User user){
        System.out.println(user.getAddresses());
        return "page.jsp";
    }
    //POJO中Map对象保存POJO的对象属性赋值，使用[key]的格式指定为Map中的对象属性赋值
    //http://localhost/user/requestParam8?addressMap['job'].city=beijing&addressMap['home'].province=henan
    @RequestMapping("/requestParam8")
    public String requestParam8(User user){
        System.out.println(user.getAddressMap());
        return "page.jsp";
    }

    //方法传递普通类型的数组参数，URL地址中使用同名变量为数组赋值
    //http://localhost/user/requestParam9?nick=Jockme&nick=zahc
    @RequestMapping("/requestParam9")
    public String requestParam9(String[] nick){
        System.out.println(nick[0]+","+nick[1]);
        return "page.jsp";
    }

    //方法传递保存普通类型的List集合时，无法直接为其赋值，需要使用@RequestParam参数对参数名称进行转换
    //http://localhost/user/requestParam10?nick=Jockme&nick=zahc
    @RequestMapping("/requestParam10")
    public String requestParam10(@RequestParam("nick") List<String> nick){
        System.out.println(nick);
        return "page.jsp";
    }
    
}
```

##### Uesr.java

```java
package domain;

import java.util.List;
import java.util.Map;

public class User {
    
    private String name;
    private Integer age;

    private Address address;

    private List<String> nick;

    private List<Address> addresses;

    public Map<String,Address> addressMap;
    

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", address=" + address +
                ", nick=" + nick +
                ", addresses=" + addresses +
                ", addressMap=" + addressMap +
                '}';
    }

    public List<Address> getAddresses() {
        return addresses;
    }

    public Map<String, Address> getAddressMap() {
        return addressMap;
    }

    public void setAddressMap(Map<String, Address> addressMap) {
        this.addressMap = addressMap;
    }

    public void setAddresses(List<Address> addresses) {
        this.addresses = addresses;
    }

    public List<String> getNick() {
        return nick;
    }

    public void setNick(List<String> nick) {
        this.nick = nick;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

}
```



#### 5.5 类型转换器

##### spring-mvc.xml

```xml
<!--    自定义格式转换器案例，配合不带@DateTimeFormat的requestParam11使用-->
<!--开启注解驱动，加载自定义格式化转换器对应的类型转换服务-->
<mvc:annotation-driven conversion-service="conversionService"/>
<!--自定义格式化转换器-->
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <!--覆盖格式化转换器定义规则，该规则是一个set集合，对格式化转换器来说是追加和替换的思想，而不是覆盖整体格式化转换器-->
    <property name="formatters">
        <set>
            <!--具体的日期格式化转换器-->
            <bean class="org.springframework.format.datetime.DateFormatter">
                <!--具体的规则，不具有通用性，仅适用于当前的日期格式化转换器-->
                <property name="pattern" value="yyyy-MM-dd"/>
            </bean>
        </set>
    </property>
</bean>
```

##### UserController.java

- http://localhost/user/requestParam11?date=1999-09-09
- http://localhost/user/requestParam12?date=1999-09-09
- @DateTimeFormat(pattern = "yyyy-MM-dd")   也可以修饰在JavaBean属性上

```java 
@Controller
@RequestMapping("/user")
public class UserController {

    //数据类型转换，使用自定义格式化器或@DateTimeFormat注解设定日期格式
    //两种方式都依赖springmvc的注解启动才能运行
    //http://localhost/user/requestParam11?date=1999-09-09
    @RequestMapping("/requestParam11")
    public String requestParam11(@DateTimeFormat(pattern = "yyyy-MM-dd") Date date){
        System.out.println(date);
        return "page.jsp";
    }

    //数据类型转换，使用自定义类型转换器，需要配置后方可使用
    //http://localhost/user/requestParam12?date=1999-09-09
    @RequestMapping("/requestParam12")
    public String requestParam12(Date date){
        System.out.println(date);
        return "page.jsp";
    }
    
}
```

#### 5.6 自定义转换器（了解）

##### spring-mvc.xml

```xml
<!--    自定义格数据类型转换器案例，配合requestParam12使用-->
<!--    自定义格数据类型转换器案例，配合requestParam12使用-->
<mvc:annotation-driven conversion-service="conversionService"/>
<!--自定义类型转换器-->
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
    <!--覆盖类型转换器定义规则，该规则是一个set集合，对类型转换器来说是追加和替换的思想，而不是覆盖整体格式化转换器-->
    <property name="converters">
        <set>
            <!--添加自定义的类型转换器，会根据定义的格式覆盖系统中默认的格式-->
            <!--当前案例中是将String转换成Date的类型转换器进行了自定义，所以添加后，系统中原始自带的String——>Date的类型转换器失效-->
            <bean class="com.itheima.converter.MyDateConverter"/>
        </set>
    </property>
</bean>
```

##### MyDateConverter.java

```java
package com.itheima.converter;

import org.springframework.core.convert.converter.Converter;

import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

//自定义类型转换器，实现Converter接口，接口中指定的泛型即为最终作用的条件
//本例中的泛型填写的是String，Date，最终出现字符串转日期时，该类型转换器生效
public class MyDateConverter implements Converter<String, Date> {
    //重写接口的抽象方法，参数由泛型决定
    public Date convert(String source) {
        DateFormat df = new SimpleDateFormat("yyyy-MM-dd");
        Date date = null;
        //类型转换器无法预计使用过程中出现的异常，因此必须在类型转换器内部捕获，不允许抛出，框架无法预计此类异常如何处理
        try {
            date = df.parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}
```

##### UserController.java

- http://localhost/user/requestParam11?date=1999-09-09
- http://localhost/user/requestParam12?date=1999-09-09

```java 
@Controller
@RequestMapping("/user")
public class UserController {

    //数据类型转换，使用自定义格式化器或@DateTimeFormat注解设定日期格式
    //两种方式都依赖springmvc的注解启动才能运行
    //http://localhost/user/requestParam11?date=1999-09-09
    @RequestMapping("/requestParam11")
    public String requestParam11(@DateTimeFormat(pattern = "yyyy-MM-dd") Date date){
        System.out.println(date);
        return "page.jsp";
    }

    //数据类型转换，使用自定义类型转换器，需要配置后方可使用
    //http://localhost/user/requestParam12?date=1999-09-09
    @RequestMapping("/requestParam12")
    public String requestParam12(Date date){
        System.out.println(date);
        return "page.jsp";
    }
    
}
```

##### 

### 6 请求映射

#### @RequestMapping  

- 功能性属性：value
- 校验性属性：
  ◼ method ◼ params ◼ headers ◼ consumes ◼ produces

```java
@RequestMapping(
    value="/requestURL3",            // 设定请求路径，与 path 属性、 value 属性相同
    method = RequestMethod.GET,      // 设定请求方式
    params = "name",                 // 设定请求参数条件
    headers = "content-type=text/*", // 设定请求消息头条件
    consumes = "text/*",             // 用于指定可以接收的请求正文类型（ MIME 类型）
    produces = "text/*"              // 用于指定可以生成的响应正文类型（ MIME 类型）
)
public String requestURL3() {
	return "/page.jsp";
}
```

#### 方式一

- 访问 webapp 目录下 page.jsp

- 范例：

  ```java
  @Controller
  public class UserController {
      
      @RequestMapping("/requestURL1")
      public String requestURL1() {
      	return "page.jsp";
      }
  }
  ```

#### 方式二

- 访问 webapp/user/目录下 page.jsp

- 范例：

  ```java
  @Controller
  @RequestMapping("/user")
  public class UserController {
      
      @RequestMapping("/requestURL2")
      public String requestURL2() {
      	return "page.jsp";
      }
  }
  ```



### 7 页面跳转 

- 当处理器的返回值类型为String时，可以通过return “返回值”来完成页面的跳转，也可请求转发或者重定向。

#### 7.1 三种跳转资源方式

- 默认return “” 为**请求转发** 跳转 页面
- 请求转发跳转其他 @RequestMapping修饰方法，需要：return "forward:/showPage2";
- 重定向写法：return "redirect:http://www.itcast.cn";

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class UserController {
    
    //直接跳转页面
    @RequestMapping("/showPage")
    public String showPage() {
        System.out.println("user mvc controller is running ...");
        return "page.jsp";
    }

    //forward:page.jsp转发访问，支持访问WEB-INF下的页面
    @RequestMapping("/showPage1")
    public String showPage1() {
        System.out.println("user mvc controller is running ...");
        return "forward:/showPage2";
    }

    //redirect:page.jsp重定向访问，不支持访问WEB-INF下的页面
    @RequestMapping("/showPage2")
    public String showPage2() {
        System.out.println("user mvc controller is running ...");
        return "redirect:http://www.itcast.cn";
//        return "redirect:/WEB-INF/page/page.jsp";
    }

}
```



#### 7.2 页面访问-固定位置

- 静态页面所在的位置一般都是固定的，我们可以利用spring的配置来进行设置

##### spring-mvc.xml

```xml
<!--设定页面加载的前缀后缀，仅适用于默认形式，不适用于手工标注转发或重定向的方式-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/page/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

##### UserController.java

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class UserController {

    //页面简化配置格式，使用前缀+页面名称+后缀的形式进行，类似于字符串拼接
    @RequestMapping("/showPage3")
    public String showPage3() {
        System.out.println("user mvc controller is running ...");
        return "page";
    }

    //页面简化配置格式不支持书写forward
    @RequestMapping("/showPage4")
    public String showPage4() {
        System.out.println("user mvc controller is running ...");
        return "forward:page";
    }

}

```



#### 7.3 带数据跳转JSP

##### 方式一

- 利用HttpServletRequest对象完成域中数据共享

###### BookController.java

```java
package com.itheima.controller;

import com.itheima.domain.Book;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;

@Controller
public class BookController {

    //使用原生request对象传递参数
    @RequestMapping("/showPageAndData1")
    public String showPageAndData1(HttpServletRequest request) {
        request.setAttribute("name","itheima");
        return "page";
    }
   
}
```

##### 方式二

- Model 形参传递参数

###### BookController.java

```java
package com.itheima.controller;

import com.itheima.domain.Book;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;

@Controller
public class BookController {

    //使用Model形参传递参数
    @RequestMapping("/showPageAndData2")
    public String showPageAndData2(Model model) {
        //添加数据的方式，key对value
        model.addAttribute("name","Jock");
        Book book  = new Book();
        book.setName("SpringMVC入门案例");
        book.setPrice(66.66d);
        //添加数据的方式，key对value
        model.addAttribute("book",book);
        return "page";
    }

}
```

##### 方式三

- ModelAndView形参传递参数，该对象还封装了页面信息
- modelAndView.setViewName("page") 等同于 返回值为String时， return “page”
- modelAndView.setViewName("redirect:page") 等同于 返回值为String时， return  “redirect:page”

###### BookController.java

```java
package com.itheima.controller;

import com.itheima.domain.Book;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;

@Controller
public class BookController {
    
    //使用ModelAndView形参传递参数，该对象还封装了页面信息
    @RequestMapping("/showPageAndData3")
    public ModelAndView showPageAndData3(ModelAndView modelAndView){
        //ModelAndView mav = new ModelAndView();    替换形参中的参数
        Book book  = new Book();
        book.setName("SpringMVC入门案例");
        book.setPrice(66.66d);

        //添加数据的方式，key对value
        modelAndView.addObject("book",book);
        //添加数据的方式，key对value
        modelAndView.addObject("name","Jockme");
        //设置页面的方式，该方法最后一次执行的结果生效
        modelAndView.setViewName("page");
        //返回值设定成ModelAndView对象
        return modelAndView;
    }

}
```

##### 

### 8 响应数据

#### 8.1 Response对象

- 用的不多，多用于下载文件

##### AccountController.java

```java
package com.itheima.controller;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.itheima.domain.Book;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

@Controller
public class AccountController {

    //使用原生response对象响应数据
    @RequestMapping("/showData1")
    public void showData1(HttpServletResponse response) throws IOException {
        response.getWriter().write("message");
    }

    //使用@ResponseBody将返回的结果作为响应内容，而非响应的页面名称
    @RequestMapping("/showData2")
    @ResponseBody
    public String showData2(){
        return "{'name':'Jock'}";
    }

    //使用jackson进行json数据格式转化
    @RequestMapping("/showData3")
    @ResponseBody
    public String showData3() throws JsonProcessingException {
        Book book  = new Book();
        book.setName("SpringMVC入门案例");
        book.setPrice(66.66d);

        ObjectMapper om = new ObjectMapper();
        return om.writeValueAsString(book);
    }

}
```



#### 8.2 响应json

- @ResponseBody 使用较多，需要增加额外注解驱动支持
- @RestController注解相当于@ResponseBody ＋ @Controller合在一起的作用。 
- 需要手动pom.xml 导入 JackJson 的jar包

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

    <context:component-scan base-package="com.itheima"/>

    <!--设定页面加载的前缀后缀，仅适用于默认形式，不适用于手工标注转发或重定向的方式-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/page/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--开启springmvc注解驱动，对@ResponseBody的注解进行格式增强，追加其类型转换的功能，具体实现由MappingJackson2HttpMessageConverter进行-->
    <mvc:annotation-driven/>

</beans>
```

##### AccountController.java

```java
package com.itheima.controller;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.itheima.domain.Book;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

@Controller
//@RestController注解相当于@ResponseBody ＋ @Controller合在一起的作用。
//@RestController
public class AccountController {

    //使用@ResponseBody将返回的结果作为响应内容，而非响应的页面名称
    @RequestMapping("/showData2")
    @ResponseBody
    public String showData2(){
        return "{'name':'Jock'}";
    }

    //使用jackson进行json数据格式转化
    @RequestMapping("/showData3")
    @ResponseBody
    public String showData3() throws JsonProcessingException {
        Book book  = new Book();
        book.setName("SpringMVC入门案例");
        book.setPrice(66.66d);

        ObjectMapper om = new ObjectMapper();
        return om.writeValueAsString(book);
    }

    //使用SpringMVC注解驱动，对标注@ResponseBody注解的控制器方法进行结果转换，由于返回值为引用类型，自动调用jackson提供的类型转换器进行格式转换
    @RequestMapping("/showData4")
    @ResponseBody
    public Book showData4() {
        Book book  = new Book();
        book.setName("SpringMVC入门案例");
        book.setPrice(66.66d);
        return book;
    }

    //转换集合类型数据
    @RequestMapping("/showData5")
    @ResponseBody
    public List showData5() {
        Book book1  = new Book();
        book1.setName("SpringMVC入门案例");
        book1.setPrice(66.66d);

        Book book2  = new Book();
        book2.setName("SpringMVC入门案例");
        book2.setPrice(66.66d);

        ArrayList al = new ArrayList();
        al.add(book1);
        al.add(book2);
        return al;
    }
}
```

##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>springmvc-response</artifactId>
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
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <!--springmvc的坐标-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>

        <!--json相关坐标3个-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.9.0</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.9.0</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>2.9.0</version>
        </dependency>


    </dependencies>

    <!--构建-->
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

#### 8.3 注解驱动

- @EnableWebMvc 可以替代 如下标签：

- ```xml
  <!--开启springmvc注解驱动，对@ResponseBody的注解进行格式增强-->
  <mvc:annotation-driven/>
  ```

**示例代码:**

```java
@Configuration
@ComponentScan("com.itheima")
@EnableWebMvc
public class SpringMvcConfiguration {
}
```



### 9  Servlet相关接口替换方案

#### 9.1 请求&响应&session

- HttpServletRequest
- HttpServletResponse
- HttpSession

##### UserController.java

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

@Controller
public class UserController {

    //获取request,response,session对象的原生接口
    @RequestMapping("/servletApi")
    public String servletApi(HttpServletRequest request, HttpServletResponse response, HttpSession session){
        System.out.println(request);
        System.out.println(response);
        System.out.println(session);
        return "page";
    }

}
```

#### 9.2 请求Head头数据获取

- @RequestHeader  处理器类方法形参，作用：绑定头数据与处理器方法形参之间的关系
- 也可以利用HttpServletRequest 对象来获取

##### UserController.java

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
public class UserController {

    //获取head数据的快捷操作方式
    @RequestMapping("/headApi")
    public String headApi(@RequestHeader("Accept-Encoding") String headMsg){
        System.out.println(headMsg);
        return "page";
    }

}
```

#### 9.3 Cookie数据获取

-  @CookieValue   处理器类方法形参，作用：绑定请求头中Cookie key 的数据与处理器方法形参之间的关系
- 也可以利用HttpServletRequest 对象来获取

##### UserController.java

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
public class UserController {

    //获取cookie数据的快捷操作方式
    @RequestMapping("/cookieApi")
    public String cookieApi(@CookieValue("JSESSIONID") String jsessionid){
        System.out.println(jsessionid);
        return "page";
    }

}
```

#### 9.4 Session数据获取

-  @CookieValue   处理器类方法形参，作用：绑定请求头中Cookie key 的数据与处理器方法形参之间的关系

##### UserController.java

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
//设定当前类中名称为age和gender的变量放入session范围，不常用，了解即可
@SessionAttributes(names = {"age","gender"})
public class UserController {

    //测试用方法，为下面的试验服务，用于在session中放入数据
    @RequestMapping("/setSessionData")
    public String setSessionData(HttpSession session){
        session.setAttribute("name","itheima");
//        session.setAttribute("age","23");
//        session.setAttribute("gender","男");
        return "page";
    }

    //获取session数据的快捷操作方式
    @RequestMapping("/sessionApi")
    public String sessionApi(@SessionAttribute("name") String name,
                             @SessionAttribute("age") int age,
                             @SessionAttribute("gender") String gender){
        System.out.println(name);
        System.out.println(age);
        System.out.println(gender);
        return "page";
    }

}
```

