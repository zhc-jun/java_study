##  SpringMVC-day03

### **学习目标**：

​    **1）** 使用@Valid @Validated 对表单数据进行校验 + 配合全局异常处理器，做数据返回。



### 1. 校验框架

- Hibernate 框架中包含一套独立的校验框架：hibernate-validator

- ```xml
  <!--导入校验框架实现技术-->
  <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-validator</artifactId>
      <version>5.1.0.Final</version>
      <!--6.1 版本异常: java.lang.NoClassDefFoundError: javax/el/ELManager-->
      <!-- <version>6.1.0.Final</version> -->
  </dependency>
  ```

- **注意：**

  1）tomcat7  选择版本  hibernate-validator 5.*.*.Final 

  2）tomcat8.5及以上 选择版本  hibernate-validator 6.*.*.Final

  <font color=red> 3）hibernate-validator 5.*.*.Final 此版本 @Valid 注解不能单独使用，必须配合@Validated一起使用，否则不生效</font>

  <font color=red> 4）hibernate-validator 6.*.*.Final 此版本 @Valid 注解可以单独使用</font>

#### 1.1 开启校验

##### @Validated

- 可以修饰类 和 方法 以及方法形参
- 支持校验分组
- 不能修饰某个类的成员变量

##### @Valid 

- 多用于修饰 控制器方法形参处
- 支持javabean嵌套形式下的校验，注意： 用在控制器方法入参上无法单独提供嵌套验证功能，还需在javabean类型的成员变量上使用 @Valid 才能完成嵌套形式下校验

##### 示例代码

- 如下代码只为演示，如需运行，请自行提供getter/setter 方法等

  

- | 表单数据            | @NotNull | @NotEmpty | @NotBlank |
  | ------------------- | -------- | --------- | --------- |
  | String s = null;    | false    | false     | false     |
  | String s = “”;      | true     | false     | false     |
  | String s = “     “; | true     | true      | false     |
  | String s = “Jock”;  | true     | true      | true      |

```java
package com.itheima.domain;

import com.itheima.controller.groups.GroupA;
import org.hibernate.validator.constraints.Range;

import javax.validation.Valid;
import javax.validation.constraints.*;

public class Employee{
    
    //设定校验器，设置校验不通过对应的消息
    @NotBlank(message = "姓名不能为空")
    private String name;//员工姓名

    //一个属性可以添加多个校验器
    @NotNull(message = "请输入您的年龄")
    @Max(value = 60,message = "年龄最大值不允许超过60岁")
    @Min(value = 18,message = "年龄最小值不允许低于18岁")
    private Integer age;//员工年龄
    
    @Pattern(regexp = "1[3|4|5|7|8][0-9]\\d{8}", message = "手机号格式错误")
    @NotBlank(message = "必填项")
    private String phone;

    @NotBlank(message = "必填项")
    @Length(min = 0, max = 6, message = "验证码最大长度6")
    private String smsCode;
 
}
```



#### 1.2 分组校验

- @Validated({GroupA.class}) 只有该注解能够完成分组校验

- 同一个模块中，javabean类中同一个成员变量可能校验规则不同，此时可以给该成员变量设置分组来区别校验
- 如下实例代码 @Max  @Min 校验注解无法生效，因为不在 GroupA.class 组内

##### Employee.java

```javascript
package com.itheima.domain;

import com.itheima.controller.groups.GroupA;
import org.hibernate.validator.constraints.Range;

import javax.validation.Valid;
import javax.validation.constraints.*;

public class Employee{
    //设定校验器，设置校验不通过对应的消息，设定所参与的校验组
    @NotBlank(message = "姓名不能为空",groups = {GroupA.class})
    private String name;//员工姓名

    //一个属性可以添加多个校验器
    @NotNull(message = "请输入您的年龄",groups = {GroupA.class})
    @Max(value = 60,message = "年龄最大值不允许超过60岁")
    @Min(value = 18,message = "年龄最小值不允许低于18岁")
    private Integer age;//员工年龄

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

    @Override
    public String toString() {
        return "Employee{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", address=" + address +
                '}';
    }
}

```

##### EmployeeController.java

```java
package com.itheima.controller;

import com.itheima.controller.groups.GroupA;
import com.itheima.domain.Employee;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.Errors;
import org.springframework.validation.FieldError;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.validation.Valid;
import java.util.List;

@Controller
public class EmployeeController {

    @RequestMapping(value = "/addemployee")
    public String addEmployee(@Validated({GroupA.class}) Employee employee, Errors errors, Model m){
        if(errors.hasErrors()){
            List<FieldError> fieldErrors = errors.getFieldErrors();
            System.out.println(fieldErrors.size());
            for(FieldError error : fieldErrors){
                System.out.println(error.getField());
                System.out.println(error.getDefaultMessage());
                m.addAttribute(error.getField(),error.getDefaultMessage());
            }
            return "addemployee.jsp";
        }
        return "success.jsp";
    }
    
}
```

##### GroupA.java

```java
package com.itheima.controller.groups;
//用于设定分组校验中的组名，当前接口仅提供字节码，用于识别
public interface GroupA {
}
```

#### 1.3 嵌套校验

- 当javabean多个对象之间存在嵌套规则时，也可以完成成员变量校验
- @Valid 只有该注解可以完成嵌套校验，注意：@Valid 要修饰在两处，移除Controller方法参数之前，一处在javabean 类型成员变量之上

##### Employee.java

```javascript
package com.itheima.domain;

import com.itheima.controller.groups.GroupA;
import org.hibernate.validator.constraints.Range;

import javax.validation.Valid;
import javax.validation.constraints.*;

public class Employee{
 
    //实体类中的引用类型通过标注@Valid注解，设定开启当前引用类型字段中的属性参与校验
    @Valid
    private Address address;

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }

}

```

##### Address.java

```java
package com.itheima.domain;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;
//嵌套校验的实体中，对每个属性正常添加校验规则即可
public class Address {

    @NotBlank(message = "请输入省份名称")
    private String provinceName;//省份名称

    @NotBlank(message = "请输入城市名称")
    private String cityName;//城市名称

    @NotBlank(message = "请输入详细地址")
    private String detail;//详细住址

    @NotBlank(message = "请输入邮政编码")
    @Size(max = 6,min = 6,message = "邮政编码由6位组成")
    private String zipCode;//邮政编码

}
```

##### EmployeeController.java

```java
package com.itheima.controller;

import com.itheima.controller.groups.GroupA;
import com.itheima.domain.Employee;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.Errors;
import org.springframework.validation.FieldError;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.validation.Valid;
import java.util.List;

@Controller
public class EmployeeController {

    @RequestMapping(value = "/addemployee2")
    //使用@Valid开启校验，使用@Validated也可以开启校验
    //Errors对象用于封装校验结果，如果不满足校验规则，对应的校验结果封装到该对象中，包含校验的属性名和校验不通过返回的消息
    public String addEmployee2(@Valid Employee employee, Errors errors, Model m){
        //判定Errors对象中是否存在未通过校验的字段
        if(errors.hasErrors()){
            //获取所有未通过校验规则的信息
            List<FieldError> fieldErrors = errors.getFieldErrors();
            System.out.println(fieldErrors.size());
            for(FieldError error : fieldErrors){
                System.out.println(error.getField() + "：" + error.getDefaultMessage());
                //将校验结果信息添加到Model对象中，用于页面显示，后期实际开发中无需这样设定，返回json数据即可
                m.addAttribute(error.getField(),error.getDefaultMessage());
            }
            //当出现未通过校验的字段时，跳转页面到原始页面，进行数据回显
            return "addemployee.jsp";
        }
        return "success.jsp";
    }
    
}
```



#### 1.4 异常处理器解耦校验（扩展）

- Controller 中方法，不能加 Errors  errors 形参，否则处理器不生效

##### ProjectExceptionAdvice.java

```java
package com.itheima.exception;

import org.springframework.stereotype.Component;
import org.springframework.validation.BindException;
import org.springframework.validation.Errors;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.servlet.ModelAndView;

import java.util.HashMap;
import java.util.Map;

/**
 * 此处理器一般返回都是JSON格式数据, 跳转jsp页面话无法区别
 */
@Component
@ControllerAdvice
public class ProjectExceptionAdvice {

    /**
     * 拦截 检验异常
     * org.springframework.validation.BindException
     * @param errors
     * @return
     */
/*    @ExceptionHandler({BindException.class})
    protected ModelAndView bindException(Errors errors) {

        ModelAndView modelAndView = new ModelAndView();
        //指定错误页面的模板页
        modelAndView.setViewName("addemployee.jsp");

        for(FieldError error : errors.getFieldErrors()){
            System.out.println(error.getField() + ": " + error.getDefaultMessage());
            modelAndView.addObject(error.getField(),error.getDefaultMessage());
        }
        return modelAndView;
    }*/

    /**
     * 拦截 检验异常
     * org.springframework.validation.BindException
     *
     * @param errors
     * @return
     */
    @ExceptionHandler({BindException.class})
    @ResponseBody
    protected Map bindException(Errors errors) {

        Map map = new HashMap();
        for(FieldError error : errors.getFieldErrors()){
            System.out.println(error.getField() + ": " + error.getDefaultMessage());
            map.put(error.getField(), error.getDefaultMessage());
        }
        return map;
    }

}
```

##### EmployeeController.java

```js
package com.itheima.controller;

import com.itheima.controller.groups.GroupA;
import com.itheima.domain.Employee;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.Errors;
import org.springframework.validation.FieldError;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.validation.Valid;
import java.util.List;

@Controller
public class EmployeeController {

    @RequestMapping(value = "/addemployee")
    public String addEmployee(@Validated({GroupA.class}) Employee employee, Model m){
        
        return "success.jsp";
    }

    @RequestMapping(value = "/addemployee2")
    //hibernate-validator 5.*.*.Final 此版本 @Valid 注解不能单独使用，必须配合@Validated一起使用，否则不生效
    //Errors对象用于封装校验结果，如果不满足校验规则，对应的校验结果封装到该对象中，包含校验的属性名和校验不通过返回的消息
    public String addEmployee2(@Valid Employee employee, Model m){
        return "success.jsp";
    }
}

```

##### Employee.java

```java
package com.itheima.domain;

import com.itheima.controller.groups.GroupA;
import org.hibernate.validator.constraints.Range;

import javax.validation.Valid;
import javax.validation.constraints.*;

public class Employee{
    
    //设定校验器，设置校验不通过对应的消息，设定所参与的校验组
    @NotBlank(message = "姓名不能为空Blank",groups = {GroupA.class})
    @NotEmpty(message = "姓名不能为空Empty")
    private String name;//员工姓名

    //一个属性可以添加多个校验器
    @NotNull(message = "请输入您的年龄",groups = {GroupA.class})
    @Max(value = 60,message = "年龄最大值不允许超过60岁")
    @Min(value = 18,message = "年龄最小值不允许低于18岁")
    private Integer age;//员工年龄

    //实体类中的引用类型通过标注@Valid注解，设定开启当前引用类型字段中的属性参与校验
    @Valid
    private Address address;

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

    @Override
    public String toString() {
        return "Employee{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", address=" + address +
                '}';
    }
}
```

##### Address.java

```java
package com.itheima.domain;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;
//嵌套校验的实体中，对每个属性正常添加校验规则即可
public class Address {

    @NotBlank(message = "请输入省份名称")
    private String provinceName;//省份名称

    @NotBlank(message = "请输入城市名称")
    private String cityName;//城市名称

    @NotBlank(message = "请输入详细地址")
    private String detail;//详细住址

    @NotBlank(message = "请输入邮政编码")
    @Size(max = 6,min = 6,message = "邮政编码由6位组成")
    private String zipCode;//邮政编码

    public String getProvinceName() {
        return provinceName;
    }

    public void setProvinceName(String provinceName) {
        this.provinceName = provinceName;
    }

    public String getCityName() {
        return cityName;
    }

    public void setCityName(String cityName) {
        this.cityName = cityName;
    }

    public String getDetail() {
        return detail;
    }

    public void setDetail(String detail) {
        this.detail = detail;
    }

    public String getZipCode() {
        return zipCode;
    }

    public void setZipCode(String zipCode) {
        this.zipCode = zipCode;
    }

    @Override
    public String toString() {
        return "Address{" +
                "provinceName='" + provinceName + '\'' +
                ", cityName='" + cityName + '\'' +
                ", detail='" + detail + '\'' +
                ", zipCode='" + zipCode + '\'' +
                '}';
    }
}
```

##### pom.xml

- 导入 Jackson jar 包

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
      <artifactId>springmvc_validator</artifactId>
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

        <!--导入校验的jsr303规范-->
        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>2.0.1.Final</version>
        </dependency>
        <!--导入校验框架实现技术-->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
<!--            <version>5.1.0.Final</version>-->
            <!--6.1 版本异常: java.lang.NoClassDefFoundError: javax/el/ELManager-->
            <version>6.1.0.Final</version>
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



### 2.  SSM三大框架整合

- 只记录核心配置代码


#### 2.1 XML完成整合

- 只提供代码预览，并非整个项目所有代码，如需整套代码，可以从课件压缩包中找到

##### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:applicationContext.xml</param-value>
  </context-param>

  <!--启动服务器时，通过监听器加载spring运行环境-->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

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

    <context:component-scan base-package="com.itheima.controller"/>

</beans>
```

##### applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
    <!--主程序对应的配置文件-->
    <!--主程序对应的配置文件-->
    <!--主程序对应的配置文件-->
    <!--主程序对应的配置文件-->
    <!--主程序对应的配置文件-->
    <!--主程序对应的配置文件-->

    <!--开启bean注解扫描-->
    <context:component-scan base-package="com.itheima">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!--开启注解式事务-->
    <tx:annotation-driven transaction-manager="txManager"/>

    <!--加载properties文件-->
    <context:property-placeholder location="classpath*:jdbc.properties"/>

    <!--数据源-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!--整合mybatis到spring中-->
    <bean class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="typeAliasesPackage" value="com.itheima.domain"/>
        <!--分页插件-->
        <property name="plugins">
            <array>
                <bean class="com.github.pagehelper.PageInterceptor">
                    <property name="properties">
                        <props>
                            <prop key="helperDialect">mysql</prop>
                            <prop key="reasonable">true</prop>
                        </props>
                    </property>
                </bean>
            </array>
        </property>
    </bean>

    <!--映射扫描-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.itheima.dao"/>
    </bean>

    <!--事务管理器-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

</beans>
```

##### ProjectExceptionAdivce.java

```java
package com.itheima.controller.interceptor;

import com.itheima.controller.results.Result;
import com.itheima.system.exception.BusinessException;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

@Component
@ControllerAdvice
public class ProjectExceptionAdivce {

    @ExceptionHandler(BusinessException.class)
    @ResponseBody
    //对出现异常的情况进行拦截，并将其处理成统一的页面数据结果格式
    public Result doBusinessException(BusinessException e){
        return new Result(e.getCode(),e.getMessage());
    }

}
```

##### UserController.java

```java
package com.itheima.controller;

import com.github.pagehelper.PageInfo;
import com.itheima.controller.results.Code;
import com.itheima.controller.results.Result;
import com.itheima.domain.User;
import com.itheima.service.UserService;
import com.itheima.system.exception.BusinessException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping
    public Result save(User user){
        boolean flag = userService.save(user);
        return new Result(flag ? Code.SAVE_OK:Code.SAVE_ERROR);
    }

    @PutMapping
    public Result update(User user){
        boolean flag = userService.update(user);
        return new Result(flag ? Code.UPDATE_OK:Code.UPDATE_ERROR);
    }

    @DeleteMapping("/{uuid}")
    public Result delete(@PathVariable Integer uuid){
        boolean flag = userService.delete(uuid);
        return new Result(flag ? Code.DELETE_OK:Code.DELETE_ERROR);
    }

    @GetMapping("/{uuid}")
    public Result get(@PathVariable Integer uuid){
        User user = userService.get(uuid);
        //模拟出现异常，使用条件控制，便于测试结果
        if (uuid == 10 ) throw new BusinessException("查询出错啦，请重试！",Code.GET_ERROR);
        return new Result(null != user ?Code.GET_OK: Code.GET_ERROR,user);
    }

    @GetMapping("/{page}/{size}")
    public Result getAll(@PathVariable Integer page, @PathVariable Integer size){
        PageInfo<User> all = userService.getAll(page, size);
        return new Result(null != all ?Code.GET_OK: Code.GET_ERROR,all);
    }

    @PostMapping("/login")
    public Result login(String userName,String password){
        User user = userService.login(userName,password);
        return new Result(null != user ?Code.GET_OK: Code.GET_ERROR,user);
    }

}
```



##### UserService.java

```java
package com.itheima.service;

import com.github.pagehelper.PageInfo;
import com.itheima.domain.User;
import org.springframework.transaction.annotation.Transactional;

@Transactional(readOnly = true)
public interface UserService {
    /**
     * 添加用户
     * @param user
     * @return
     */
    @Transactional(readOnly = false)
    public boolean save(User user);

    /**
     * 修改用户
     * @param user
     * @return
     */
    @Transactional(readOnly = false)
    public boolean update(User user);

    /**
     * 删除用户
     * @param uuid
     * @return
     */
    @Transactional(readOnly = false)
    public boolean delete(Integer uuid);

    /**
     * 查询单个用户信息
     * @param uuid
     * @return
     */
    public User get(Integer uuid);

    /**
     * 查询全部用户信息
     * @return
     */
    public PageInfo<User> getAll(int page, int size);

    /**
     * 根据用户名密码进行登录
     * @param userName
     * @param password
     * @return
     */
    public User login(String userName,String password);
}
```

##### UserServiceImpl.java

```java
package com.itheima.service.impl;

import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import com.itheima.dao.UserDao;
import com.itheima.domain.User;
import com.itheima.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    @Override
    public boolean save(User user) {
        return userDao.save(user);
    }

    @Override
    public boolean update(User user) {
        return userDao.update(user);
    }

    @Override
    public boolean delete(Integer uuid) {
        return userDao.delete(uuid);
    }

    @Override
    public User get(Integer uuid) {
        return userDao.get(uuid);
    }

    @Override
    public PageInfo<User> getAll(int page,int size) {
        PageHelper.startPage(page,size);
        List<User> all = userDao.getAll();
        return new PageInfo<User>(all);
    }

    @Override
    public User login(String userName, String password) {
        return userDao.getByUserNameAndPassword(userName,password);
    }
}
```



#### 2.2 注解完成整合

- 只提供代码预览，并非整个项目所有代码，如需整套代码，可以从课件压缩包中找到

##### ServletContainersInitConfig.java

- 做了web.xml 做的事情

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
        ctx.register(SpringMvcConfig.class);
        return ctx;
    }

    //注解配置映射地址方式，服务于SpringMVC的核心控制器DispatcherServlet
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    @Override
    //基本等同于<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    protected WebApplicationContext createRootApplicationContext() {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringConfig.class);
        return ctx;
    }

    //乱码处理作为过滤器，在servlet容器启动时进行配置，相关内容参看Servlet零配置相关课程
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        //触发父类的onStartup
        super.onStartup(servletContext);
        //1.创建字符集过滤器对象
        CharacterEncodingFilter cef = new CharacterEncodingFilter();
        //2.设置使用的字符集
        cef.setEncoding("UTF-8");
        //3.添加到容器（它不是ioc容器，而是ServletContainer）
        FilterRegistration.Dynamic registration = servletContext.addFilter("characterEncodingFilter", cef);
        //4.添加映射
        registration.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST, DispatcherType.FORWARD, DispatcherType.INCLUDE), false, "/*");
    }
}
```

##### SpringConfig.java

- 替代 spring-mvc.xml

```java
package com.itheima.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.*;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.stereotype.Controller;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;

@Configuration
//等同于<context:component-scan base-package="com.itheima">
@ComponentScan(value = "com.itheima",excludeFilters =
    //等同于<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    @ComponentScan.Filter(type= FilterType.ANNOTATION,classes = {Controller.class}))
//等同于<context:property-placeholder location="classpath*:jdbc.properties"/>
@PropertySource("classpath:jdbc.properties")
//开启注解事物支持, 等同于<tx:annotation-driven />，bean的名称默认取transactionManager
@EnableTransactionManagement
@Import({MyBatisConfig.class,JdbcConfig.class})
public class SpringConfig {
    //等同于<bean id="txManager"/>
    @Bean("transactionManager")
    //等同于<bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    public DataSourceTransactionManager getTxManager(@Autowired DataSource dataSource){
        DataSourceTransactionManager tm = new DataSourceTransactionManager();
        //等同于<property name="dataSource" ref="dataSource"/>
        tm.setDataSource(dataSource);
        return tm;
    }
}
```

##### JdbcConfig.java

- 注册并配置数据库连接池对象

```java
package com.itheima.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;

import javax.sql.DataSource;

public class JdbcConfig {
    //使用注入的形式，读取properties文件中的属性值，等同于<property name="*******" value="${jdbc.driver}"/>
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;

    //定义dataSource的bean，等同于<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    @Bean("dataSource")
    public DataSource getDataSource(){
        //创建对象
        DruidDataSource ds = new DruidDataSource();
        //手工调用set方法，等同于set属性注入<property name="driverClassName" value="******"/>
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        return ds;
    }
}
```

##### MyBatisConfig.java

- 注册并配置mybatis 和 spring 整合代码
- 有使用PageHelper插件

```java
package com.itheima.config;

import com.github.pagehelper.PageInterceptor;
import org.apache.ibatis.plugin.Interceptor;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.mapper.MapperScannerConfigurer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;

import javax.sql.DataSource;
import java.util.Properties;

public class MyBatisConfig {

    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;


    //定义MyBatis的核心连接工厂bean，等同于<bean class="org.mybatis.spring.SqlSessionFactoryBean">
    @Bean
    //参数使用自动装配的形式加载dataSource，为set注入提供数据，dataSource来源于JdbcConfig中的配置
    public SqlSessionFactoryBean getSqlSessionFactoryBean(@Autowired DataSource dataSource,@Autowired Interceptor interceptor){
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        //等同于<property name="typeAliasesPackage" value="com.itheima.domain"/>
        ssfb.setTypeAliasesPackage("com.itheima.domain");
        //等同于<property name="dataSource" ref="dataSource"/>
        ssfb.setDataSource(dataSource);
//        //等同于<bean class="com.github.pagehelper.PageInterceptor">
//        Interceptor interceptor = new PageInterceptor();
//        Properties properties = new Properties();
//        properties.setProperty("helperDialect","mysql");
//        properties.setProperty("reasonable","true");
//        //等同于<property name="properties">
//        interceptor.setProperties(properties);
        ssfb.setPlugins(interceptor);
        return ssfb;
    }

    //定义MyBatis的映射扫描，等同于<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    @Bean
    public MapperScannerConfigurer getMapperScannerConfigurer(){
        MapperScannerConfigurer msc = new MapperScannerConfigurer();
        //等同于<property name="basePackage" value="com.itheima.dao"/>
        msc.setBasePackage("com.itheima.dao");
        return msc;
    }

    @Bean("pageInterceptor")
    public Interceptor getPageInterceptor(){
        Interceptor interceptor = new PageInterceptor();
        Properties properties = new Properties();
        properties.setProperty("helperDialect","mysql");
        properties.setProperty("reasonable","true");
        //等同于<property name="properties">
        interceptor.setProperties(properties);
        return interceptor;
    }

}
```

##### SpringMvcConfig.java

- 配置springmvc 开启@Controller包扫描
- 开启注解支持@EnableWebMvc，开启后可实现请求参数自动装载JavaBean、 校验、返回JSON@ResponseBody、拦截器等等功能

```java
package com.itheima.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
//等同于<context:component-scan base-package="com.itheima.controller"/>
@ComponentScan("com.itheima.controller")
//等同于<mvc:annotation-driven/>，还不完全相同
@EnableWebMvc
public class SpringMvcConfig {
}
```





##### ProjectExceptionAdivce.java

```java
package com.itheima.controller.interceptor;

import com.itheima.controller.results.Result;
import com.itheima.system.exception.BusinessException;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

@Component
@ControllerAdvice
public class ProjectExceptionAdivce {

    @ExceptionHandler(BusinessException.class)
    @ResponseBody
    //对出现异常的情况进行拦截，并将其处理成统一的页面数据结果格式
    public Result doBusinessException(BusinessException e){
        return new Result(e.getCode(),e.getMessage());
    }

}
```

##### UserController.java

```java
package com.itheima.controller;

import com.github.pagehelper.PageInfo;
import com.itheima.controller.results.Code;
import com.itheima.controller.results.Result;
import com.itheima.domain.User;
import com.itheima.service.UserService;
import com.itheima.system.exception.BusinessException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping
    public Result save(User user){
        boolean flag = userService.save(user);
        return new Result(flag ? Code.SAVE_OK:Code.SAVE_ERROR);
    }

    @PutMapping
    public Result update(User user){
        boolean flag = userService.update(user);
        return new Result(flag ? Code.UPDATE_OK:Code.UPDATE_ERROR);
    }

    @DeleteMapping("/{uuid}")
    public Result delete(@PathVariable Integer uuid){
        boolean flag = userService.delete(uuid);
        return new Result(flag ? Code.DELETE_OK:Code.DELETE_ERROR);
    }

    @GetMapping("/{uuid}")
    public Result get(@PathVariable Integer uuid){
        User user = userService.get(uuid);
        //模拟出现异常，使用条件控制，便于测试结果
        if (uuid == 10 ) throw new BusinessException("查询出错啦，请重试！",Code.GET_ERROR);
        return new Result(null != user ?Code.GET_OK: Code.GET_ERROR,user);
    }

    @GetMapping("/{page}/{size}")
    public Result getAll(@PathVariable Integer page, @PathVariable Integer size){
        PageInfo<User> all = userService.getAll(page, size);
        return new Result(null != all ?Code.GET_OK: Code.GET_ERROR,all);
    }

    @PostMapping("/login")
    public Result login(String userName,String password){
        User user = userService.login(userName,password);
        return new Result(null != user ?Code.GET_OK: Code.GET_ERROR,user);
    }

}
```



##### UserService.java

```java
package com.itheima.service;

import com.github.pagehelper.PageInfo;
import com.itheima.domain.User;
import org.springframework.transaction.annotation.Transactional;

@Transactional(readOnly = true)
public interface UserService {
    /**
     * 添加用户
     * @param user
     * @return
     */
    @Transactional(readOnly = false)
    public boolean save(User user);

    /**
     * 修改用户
     * @param user
     * @return
     */
    @Transactional(readOnly = false)
    public boolean update(User user);

    /**
     * 删除用户
     * @param uuid
     * @return
     */
    @Transactional(readOnly = false)
    public boolean delete(Integer uuid);

    /**
     * 查询单个用户信息
     * @param uuid
     * @return
     */
    public User get(Integer uuid);

    /**
     * 查询全部用户信息
     * @return
     */
    public PageInfo<User> getAll(int page, int size);

    /**
     * 根据用户名密码进行登录
     * @param userName
     * @param password
     * @return
     */
    public User login(String userName,String password);
}
```

##### UserServiceImpl.java

```java
package com.itheima.service.impl;

import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import com.itheima.dao.UserDao;
import com.itheima.domain.User;
import com.itheima.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    @Override
    public boolean save(User user) {
        return userDao.save(user);
    }

    @Override
    public boolean update(User user) {
        return userDao.update(user);
    }

    @Override
    public boolean delete(Integer uuid) {
        return userDao.delete(uuid);
    }

    @Override
    public User get(Integer uuid) {
        return userDao.get(uuid);
    }

    @Override
    public PageInfo<User> getAll(int page,int size) {
        PageHelper.startPage(page,size);
        List<User> all = userDao.getAll();
        return new PageInfo<User>(all);
    }

    @Override
    public User login(String userName, String password) {
        return userDao.getByUserNameAndPassword(userName,password);
    }
}
```






