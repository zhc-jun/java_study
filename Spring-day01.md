# **Spring 第一天   ----框架阶段**

昨日回顾：打开昨天md 文档复习

### **今日学习目标**：

​    1）了解spring 框架的概念和体系结构组成 ---------------------------->**先认识这个框架是个什么东西？**

​    2）掌握两个核心功能：控制反转（IOC）和  依赖注入 （DI）---------> **它的作用是啥？**

​    3）掌握框架 的相关配置  -----------------------------------------------> **如何去使用这个 spring 框架 ？(要知道配置)**

​    4）掌握spring 框架与myabtis 框架的整合------------------------------>**结合其它框架去是使用spring 综合演练一下** 

### 1. Spring简介 

1.1 【spring概述】问题

1.问题 ：什么是框架？好处是什么？

2.问题：什么是spring ？

3.问题：spring的优势在哪？



1.2【spring 概述】答案

1.什么是框架？好处是什么？

​          经过验证的，具有一定功能的，半成品软件；

​          提高开发效率，增强可重用性，提供编写规范，节约维护成本，解耦底层实现原理

2.什么是spring ？

​          Spring是分层的JavaSE/EE应用full-stack轻量级开源框架   

![image-20210415204425631](C:\Users\jiangyudeng\AppData\Roaming\Typora\typora-user-images\image-20210415204425631.png)

​        重点了解spring 的体系结构  -----对spring 结构体系有个总体认知

![image-20201206100608041](C:\Users\jiangyudeng\AppData\Roaming\Typora\typora-user-images\image-20201206100608041.png)



​         3.spring的优势在哪？

​               ◆简化开发

​               ◆方便集成各种优秀框架 

​               ◆方便程序的测试

​              ◆AOP编程的支持

​             ◆声明式事务的支持

​             ◆降低JavaEEAPI的使用难度

1.3 总结

​        spring 框架是一个大的概念范围 （这里我们学习的主要是spring 家族中的spring framework 技术）

​        Spring是分层的JavaSE/EE应用full-stack轻量级开源框架,可以简化开发,可以集成其它相关技术-----集成 ；整合

### 2. IoC简介

2.1 【spring IoC 概述】问题

1. 什么是高内聚低耦合？

​            2.spring 的演化过程是怎样的？

​            3.什么是IOC？

 1.2【spring IoC 概述】答案

​           1.什么是高内聚低耦合？

​           ◆就是同一个模块内的各个元素之间要高度紧密(举列：一个程序有50个函数，这个程序执行得非常好)，但是各个模块之间的相互依存度却不要那么紧密（举列：然而一旦你修改其中一个函数，其他49个函数都需要做修改）

​         2.spring 的演化过程是怎样的？

![image-20210419205700454](C:\Users\jiangyudeng\AppData\Roaming\Typora\typora-user-images\image-20210419205700454.png)

​      3.什么是IOC？

IoC（Inversion Of Control）控制反转，Spring反向控制应用程序所需要使用的外部资源

传统模式：需要我们主动去new 出一个对象--->主动权在类手中

ioc模式：主动变被动，不需要我们主动去new 一个类对象，交给spring 的 ioc 容器去管理--->主动权在spring 手中

<font color=red>注意：spring创建好了对象之后，都会放到bean容器（ioc 容器）中，这里同学们不需要较真这个容器到底是什么东西，简单理解为：“spring维护了一个map集合，存放着各种我们让它实例化的对象”。</font>

与之对应的DI-->两者必然同时存在

DI(依赖注入): 由IoC容器动态的将某个依赖关系(即bean 对象)注入到组件或者业务逻辑代码中,过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现

**IoC和DI**由什么**关系**呢？其实它们是同一个概念的不同角度描述

2.1  总结

正因为spring搞出bean容器，围绕容器实现依赖注入和控制反转，才使得我们代码耦合度降低，bean容器底层基于工厂模式衍化而来（了解即可）。

**注意**：从功能上出发，依赖注入和控制反转本身就存在部分雷同。



### 3. 入门案例

IoC入门案例制作步骤：

1.导入spring坐标（5.1.9.release）

2.编写业务层与表现层（模拟）接口与实现类

3.建立spring配置文件

4.配置所需资源（Service）为spring控制的资源

5.表现层（App）通过spring获取资源（Service实例）

#### pom.xml 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>Spring_day01_01_ioc入门程序</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
    </dependencies>

</project>
```

#### applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 1.创建spring控制的资源-->
    <bean id="userService" class="com.itheima.service.impl.UserServiceImpl"/>

</beans>
```

#### UserServiceImpl.java

```java
package com.itheima.service.impl;

import com.itheima.service.UserService;

public class UserServiceImpl implements UserService {
   
    public void save() {
        System.out.println("user service running...");
    }
}

```

#### UserApp.java

```java
import com.itheima.service.UserService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class UserApp {
    public static void main(String[] args) {
        //2.加载配置文件
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        //3.获取资源
        UserService userService = (UserService) ctx.getBean("userService");
        userService.save();
    }
}

```



### 4. IoC配置（XML格式）

#### 4.1 bean标签 id, class, name 属性

**id和name**本质上其实是相同的，都可以唯一地标识一个bean。区别是id只能定义一个值，name可以定义多个值。

2. 如果设置id属性，那么id就是bean的唯一标识符，在spring容器中必需唯一。
3. 如果仅设置name属性，那么name就是bean的唯一标识符，必需在容器中唯一。
4. 如果同时设置id和name，那么id是唯一标识符，name是别名。如果id和name的值相同，那么spring容器会自动检测并消除冲突：让这个bean只有标识符，而没有别名。
5. name属性设置多个值且不设置id属性，那么第一个被用作标识符，其他的被视为别名。如果设置了id，那么name的所有值都是别名。
6. **不管是标识符，还是别名，在容器中必需唯一。因为标识符和别名，都是可以用来获取bean的，如果不唯一，显然不知道获取的到底是哪儿一个bean**。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- bean基本属性id,name,class实验-->
    <!-- bean基本属性id,name,class实验 -->
    <!-- bean基本属性id,name,class实验 -->
    <!-- bean基本属性id,name,class实验 -->
    <!-- bean可以定义多个名称，使用name属性完成，中间使用,分割-->
    <bean id="userService" name="userService1,userService2" class="com.itheima.service.impl.UserServiceImpl"/>


</beans>
```

```java
import com.itheima.service.UserService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class UserApp {
    public static void main(String[] args) {

        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService1 = (UserService) ctx.getBean("userService");
        UserService userService2 = (UserService) ctx.getBean("userService1");
        UserService userService3 = (UserService) ctx.getBean("userService2");
        System.out.println(userService1);
        System.out.println(userService2);
        System.out.println(userService3);
        
    }
}

```

#### 4.2 bean标签scope 属性

- scope="singleton"默认值， Singleton是单例类型，就是在创建起容器时就同时自动创建了一个bean的对象，不管你是否使用，他都存在了，每次获取到的对象都是同一个对象 
- scope="prototype"原型， Prototype是原型类型，它在我们创建容器的时候并没有实例化，而是当我们获取bean的时候才会去创建一个对象，而且我们每次获取到的对象都不是同一个对象 。
-  scope=request、session、application、websocket ：设定创建出的对象放置在web容器对应的位置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- bean创建后的对象是多实例的 -->
    <!--<bean id="userService3" scope="prototype" class="com.itheima.service.impl.UserServiceImpl"/>-->
    
    <!--默认值 单利-->
    <bean id="userService3" scope="singleton" class="com.itheima.service.impl.UserServiceImpl"/>

</beans>
```

#### 4.3 bean标签声明周期 （了解）

- bean标签    init-method="init方法名"             destroy-method="destroy方法名" 

- 不能带有形参

- 取值：bean对应的类中对应的具体方法名 

- 注意事项： 

  ◆ 当scope=“singleton”时，spring容器中有且仅有一个对象，init方法在创建容器时仅执行一次 

  ◆ 当scope=“prototype”时，spring容器要创建同一类型的多个对象，init方法在每个对象创建 时均执行一次

   ◆ 当scope=“singleton”时，关闭容器会导致bean实例的销毁，调用destroy方法一次 

  ◆ 当scope=“prototype”时，对象的销毁由垃圾回收机制gc()控制，destroy方法将不会被执行

```xml
<bean init-method="init" destroy-method="destroy></bean>
```

```java
package com.itheima.service.impl;

import com.itheima.service.UserService;

public class UserServiceImpl implements UserService {

    public UserServiceImpl(){
        System.out.println(" constructor is running...");
    }

    public void init(){
        System.out.println("init....");
    }

    public void destroy(){
        System.out.println("destroy....");
    }

    public void save() {
        System.out.println("user service running...");
    }
}

```

#### 4.4 bean对象创建方式（了解）

##### 4.4.1 静态工厂创建bean对象

-  class属性必须配置成静态工厂的**包名.类名**
- factory-method属性必须配置为静态工厂类创建对象的 **静态方法**

```xml
<!--静态工厂创建bean-->
<bean id="userService4" class="com.service.UserServiceFactory" factory-method="getService"/>
```

```java
package com.service;

import com.service.impl.UserServiceImpl;

public class UserServiceFactory {
    public static UserService getService(){
        System.out.println("factory create object...");
        return new UserServiceImpl();
    }
}
```

##### 4.4.2 实例工厂创建bean对象

-  使用实例工厂创建bean首先需要将实例工厂配置bean，交由spring进行管理 
- factory-bean属性是实例工厂的**beanId**
- factory-method属性必须配置为静态工厂类创建对象的 **普通方法**

```xml
<!--实例工厂对应的bean-->
<bean id="factoryBean" class="com.service.UserServiceFactory2"/>

<!--实例工厂创建bean，依赖工厂对象对应的bean-->
<bean id="userService5" factory-bean="factoryBean" factory-method="getService" />
```

```java
package com.service;

import com.service.impl.UserServiceImpl;

public class UserServiceFactory2 {

    public UserService getService(){
        System.out.println(" instance factory create object...");
        return new UserServiceImpl();
    }
}
```

#### 4.5 依赖注入（DI）

##### 4.5.1 setter注入（推荐）

- property 标签

- 属性：

  ◆ name：对应bean中的属性名，要求该属性必须提供可访问的set方法（严格规范为此名称是set方法对应名称） 

  ◆ value：设定非引用类型属性对应的值，**不能与ref同时使用** 

  ◆ ref：设定引用类型属性对应bean的id ，**不能与value同时使用** 

- 注意：

  1）一个bean可以有多个property标签

  2）value 或 ref 设置为 类的属性名时，必须使用idea为属性提供setter 方法，因为spring底层基于setter方法完成的属性赋值；

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- set注入实验- -->
    <bean id="userService" class="com.impl.UserServiceImpl">
        <!--3.将要注入的引用类型的变量通过property属性进行注入，对应的name是要注入的变量名，使用ref属性声明要注入的bean的id-->
        <property name="userDao" ref="userDao"/>
        <property name="num" value="666"/>
        <property name="version" value="itheima"/>
    </bean>

</beans>
```

```java
package com.itheima.service.impl;

import com.itheima.dao.BookDao;
import com.itheima.dao.UserDao;
import com.itheima.service.UserService;

public class UserServiceImpl implements UserService {

    private UserDao userDao;
    private BookDao bookDao;
    private int num;
    private String version;

    public UserServiceImpl() {
    }

    public UserServiceImpl(UserDao userDao, int num, String version){
        this.userDao = userDao;
        this.num = num;
        this.version = version;
    }

    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }
    public void setVersion(String version) {
        this.version = version;
    }
    public void setNum(int num) {
        this.num = num;
    }
    //1.对需要进行诸如的变量添加set方法
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void save() {
        System.out.println("user service running..."+num+" "+version);
        userDao.save();
        bookDao.save();
    }
}

```

##### 4.5.2 构造器注入（了解）

- constructor-arg 标签

- 属性：

  ◆ name：对应bean中的构造方法所携带的参数名

  ◆ value：设定非引用类型属性对应的值，**不能与ref同时使用** 

  ◆ ref：设定引用类型属性对应bean的id ，**不能与value同时使用** 

  ◆ index（了解）：指定参数索引下标进行指定赋值, 默认从 0 开始。

  ◆ type （了解）：设定构造方法参数的类型，用于按类型匹配参数或进行类型校验 

- 注意：

  1）一个bean可以有多个constructor-arg标签

  2）name 与 index 不推荐结合使用

  3）构造输入时，index 或 name 任选其一进行使用，不推荐采用默认的一致顺序或类型来赋值；

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService" class="com.impl.UserServiceImpl">
        <!--使用构造方法进行set注入，需要保障注入的属性与bean中定义的属性一致-->
        <!--一致指顺序一致或类型一致或使用index解决该问题-->
        <constructor-arg name="userDao" ref="userDao"/>
        <constructor-arg name="num" value="666666"/>
        <constructor-arg name="version" value="itcast"/>
    </bean>

    <!-- 了解即可 -->
    <bean id="userDao" class="com.impl.UserDaoImpl">
        <constructor-arg index="2" value="123"/>
        <constructor-arg index="1" value="root"/>
        <constructor-arg index="0" value="com.mysql.jdbc.Driver"/>
    </bean>

</beans>
```

```java
package com.itheima.service.impl;

import com.itheima.dao.BookDao;
import com.itheima.dao.UserDao;
import com.itheima.service.UserService;

public class UserServiceImpl implements UserService {

    private UserDao userDao;
    private BookDao bookDao;
    private int num;
    private String version;

    public UserServiceImpl() {
    }

    public UserServiceImpl(UserDao userDao, int num, String version){
        this.userDao = userDao;
        this.num = num;
        this.version = version;
    }

    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }
    public void setVersion(String version) {
        this.version = version;
    }
    public void setNum(int num) {
        this.num = num;
    }
    //1.对需要进行诸如的变量添加set方法
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void save() {
        System.out.println("user service running..."+num+" "+version);
        userDao.save();
        bookDao.save();
    }
}
```

##### 4.5.3 集合类型注入（了解）

- array，list，set，map，props  标签

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 集合注入顺序实验- -->
    <bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl">
        <!-- list注入 -->
        <property name="al">
            <list>
                <value>itheima</value>
                <value>66666</value>
            </list>
        </property>
        
        <!-- properties注入 -->
        <property name="properties">
            <props>
                <prop key="name">itheima666</prop>
                <prop key="value">666666</prop>
            </props>
        </property>
        
        <!-- array注入 -->
        <property name="arr">
            <array>
                <value>123456</value>
                <value>66666</value>
            </array>
        </property>
        
        <!-- set注入 -->
        <property name="hs">
            <set>
                <value>itheima</value>
                <value>66666</value>
            </set>
        </property>
        
        <!-- map注入 -->
        <property name="hm">
            <map>
                <entry key="name" value="itheima66666"/>
                <entry key="value" value="6666666666"/>
            </map>
        </property>
    </bean>


</beans>
```

```java
package com.itheima.dao.impl;

import com.itheima.dao.BookDao;

import java.util.*;

public class BookDaoImpl implements BookDao {

    private List al;
    private Properties properties;
    private int[] arr;
    private Set hs;
    private Map hm ;

    public void setAl(List al) {
        this.al = al;
    }

    public void setProperties(Properties properties) {
        this.properties = properties;
    }

    public void setArr(int[] arr) {
        this.arr = arr;
    }

    public void setHs(Set hs) {
        this.hs = hs;
    }

    public void setHm(Map hm) {
        this.hm = hm;
    }
}
```

##### 4.5.4 p命名空间 （了解）

- beans 标签引入：xmlns:p="http://www.springframework.org/schema/p"
- 语法：**p:属性名-ref="属性值beanid"**, 例如：p:userDao-ref="userDao"   userDao是真实存在bean标签id 属性值。
- 好处：简化我们bean标签的抒写，内容过多也不利于阅读
- 如果是给该类String int 等类型赋值格式如下：**p:属性名-ref="属性值"**, 例如：p:userName="张三"  p:passWord="123456", 类中可以有String userName; int passWord; 两个属性；

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 构造器注入实验- -->
    <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl">
        <constructor-arg index="2" value="123"/>
        <constructor-arg index="1" value="root"/>
        <constructor-arg index="0" value="com.mysql.jdbc.Driver"/>
    </bean>
   
    <!-- 集合注入顺序实验- -->
    <bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl"> 
    </bean>


    <bean id="userService" class="com.itheima.service.impl.UserServiceImpl">
        <property name="userDao" ref="userDao"/>
        <property name="bookDao" ref="bookDao"/>
    </bean>
    <!-- p命名空间简化注入书写实验- -->
    <bean
        id="userService"
        class="com.itheima.service.impl.UserServiceImpl"
        p:userDao-ref="userDao"
        p:bookDao-ref="bookDao"
        />

</beans>
```

##### 4.5.5 SpEl表达式注入 （了解）

- 所有格式统一使用 value=“********” 

  ◆ 常量 #{10}  #{3.14}  #{2e5}  #{‘itcast’} 

  ◆ 引用bean #{beanId}  

  ◆ 引用bean属性 #{beanId.propertyName} 

  ◆ 引用bean方法 #{beanId.methodName()}

  ..........

- 用处：所有类型的注入都是value属性来完成，替代掉ref 属性。

- 最后：spel 用的不多，了解即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 构造器注入实验- -->
    <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl">
     </bean>
    <!-- 构造器注入实验- -->
    <bean id="bookDao" class="com.itheima.dao.impl.UserDaoImpl">   
    </bean>
    
   <!-- Spring EL表达式实验- -->
    <bean id="userService" class="com.itheima.service.impl.UserServiceImpl">
        <property name="userDao" value="#{userDao}"/>
        <property name="bookDao" value="#{bookDao}"/>
        <property name="num" value="#{666666666}"/>
        <property name="version" value="#{'itcast'}"/>
    </bean>
</beans>
```



##### 4.5.6 读取外部properties文件信息

- 引入命名空间

  ```xml
  xmlns:context="http://www.springframework.org/schema/context"
          http://www.springframework.org/schema/context
          https://www.springframework.org/schema/context/spring-context.xsd
  ```

- 配置properties文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:p="http://www.springframework.org/schema/p"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          https://www.springframework.org/schema/context/spring-context.xsd">
  
      <!-- 加载properties文件属性实验- -->
      <!--加载配置文件: 模糊匹配-->
      <context:property-placeholder location="classpath:*.properties"/>
      
      <!--加载配置文件：带文件夹层级模糊匹配 -->
      <context:property-placeholder location="classpath:*/*.properties"/>
      
      <!--加载配置文件：指定文件夹 -->
      <context:property-placeholder location="classpath:config/jdbc.properties"/>
      
      <!-- 加载配置文件：指定文件 -->
      <context:property-placeholder location="classpath:jdbc.properties"/>
  
      <bean id="userDao" class="com.dao.impl.UserDaoImpl">
          <property name="userName" value="${username}"/>
          <property name="password" value="${pwd}"/>
      </bean>
  
  </beans>
  ```

- 注意事项：${} 要被 “” 号包裹起来, {} 中的文本应该为properties文件中的key.



##### 4.5.7 import导入配置文件

- import 标签

- 属性:

   1）resource：加载的配置路径，如果有文件夹名，则：文件夹名称/文件名.xml

- ```xml
  <!-- applicationContext.xml 文件内容如下 -->
  <beans .....此处省略>
      
      <!-- 团队合作import实验- -->
      <import resource="applicationContext-user.xml"/>
      <import resource="applicationContext-book.xml"/>
  
  </beans>
  ```

  ```java
  new ClassPathXmlApplicationContext("applicationContext.xml");
  ```

- Spring容器也可以加载多个配置文件（了解）

  ```java
  new ClassPathXmlApplicationContext("applicationContext-user.xml","applicationContext-book.xml");       
  ```

-  Spring容器中的bean定义冲突问题 

  ◆ 同id的bean，后定义的覆盖先定义的 

  ◆ 导入配置文件可以理解为将导入的配置文件复制粘贴到对应位置 

  ◆ 导入配置文件的顺序与位置不同可能会导致最终程序运行结果不同

<font color=red>注意：相同名称bean id 不允许在同一个xml配置文件中配置两次，import 引入的外部bean id 可以；</font>

##### 4.5.8 ApplicationContext 层次结构 （了解）

**ApplicationContext**

- ApplicationContext是一个接口，提供了访问spring容器的API 

- ClassPathXmlApplicationContext是一个类，实现了上述功能 

- ApplicationContext的顶层接口是BeanFactory 

- BeanFactory定义了bean相关的最基本操作 

- ApplicationContext在BeanFactory基础上追加了若干新功能

**对比BeanFactory**

- BeanFactory创建的bean采用延迟加载形式，使用才创建
- ApplicationContext创建的bean默认采用立即加载形式

**FileSystemXmlApplicationContext**

- 可以加载文件系统中任意位置的配置文件，而ClassPathXmlApplicationContext只能加载类路径下的配置文件



##### 4.5.9 三方jar 类bean配置

- 注意:

  1）三方类中定义的属性名

  2）三方类构造方法是什么情况，有无参数，setter/构造方法赋值

```xml
<!-- applicationContext.xml 文件内容如下 -->
<beans .....此处省略>
    
    <!-- 加载第三方类库资源实验- -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring_db"/>
        <property name="username" value="root"/>
        <property name="password" value="itheima"/>
    </bean>
</beans>
```

```java
import com.alibaba.druid.pool.DruidDataSource;
import com.itheima.service.UserService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class UserApp {

    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");

        DruidDataSource dataSource = (DruidDataSource) ctx.getBean("dataSource");
        System.out.println(dataSource);
    }
}

```



### 5. 综合案例-用spring整合mybatis

#### 整合前基础准备工作 

1. spring配置文件，加上context命名空间，用于加载properties文件 
2. 开启加载properties文件 
3. 配置数据源druid（备用） 
4. 定义service层bean，注入dao层bean 
5. dao的bean无需定义，使用代理自动生成

#### 整合工作 

1. 导入Spring整合MyBatis坐标

2. 将mybatis配置成spring管理的bean（SqlSessionFactoryBean）

    ◆ 将原始配置文件中的所有项，转入到当前配置中 

   ​		◼ 数据源转换 

   ​		◼ 映射转换 

​          3.通过spring加载mybatis的映射配置文件到spring环境中 

​          4.设置类型别名 

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>Spring_day01_05_（spring整合mybatis）</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.3</version>
    </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.16</version>
        </dependency>
        
        <!-- 该jar包为了我们做了很多事情 -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.0</version>
        </dependency>
    </dependencies>

</project>
```

#### applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!--加载perperties配置文件的信息-->
    <context:property-placeholder location="classpath:*.properties"/>

    <!--加载druid资源-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!--配置service作为spring的bean,注入dao-->
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
    </bean>

    <!--spring整合mybatis后控制的创建连接用的对象-->
    <bean class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="typeAliasesPackage" value="com.itheima.domain"/>
    </bean>

    <!--加载mybatis映射配置的扫描，将其作为spring的bean进行管理-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurerss">
        <property name="basePackage" value="com.itheima.dao"/>
    </bean>


</beans>
```



#### 总结

⚫ 需要专用的spring整合mybatis的jar包 

⚫ Mybatis核心配置文件消失 

​		◼ 环境environment转换成数据源对象 

​		◼ 映射Mapper扫描工作交由spring处理 

​		◼ 类型别名交由spring处理 

⚫ 业务发起使用spring上下文对象获取对应的bean