# Maven高级

### 1. Maven简介

- Maven 它是一个项目管理工具，用了它，现在项目管理上，我们可以省掉好多事，非常便捷。

#### 1.1 学习目标

- 使用maven进行项目多模块开发
- 知道聚合的作用，能够使用聚合对所有maven模块进行统一管理
- 可以利用maven自定义属性，对依赖jar版本进行统一管理
- 能够理解性记忆：版本号的组成机构分别代表什么含义
- maven在打包项目时，如何跳过测试，至少知道命令方式
- 知道maven可以配置多环境这一回事即可，至于标签忘了随用随查
- 能够动手搭建nexus私服，并可以将项目打包发布到私服仓库中



### 2. 多模块开发SSM（重点）

- 将整体SSM中 javabean  controller  service  dao 从包拆分/升级为模块项目
- 模块与模块之间可以相互通过坐标进行导入
- 每个模块只关心自身依赖的jar资源和配置文件即可
- maven依赖分为：直接依赖&间接依赖
  - 已知：ssm_dao 依赖 ssm_pojo, 当 ssm_dao 想编译通过时，必须先将ssm_pojo 进行install操作，否则报错。
- dao 的xml 和 properties 配置 可以放在ssm_service 模块下，也可以将部分配置信息单独提取出去放到 dao 模块下。

#### ssm_pojo

- ssm_pojo  packaging标签 定义为jar（默认也是 jar）, 供其他工程根据

##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>ssm_pojo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

</project>
```

#### ssm_dao

- ssm_dao  packaging标签 定义为jar（默认也是 jar）, 供其他工程根据
- 相关其他坐标：
  - spring
  - mybatis
  - spirng整合mybatis
  - mysql驱动
  - druid连接池
  - pageHelper分页插件
  - 直接依赖ssm_pojo：使用该模块下的所有资源, 例如：.java .xml 等

##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  
    <modelVersion>4.0.0</modelVersion>
    <artifactId>ssm_dao</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <dependencies>
        <!--导入资源文件pojo-->
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>ssm_pojo</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

    </dependencies>

</project>
```

#### ssm_service

- ssm_service  packaging标签 定义为jar（默认也是 jar）, 供其他工程根据坐标导入
- 相关其他坐标：
  - spring
  - junit
  - spirng整合junit
  - mysql驱动
  - druid连接池
  - pageHelper分页插件
  - 直接依赖ssm_dao：使用该模块下的所有资源, 例如：.java .xml 等
  - 间接依赖ssm_pojo：使用该模块下的所有资源, 例如：.java .xml 等

##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 
    <modelVersion>4.0.0</modelVersion>
    <artifactId>ssm_service</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <dependencies>

        <!--导入资源文件dao-->
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>ssm_dao</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

    </dependencies>

</project>
```

#### ssm_controller

- ssm_service  packaging标签 定义为war（默认是 jar）, 需要部署到tomcat中
- 相关其他坐标：
  - spring
  - springmvc
  - jackson
  - servlet/jsp
  - tomcat7插件
  - 直接依赖ssm_service：使用该模块下的所有资源, 例如：.java .xml 等
  - 间接依赖ssm_dao 和 ssm_pojo：使用该模块下的所有资源, 例如：.java .xml 等

##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 
    <modelVersion>4.0.0</modelVersion>
    <artifactId>ssm_controller</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <dependencies>

        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>ssm_service</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

    </dependencies>

    <build>
        <!--设置插件-->
        <plugins>
            <!--具体的插件配置-->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

##### web.xml

- ```java
  //* 为通配符, 所有需要web.xml 加载的配置文件前缀要一致
  classpath*:applicationContext-*.xml  
  ```

- ```java
  //举例演示: 可以指定多个文件
  classpath:spring-service.xml,classpath:spring-dao.xml
  ```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

  <context-param>
    <param-name>contextConfigLocation</param-name>
<!--    <param-value>classpath*:applicationContext-service.xml,classpath*:applicationContext-dao.xml</param-value>-->
    <param-value>classpath*:applicationContext-*.xml</param-value>
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



#### 小结

   1）ssm_dao   ssm_pojo  ssm_service 不独立运行

   2）ssm_controller  项目把其他模块已jar包的方式导入到自己项目中使用，这种

做法仅仅在项目结构设计上与原来存在差异，运行时与原来没有任何差异。

   3）学已至此，将代码从一个包下拆分为三层架构三个包，再到拆分为多个项目工程，这种思想目的就是：分而治之，提供系统的维护性。





### 3. 聚合（重点）

- 作用：一键构建所有的maven多模块，并自行解决模块与模块之间的依赖顺序关系，举例：ssm_dao 依赖ssm_pojo, 聚合后一定会先构建ssm_pojo。



#### 3.1 制作聚合模块

- packaging标签 定义为war

- modules  module 标签将其他模块引入到聚合模块下


##### 示例代码

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>ssm</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--定义该工程用于进行构建管理-->
    <packaging>pom</packaging>

    <!--管理的工程列表-->
    <modules>
        <!--具体的工程名称-->
        <module>../ssm_controller</module>
        <module>../ssm_service</module>
        <module>../ssm_dao</module>
        <module>../ssm_pojo</module>
    </modules>

</project>

```



<font color=red>注意：</font>

 1）参与聚合的模块构建顺序与配置无关，与依赖有关，maven会自动识别处理。

 2）packaging标签取值：pom(聚合)，war（部署web压缩文件），jar（默认值）



### 4. 继承（重要）

- 模块与模块之间可以通过 parent 标签来实现继承关系
- 类似java，继承后，子模块可以使用父模块中的部分配置。

#### 示例代码

- 如下为：ssm_service、ssm_dao、ssm_pojo继承 ssm(packing为 pom) 实例代码
- 如下代码写在子模块pom.xml 中

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!--定义该工程的父工程-->
    <parent>
        <groupId>com.itheima</groupId>
        <artifactId>ssm</artifactId>
        <version>1.0-SNAPSHOT</version>
        <!--填写父工程的pom文件-->
        <relativePath>../ssm/pom.xml</relativePath>
    </parent>
    
</project>   
```



#### 4.1 继承依赖定义

- 在父工程中将依赖进行统一管理，子模块编写依赖坐标时，可以省略版本配置，具体版本取决于父模块中定义的版本。
- 如下代码仅仅只作为演示，已删除多余依赖。

##### 父模块pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>ssm</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--定义该工程用于进行构建管理-->
    <packaging>pom</packaging>

    <!--管理的工程列表-->
    <modules>
        <!--具体的工程名称-->
        <module>../ssm_controller</module>
        <module>../ssm_service</module>
        <module>../ssm_dao</module>
        <module>../ssm_pojo</module>
    </modules>

    <!--声明此处进行依赖管理-->
    <dependencyManagement>
        <!--具体的依赖-->
        <dependencies>
            <!--spring整合mybatis-->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis-spring</artifactId>
                <version>2.0.3</version>
            </dependency>
            <!--druid连接池-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.1.16</version>
            </dependency> 

        </dependencies>

    </dependencyManagement>

</project>
```

##### 子模块pom.xml

- 子模块使用 parent标签来指向父工程坐标即可，依赖继承这块，可以省略具体jar**版本**信息，编译时会去父工程中确认具体版本，groupId artifactId还是要写的。
- 子模块自身 坐标的 groupid 和 version 都可以继承父工程 的groupid 和 version

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!--定义该工程的父工程-->
    <parent>
      <groupId>com.itheima</groupId>
      <artifactId>ssm</artifactId>
      <version>1.0-SNAPSHOT</version>
      <!--填写父工程的pom文件-->
      <relativePath>../ssm/pom.xml</relativePath>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <artifactId>ssm_controller</artifactId>
    <packaging>war</packaging>

    <dependencies>
        <!--spring整合mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
        </dependency>
        <!--druid连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency> 

    </dependencies>
</project>
```



#### 4.2 常见的可继承资源（了解）

- groupId          项目组ID，项目坐标的核心元素
- version           项目版本，项目坐标的核心元素
- properties     自定义的maven属性
- dependencies  项目的依赖配置
- build              包括项目的源码目录配置，输出目录配置，插件配置，插件管理配置等
- repositories  项目仓库位置配置



#### <font color=red>4.3 注意事项</font>

- artifactId 不可以继承，每个工程都有自己的artifactId

- 工程之间的继承 与 工程的聚合 是两个完全独立的知识点，互不干扰

  1）聚合是为了快速构建工程

  2）继承是让子工程去继承资源实现资源统一管理

- 因为聚合模块没有任何代码可言，它不需要父模块。

- 聚合模块 和 父模块 pom.xml 中 packaging 均为：**pom**

- 聚合模块可以感知到其他模块，因为要对他们进行构建等操作，而继承中父工程是感知不到子工程存在。



### 5. 属性（重点）

- 自定义属性重点，其他均为了解

#### 5.1 分类

##### <font color=red>自定义属性（重要）</font>

- 重要-最为常用

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      
    <!--定义自定义属性-->
    <properties>
        <spring.version>5.1.9.RELEASE</spring.version>
        <junit.version>4.12</junit.version>
    </properties>

    <!--声明此处进行依赖管理-->
    <dependencyManagement>
        <!--具体的依赖-->
        <dependencies>
            <!--其他组件-->
            <!--junit单元测试-->
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
            </dependency>
            <!--spring整合junit-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <version>${spring.version}</version>
            </dependency>
        </dependencies>

    </dependencyManagement>

</project>
```



##### 项目属性

- 把当前项目pom.xml 标签 名直接当成属性来使用
-  常用的两个属性    `${project.basedir} `   项目的根目录(包含pom.xml文件的目录)，`${project.version}`项目版本 
- ${project.parent.version  }  获取当前工程父工程中的版本信息，用的不多。

```java
1）${project.build.sourceDirectory}:项目的主源码目录，默认为src/main/java/.
2）${project.build.testSourceDirectory}:项目的测试源码目录，默认为/src/test/java/.
3）${project.build.directory}:项目构建输出目录，默认为target/.
4）${project.build.outputDirectory}:项目主代码编译输出目录，默认为target/classes/.
5）${project.build.testOutputDirectory}:项目测试代码编译输出目录，默认为target/testclasses/.
6）${project.groupId}:项目的groupId.
7）${project.artifactId}:项目的artifactId.
8）${project.version}:项目的version,等同于${version}
9）${project.basedir} 项目的根目录(包含pom.xml文件的目录)
10）${project.build.finalName}:项目打包输出文件的名称，默认为${project.artifactId}${project.version}.
```

##### setting属性

- 从maven安装目录/conf/setting.xml 中读取标签当属性使用

```java
//所有 settings.xml 中的标签都可以通过 settings. 前缀进行引用 
//例如: ${settings.localRepository}指向用户本地仓库的地址
```

##### java系统属性

- 从maven安装目录/conf/setting.xml 中读取标签当属性使用

```java
//所有: Java系统属性都可以使用Maven属性引用，
//例如: ${user.home}指向了用户目录。
//可以通过命令行mvn help:system查看所有的Java系统属性
```

##### 环境变量属性

- 所有环境变量都可以使用以env.开头的Maven属性引用
- 也可以通过命令行mvn help:system查看所有环境变量。

```java
//必须保证M2_HOME JAVA_HOME 为真实存在的 环境变量名
//${env.M2_HOME } 代表Maven2的安装路径, 
//${env.JAVA_HOME} 指代了JAVA_HOME环境变量的值。
```



### 6. 版本管理 (理解记忆)

#### 6.1 工程版本

##### **SNAPSHOT（快照版本）**

​          1）项目开发过程中，为方便团队成员合作，解决模块间相互依赖和实时更新的问题，开发者对每个模块进行构建的时候，出处的临时性版本叫快照版本（测试阶段版本），很不稳定，也不推荐其他人使用。

​          2）快照版本会随着开发的进度不断更新

##### **RELEASE（发布版本）**

​          1）项目开发到进入阶段里程碑后，想团队外部发布较为稳定的版本，这种版本所对应的构件文件是稳定的，即便进行功能的后续开发，也不会改变当前发布版本的内容，这种版本称为**发布版本**，稳定的版本推荐其他人使用的版本。

​         

#### 6.2 工程版本号

##### 约定规范

- <主版本>.<次版本>.<增量版本>.<里程碑版本>

- 主版本：表示项目重大架构的变更，例如：spring5 相较于spring4的迭代
- 次版本：表示有较大的功能增加和变化，或者全面系统地修复漏洞
- 增量版本：表示有重大漏洞的修复
- 里程碑版本（一般不用）：表名一个版本里的里程碑。这样的版本同下一个正式版本相比，相对来说不是很稳定，有待更多的测试。

##### 范例

- 5.1.9.RELEASE   该版本号中没有使用到里程碑版本号





### 7. 资源配置 （了解）

- 项目中properties文件使用pom.xml 中自定义属性来读取值，了解即可。

#### pom.xml

- ${project.basedir} 获取当前pom.xml 所在项目（文件夹）的根路径

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>ssm</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--定义该工程用于进行构建管理-->
    <packaging>pom</packaging>
    
    <!--定义自定义属性-->
    <properties>
        <jdbc.url>jdbc:mysql://127.0.0.1:3306/ssm_db</jdbc.url>
    </properties>


    <build>
        <resources>
            <resource>
              <directory>${project.basedir}/src/main/resources</directory>
                <!--支持过滤，必写-->
                <filtering>true</filtering>
            </resource>
        </resources>
        <!--配置测试资源文件对应的信息-->
        <testResources>
            <testResource>
                                  <directory>${project.basedir}/src/test/resources</directory>
                <filtering>true</filtering>
            </testResource>
        </testResources>
    </build>
</project>
```

#### jdbc.properties

- ${jdbc.url} 动态从 pom.xml  properties 标签的子标签jdbc.url 中获取值

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=${jdbc.url}
jdbc.username=root
jdbc.password=itheima
```



### 8. 多环境配置 （了解）

- 真实开发中，存在很多环境，比如：测试 和 开发两个环境，一些大公司还有更细致的环境

- 运行命令：mvn  install -P  "环境名称"

  ```shell
  mvn install -P pro_env
  ```

  

#### pom.xml

- activeByDefault 标签  值 true 可以设置 那一套配置为默认配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>ssm</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--定义该工程用于进行构建管理-->
    <packaging>pom</packaging>
    
     <!--创建多环境-->
    <profiles>
        <!--定义具体的环境：生产环境-->
        <profile>
            <!--定义环境对应的唯一名称-->
            <id>pro_env</id>
            <!--定义环境中换用的属性值-->
            <properties>
                <jdbc.url>jdbc:mysql://127.1.1.1:3306/ssm_db</jdbc.url>
            </properties>
            <!--设置默认启动-->
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <!--定义具体的环境：开发环境-->
        <profile>
            <id>dep_env</id>
            <properties>
                <jdbc.url>jdbc:mysql://127.2.2.2:3306/ssm_db</jdbc.url>
            </properties>
        </profile>
    </profiles>
    
    
</project>
```

#### jdbc.properties

- ${jdbc.url} 动态从 pom.xml  properties 标签的子标签jdbc.url 中获取值

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=${jdbc.url}
jdbc.username=root
jdbc.password=itheima
```

#### <font color=red>小结</font>

一般都是在 profile 标签中 定义 properties 属性给jdbc.properties 这类属性文件更改值，至于依赖很少有不同的环境采用不同的依赖，所以没必要profile 标签中定义两套dependencyManagement 依赖配置



### 9. 跳过测试 （了解）

####   方式一 

- 运行命令：mvn  install -Dmaven.test.skip=true

  ```shell
  mvn install -Dmaven.test.skip=true -P pro_env
  ```

  

#### 方式二

- 利用插件跳过测试代码执行

##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>ssm</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--定义该工程用于进行构建管理-->
    <packaging>pom</packaging>

    <plugin>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.22.1</version>
        <configuration>
            <skipTests>true</skipTests>
            <includes> 
                 <include>**/User*Test.java</include>
            </includes>
            <excludes>
                <exclude>**/User*TestCase.java</exclude>
            </excludes>
        </configuration>
    </plugin>
    
</project>
```

#### 方式三

- 使用idea 窗口 maven--> 选中命令 ---> 点击上方蓝色小闪电



### 10.私服nexus

#### 10.1 介绍

- nexus 是 SonaType 公司研发的一款maven私服产品
- nexus 想tomcat redis 一样，需要我们下载，安装，运行后才可以使用，该软件的功能是为我们提供：maven 的私服仓库，因为是远程仓库，个人使用较少，有能力的公司会自己在服务器上搭建，供自己公司人使用。
- nexus 是基于java编写，使用类似tomcat同类产品jetty 来做web容器。



#### 10.2 安装、启动、配置

- 启动 nexus server 命令

  ```shell
  nexus.exe /run nexus
  ```

- nexus 管理页面地址（8081默认端口）

  http://localhost:8081

- 修改基础配置信息

  - 安装目录/etc/nexus-default.properties文件是nexus基础配置文件，例如：默认端口

- 修改运行配置信息

  - 安装目录/bin/nexus.vmoptions是nexus启动时与进程有关配置信息，例如：所占内存大小



#### 10.3 操作私服资源

- 登录：admin  密码在 F:\worksoft\nexus-3.20-win64\sonatype-work\nexus3\admin.password文件中
- 第一次登录后，会提示修改密码

**你需要知道的几件事**

   1）项目中需要使用私服要在pom.xml 做通过标签配置才可以

   2）项目配置坐标后，先找本地，找不到去找私服，还找不到，再去找公网中央仓库

   3）私服可以和中央仓库进行资源的同步

   4）nexus可以进行用户和用户权限管理

   5）nexus可以**分类**建立仓库，操作web页面可以将 jar 、war资源文件上传到仓库，可以对仓库和资源进行管理

   6）nexus中仓库有组的概念，可以将部分仓库归为一个组内，统一管理

**仓库分类：**

   1）宿主仓库

   2）代理仓库

   3）仓库组



#### 10.4 本地仓库与私服之间配置

- 本地仓库可以实现资源上传到私服
- 本地仓库也可以从私服下载资源

- 私服有地址，有账号密码认证
- 去本地maven安装目录找：F:\worksoft\apache-maven-3.3.9\conf\settings.xml 配置私服地址和账号密码

##### settings.xml

- 如下代码，只展示相关配置，非所有settings.xml 文件全部配置
- 阿里的远程仓库也可以视为：公共私服，一般给maven官方开放的仓库称为：中央仓库
- server 标签 子标签 id 要配置为具体的仓库名称
- mirror 标签 子标签 id 任意，具备语义化即可

```xml
<!--ؔ֨ 配置私服账号密码, 用于上传资源到私服 -->
<servers>
	<server>
		<id>heima-release</id>
		<username>admin</username>
		<password>admin</password>
	</server>
	<server>
		<id>heima-snapshots</id>
		<username>admin</username>
		<password>admin</password>
	</server>
</servers>

<!--ؔ֨ 配置私服地址: 用于从私服下载资源 -->
<mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
 <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
    <!--ؔ֨ 自己私服地址, -->
    <mirror>
        <id>nexus-heima</id>
        <mirrorOf>*</mirrorOf>
        <url>http://localhost:8081/repository/maven-public/</url>
    </mirror>
</mirrors>
```



#### 10.5 项目配置私服地址

- 项目（pom.xml）中配置私服地址，可以使用deploy 命令将项目成果物上传到 nexus搭建的私服仓库

##### pom.xml

- 私服地址配置在 所有模块父工程最为合适，父工程可以是聚合模块也可以不是
- distributionManagement 配置私服地址: 用于配合本地仓库将项目资源上传到私服 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>ssm</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--定义该工程用于进行构建管理-->
    <packaging>pom</packaging>

    <!--管理的工程列表-->
    <modules>
        <!--具体的工程名称-->
        <module>../ssm_controller</module>
        <module>../ssm_service</module>
        <module>../ssm_dao</module>
        <module>../ssm_pojo</module>
    </modules>

    
    <!-- 配置私服地址: 用于配合本地仓库将项目资源上传到私服 -->
    <distributionManagement>
        <repository>
            <!--要与settings.xml server子标签id 配置一致 -->
            <id>heima-release</id>
            <url>http://localhost:8081/repository/heima-release/</url>
        </repository>
        <snapshotRepository>
            <!--要与settings.xml server子标签id 配置一致 -->
            <id>heima-snapshots</id>
            <url>http://localhost:8081/repository/heima-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>

</project>
```

