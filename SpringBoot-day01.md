## SpringBoot-day01

### **学习目标**：

​    1）springboot实现入门案列功能

​    2）知道springboot的起步依赖（重点）

​    3）spirngboot的yaml 配置

​    4）springboot整合其他框架



### 1.  SpringBoot 概述。  

SpringBoot提供了一种快速使用Spring的方式，基于**约定优于配置**的思想，可以让开发人员不必在配置与逻 辑业务之间进行思维的切换，全身心的投入到逻辑业务的代码编写中，从而大大提高了开发的效率，一定程度 上缩短了项目周期。2014 年 4 月，Spring Boot 1.0.0 发布。Spring的顶级项目之一(https://spring.io)



#### 1.1 Spring 缺点

- 1） 配置繁琐 虽然Spring的组件代码是轻量级的，但它的配置却是重量级的。一开始，Spring用XML配置，而且是很多 XML配置。Spring 2.5引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式XML配置。 Spring 3.0引入了基于Java的配置，这是一种类型安全的可重构配置方式，可以代替XML。 所有这些配置都代表了开发时的损耗。因为在思考Spring特性配置和解决业务问题之间需要进行思维切换，所 以编写配置挤占了编写应用程序逻辑的时间。和所有框架一样，Spring实用，但它要求的回报也不少。
- 2）依赖繁琐 项目的依赖管理也是一件耗时耗力的事情。在环境搭建时，需要分析要导入哪些库的坐标，而且还需要分析导 入与之有依赖关系的其他库的坐标，一旦选错了依赖的版本，随之而来的不兼容问题就会严重阻碍项目的开发 进度。



#### 1.2 SpringBoot 功能

- 1） **自动配置** Spring Boot的自动配置是一个运行时（更准确地说，是应用程序启动时）的过程，考虑了众多因素，才决定 Spring配置应该用哪个，不该用哪个。该过程是SpringBoot自动完成的。

- 2） **起步依赖** 起步依赖本质上是一个Maven项目对象模型（Project Object Model，POM），定义了对其他库的传递依赖 ，这些东西加在一起即支持某项功能。 简单的说，起步依赖就是将具备某种功能的坐标打包到一起，并提供一些默认的功能。

- 3） **辅助功能** 提供了一些大型项目中常见的非功能性特性，如嵌入式服务器、安全、指标，健康检测、外部配置等。


<font color=red>Spring Boot 并不是对 Spring 功能上的增强，而是提供了一种快速使用Spring 的方式。</font>




### 2. SpringBoot 快速入门

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>springboot-helloworld</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--springboot工程需要继承的父工程-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.8.RELEASE</version>
    </parent>

    <dependencies>
        <!--web开发的起步依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```

#### HelloController.java

```java
package com.itheima.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello(){
        return " hello Spring Boot !";
    }
}
```

#### HelloApplication.java

```java
package com.itheima;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * 引导类。 SpringBoot项目的入口
 */
@SpringBootApplication
public class HelloApplication {

    public static void main(String[] args) {
        SpringApplication.run(HelloApplication.class,args);
    }
}
```



### 3.  SpringBoot 起步依赖原理分析

⚫ 在spring-boot-starter-parent的pom.xml 中定义了各种技术的版本信息，组合了一套最优搭配的技术版本。
⚫ 在各种starter中，定义了完成该功能需要的坐标合集，其中大部分版本信息来自于父工程。
⚫ 我们的工程继承parent，引入starter后，通过依赖传递，就可以简单方便获得需要的jar包，并且不会存在 版本冲突等问题, 这些jar包版本都是springboot官方经过测试验证过的。



### 4. SpringBoot 配置

#### 4.1 配置文件分类

- <font color=red>properties 与 yml 在线转换工具: https://www.toyaml.com/index.html </font>

SpringBoot是**基于约定**的，所以很多配置都有默认值，但如果想使用自己的配置替换默认配置的话，就可以使用:

**application.properties**或者**application.yml**（application.yaml）进行配置.

⚫ properties：
           server.port=8080
⚫ yml:
server:

​		port: 8080



**小结：**

⚫ SpringBoot提供了2种配置文件类型：properteis和yml/yaml
⚫ 默认配置文件名称：application
⚫ 在同一级目录下优先级为：properties > yml > yaml (2.1.8.RELEASE)

⚫ 在同一级目录下优先级为：properties < yml < yaml (2.4.0.RELEASE)



#### 4.2 yaml

YAML全称是 YAML Ain't Markup Language 。YAML是一种直观的能够被电脑识别的的数据数据序列化格式，并且容易被人类阅 读，容易和脚本语言交互的，可以被支持YAML库的不同的编程语言程序导入，比如： C/C++, Ruby, Python, Java, Perl, C#, PHP 等。YML文件是以数据为核心的，比传统的xml方式更加简洁。 YAML文件的扩展名可以使用.yml或者.yaml。



⚫ properties:

```properties
server.port=8080 
server.address=127.0.0.1
```

⚫ xml:

```xml
<server> 
  <port>8080</port> 
  <address>127.0.0.1</address> 
</server>
```

⚫ yml:

```yaml
server: 
    port: 8080 
    address: 127.0.0.1
```



<font color=red>简洁，以数据为核心 ! </font>



##### **YAML：基本语法**

⚫ 大小写敏感

⚫ 数据值前边必须有空格，作为分隔符 

⚫ 使用缩进表示层级关系 

⚫ 缩进时不允许使用Tab键，只允许使用空格（各个系统Tab对应的 空格数目可能不同，导致层次混乱）。 

⚫ 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可 

⚫ # 表示注释，从这个字符一直到行尾，都会被解析器忽略。

```yaml
server: 
    port: 8080 
    address: 127.0.0.1
```

##### YAML：数据格式

⚫ 对象(map)：键值对的集合。

```yaml
person: 
  name: zhangsan 
# 行内写法 
person: {name: zhangsan}
```

⚫ 数组：一组按次序排列的值

```yaml
address: 
  - beijing 
  - shanghai 
# 行内写法 
address: [beijing,shanghai]
```

⚫ 纯量：单个的、不可再分的值

```yaml
msg1: 'hello \n world'  # 单引忽略转义字符 
msg2: "hello \n world"  # 双引识别转义字符
```

##### YAML：参数引用

```yaml
name: lisi
person: 
  name: ${name} # 引用上边定义的 name 值
```



##### YAML：小结

1） 配置文件类型 

​       ⚫ properties：和以前一样 

​       ⚫ yml/yaml：注意空格 

2） yaml：简洁，以数据为核心 

​		⚫ 基本语法 

​                  • 大小写敏感 

​                  • 数据值前边必须有空格，作为分隔符 

​                  • 使用空格缩进表示层级关系，相同缩进表示同一级 

​		⚫ 数据格式

​                   • 对象 
​                   • 数组: 使用 “- ”表示数组每个元素 

​                   • 纯量 

​		⚫ 参数引用 

​                   • ${key} 





#### 4.3 读取配置文件内容

1） @Value
2） Environment
3） @ConfigurationProperties 

**示例代码**

##### application.yml

```yaml
#server:
#  port: 2000

name: zhagnsan

person:
  name: zhangsan
  age: 20
  address:
    - beijing
    - shanghai

# 行内写法
person2: {name: lisi,age: 25}

# 数组
address:
  - beijing
  - shanghai

# 纯量
msg1: 'hello \n world'  # 单引忽略转义字符
msg2: "hello \n world"  # 双引识别转义字符

```

##### HelloController.java

```java
/*
package com.itheima.springbootinit;*/
package com.itheima.springbootinit;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Value("${name}")
    private String name;

    @Value("${person2.name}")
    private String name2;

    @Value("${person2.age}")
    private int age;

    @Value("${address[0]}")
    private String address1;

    @Value("${msg1}")
    private String msg1;

    @Value("${msg2}")
    private String msg2;

    @Autowired
    private Environment env;

    @Autowired
    private Person person;

    @RequestMapping("/hello2")
    public String hello2() {

        System.out.println(name);
        System.out.println(name2);
        System.out.println(age);
        System.out.println(address1);
        System.out.println(msg1);
        System.out.println(msg2);

        System.out.println("----------------------");

        System.out.println(env.getProperty("person.name"));
        System.out.println(env.getProperty("address[0]"));

        System.out.println("-------------------");


        System.out.println(person);
        String[] address = person.getAddress();
        for (String s : address) {
            System.out.println(s);
        }


        return "hello Spring Boot 222!";
    }


    @RequestMapping("/hello")
    public String hello() {
        return "hello Spring Boot 222!";
    }
}

```

##### Person.java

```java
package com.itheima.springbootinit;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Arrays;

@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String name;
    private int age;
    private String[] address;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String[] getAddress() {
        return address;
    }

    public void setAddress(String[] address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", address=" + Arrays.toString(address) +
                '}';
    }
}

```

##### SpringbootInitApplication.java

```java
package com.itheima.springbootinit;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootInitApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootInitApplication.class, args);
    }

}
```



#### 4.4 profile

我们在开发Spring Boot应用时，通常同一套程序会被安装到不同环境，比如：开发、测试、生产等。其中数据库地址、服务 器端口等等配置都不同，如果每次打包时，都要修改配置文件，那么非常麻烦。profile功能就是来进行动态配置切换的。

<font color=red>切记：约定大于配置，文件名称和格式固定</font>

##### profile配置方式 

-  多profile文件方式 
-  yml多文档方式 

**多profile文件方法**

- 推荐, 我们可以在application.properties 中配置共性的配置, 在其他三个环境配置中配置因环境不同导致差异性的配置。

- 项目中准备四个properties文件
  - application.properties
  - application-dev.properties
  - application-test.properties
  - application-pro.properties

###### application.properties

```properties
# 激活 application-dev.properties文件
spring.profiles.active=dev
```



**yml多文档方式**

- 在yml 文件中 --- 三个横杠表示一个文档 

###### application.yml

```yaml
---
server:
  port: 8081

spring:
  profiles: dev
---

server:
  port: 8082

spring:
  profiles: test
---
server:
  port: 8083

spring:
  profiles: pro
---
spring:
  profiles:
    active: dev
```



##### profile激活方式 

​	⚫ 配置文件中：spring.profiles.active=dev 

​	⚫ 虚拟机参数：在VM options 指定：-Dspring.profiles.active=dev 

​	⚫ 命令行参数：java –jar xxx.jar  --spring.profiles.active=dev

<font color=red>优先级: 命令行 > 虚拟机参数 > 配置文件</font>



##### Profile-小结

1） profile是用来完成不同环境下，配置动态切换功能的。 

2） profile配置方式 

​	⚫ 多profile文件方式：提供多个配置文件，每个代表一种环境。 

​		• application-dev.properties/yml 开发环境 

​		• application-test.properties/yml 测试环境 

​		• application-pro.properties/yml 生产环境 

​	⚫ yml多文档方式： • 在yml中使用 --- 分隔不同配置 

3） profile激活方式 

​	⚫ 配置文件中配置：spring.profiles.active=dev 

​	⚫ 虚拟机参数：在VM options 指定：-Dspring.profiles.active=dev 

​	⚫ 命令行参数：java –jar xxx.jar  --spring.profiles.active=dev



#### 4.5 内部配置加载顺序

- Springboot程序启动时，会从以下位置加载配置文件： 
1. (外部) file:./config/：当前项目下的/config目录下 
  2. (外部) file:./           ：当前项目的根目录 
3. classpath:/config/：classpath的/config目录 
  4. classpath:/  ：classpath的根目录

<font color=red>注意：</font>

- 以上四个位置的文件都会被加载
- 加载顺序为1-4排列顺序，1的优先级最高,  4优先级最低
- 同一个配置在两个优先级文件中定义，低优先级会被忽略掉
- 某个配置在只存在于低优先级文件中, 也会生效。



#### 4.6 外部配置加载顺序

- 通过官网查看外部属性加载顺序： https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html

<font color=red>常见的外部配置：</font>

- 命令行参数配置: --server.port=2000

- 命令行参数配置外部配置文件路径: 

  ​    --spring.config.location=d://properties/application.properties

- 执行java -jar 命令的同级目录下 config/application.properties 

- 执行java -jar 命令的同级目录下 application.properties



#### 4.7 常见配置文件及优先级

1. 命令行 --server.port
2. 命令行 --spring.config.location
3. 配置文件properties/yml 中激活配置spring.profiles.active: pro
4. 执行java -jar 命令的同级目录下config/application.properties
5. 执行java -jar 命令的同级目录下application.properties
6. jar 包内部classpath:config/application.properties
7. jar 包内部classpath:application.properties
8. 同级目录下.properties  >  .yml  > .yaml

**小结：**

- 如上1-8 优先级递减
- 人为指定的配置永远大于默认读取的配置
- 命令行做的配置优先级普遍高于默认加载的配置



### 5. SpringBoot 整合其他框架

- springboot 整合 Junit 
- springboot 整合 Redis
- springboot 整合 Mybatis



#### 5.1 SpringBoot整合Junit

​	① 搭建SpringBoot工程 

​	② 引入starter-test起步依赖 

​	③ 编写测试类 

​	④ 添加测试相关注解 

​			• @RunWith(SpringRunner.class) 

​			• @SpringBootTest(classes = 启动类.class) 

​	⑤ 编写测试方法



##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.8.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    
    <groupId>com.itheima</groupId>
    <artifactId>springboot-redis</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-redis</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

##### SpringbootTestApplication.java

```java
package com.itheima.springboottest;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootTestApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootTestApplication.class, args);
    }

}
```

##### UserService.java

```java
package com.itheima.springboottest;

import org.springframework.stereotype.Service;

@Service
public class UserService {

    public void add() {
        System.out.println("add...");
    }
    
}
```

##### UserServiceTest.java

- 如果@SpringBootTest没有指定classes属性, 则当前测试类所属的包必须与@SpringBootApplication 注解所修饰的启动类, 包结构和层级相同, 否则启动将会发生异常。

```java
package com.itheima.springboottest;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * userService的测试类
 */

@RunWith(SpringRunner.class)
/**
 * 如果@SpringBootTest没有指定classes属性, 则当前测试类所属的包必须与   * @SpringBootApplication 注解所修饰的启动类, 包结构和层级相同
 */
//@SpringBootTest
@SpringBootTest(classes = SpringbootTestApplication.class)
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    public void testAdd() {
        userService.add();
    }
}
```





#### 5.2 SpringBoot整合Redis

​	① 搭建SpringBoot工程 

​	② 引入redis起步依赖 

​	③ 配置redis相关属性 

​	④ 注入RedisTemplate模板 

​	⑤ 编写测试方法，测试



##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.8.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.itheima</groupId>
    <artifactId>springboot-redis</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-redis</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

##### application.yml

```yaml
spring:
  redis:
    host: 127.0.0.1 # redis的主机ip
    port: 6379
```

##### SpringbootRedisApplication.java

```java
package com.itheima.springbootredis;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootRedisApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootRedisApplication.class, args);
    }

}

```

##### SpringbootRedisApplicationTests.java

- 如果@SpringBootTest没有指定classes属性, 则当前测试类所属的包必须与@SpringBootApplication 注解所修饰的启动类, 包结构和层级相同, 否则启动将会发生异常。

```java
package com.itheima.springbootredis;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.BoundValueOperations;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringbootRedisApplication.class)
public class SpringbootRedisApplicationTests {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    public void testSet() {
        //存入数据
        redisTemplate.boundValueOps("name").set("zhangsan");
    }

    @Test
    public void testGet() {
        //获取数据
        Object name = redisTemplate.boundValueOps("name").get();
        System.out.println(name);
    }

}

```



#### 5.3 SpringBoot整合Mybatis

​	① 搭建SpringBoot工程 

​	② 引入mybatis起步依赖，添加mysql驱动 

​	③ 编写DataSource和MyBatis相关配置 

​	④ 定义表和实体类 

​	⑤ 编写dao和mapper文件/纯注解开发 

​	⑥ 测试



##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.itheima</groupId>
    <artifactId>springboot-mybatis</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-mybatis</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <!--<scope>runtime</scope>-->
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

##### application.yml

```yaml
# datasource
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/springboot?serverTimezone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver


# mybatis
mybatis:
  mapper-locations: classpath:mapper/*Mapper.xml # mapper映射文件路径
  type-aliases-package: com.itheima.springbootmybatis.domain

  # config-location:  # 指定mybatis的核心配置文件

```

##### SpringbootMybatisApplication.java

```java
package com.itheima.springbootmybatis;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootMybatisApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootMybatisApplication.class, args);
    }

}

```

##### SpringbootMybatisApplicationTests.java

- 如果@SpringBootTest没有指定classes属性, 则当前测试类所属的包必须与@SpringBootApplication 注解所修饰的启动类, 包结构和层级相同, 否则启动将会发生异常。

```java
package com.itheima.springbootmybatis;

import com.itheima.springbootmybatis.domain.User;
import com.itheima.springbootmybatis.mapper.UserMapper;
import com.itheima.springbootmybatis.mapper.UserXmlMapper;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringbootMybatisApplication.class)
public class SpringbootMybatisApplicationTests {

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private UserXmlMapper userXmlMapper;


    @Test
    public void testFindAll() {
        List<User> list = userMapper.findAll();
        System.out.println(list);
    }
    @Test
    public void testFindAll2() {
        List<User> list = userXmlMapper.findAll();
        System.out.println(list);
    }

}
```

##### UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.springbootmybatis.mapper.UserXmlMapper">
   
    <select id="findAll" resultType="user">
        select * from t_user
    </select>
    
</mapper>
```

##### UserXmlMapper.java

```java
package com.itheima.springbootmybatis.mapper;

import com.itheima.springbootmybatis.domain.User;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Repository;

import java.util.List;

@Mapper
@Repository
public interface UserXmlMapper {

    public List<User> findAll();
}
```

##### User.java

```java
package com.itheima.springbootmybatis.domain;

public class User {
    
    private int id;
    private String username;
    private String password;


    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

##### user.sql

```sql

/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/`springboot` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci */;

USE `springboot`;

/*Table structure for table `t_user` */

DROP TABLE IF EXISTS `t_user`;

CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `password` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

/*Data for the table `t_user` */

insert  into `t_user`(`id`,`username`,`password`) values (1,'zhangsan','123'),(2,'lisi','234');

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

```









