# Mybatis基础

### 学习目标：

- 能够基于Mybatis API  + 映射配置文件（xml）实现增删改查操作。
- 能够熟知Mybatis 核心配置文件（xml）中的各项配置。
- 能够使用LOG4J 打印 mybatis执行SQL的日志。
- 基于Mybatis，自己改造学生管理系统，dao层代码实现。



###  1. MyBatis 快速入门

- **框架的介绍**: 框架就是一些公司基于软件开发过程中针对一些功能点研发出来的小工具，从自研开发软件角度看，也可以称呼它: "半成品软件"。
- **框架的优点**: 在开发软件系统时，使用框架研发可以让我们的代码更为精简，开发效率更快，维护性也高。
- **框架的缺点**: 在java中，框架都是基于jar包形式存在的，当我们使用框架时，框架自身正常工作也会创建很多的对象在内存中，势必我们要拿出来更多的内存。当然这点资源开销和它的优点相比，完全可以接受。

**为什么要选择Mybatis?**

jdbc

 1) sql夹在java业务代码块中，耦合度高导致维护麻烦

2）开发要编写大量的代码，大部分都是冗余的代码。

HIbernate和JPA

1) 长难度复杂SQL，对于Hibernate而言处理不容易

2） 内部自动生成的SQL，不容易做特殊优化

3）基于全映射的全自动框架，大量字段的POJO进行部分映射时比较困难。导致数据库性能下降。

Mybatis

1） SQL和java代码分开，所以功能便捷很清晰，代码专注业务，mybatis专注数据。

2） 可针对SQL做出优化。



##### 1.1 ORM 介绍

 **ORM(Object Relational Mapping)：对象关系映射** 

> Java语言的Javabean中的数据可以存储Mysql数据库的表中，表中的数据可以取出来后存储到Javabean对象中

**映射规则：** 数据表     <---->     类 

​                    表字段     <---->     类属性 

​                    表数据     <---->     对象

##### 1.2 MyBatis 介绍

- MyBatis 官网：http://www.mybatis.org/mybatis-3/

- MyBatis 是一个优秀的基于 Java 的持久层框架，它内部封装了 JDBC，使开发者只需要关注 SQL 语句本身， 而不需要花费精力去处理加载驱动、创建连接、创建执行者等复杂的操作。 

- MyBatis 通过 xml 或注解的方式将要执行的各种Statement 配置起来，并通过Java 对象和 Statement 中 SQL 的动态参数进行映射生成最终要执行的SQL 语句。 

- 最后 MyBatis 框架执行完SQL 并将结果映射为 Java 对象并返回。采用ORM 思想解决了实体和数据库映射 的问题，对 JDBC 进行了封装，屏蔽了JDBC API 底层访问细节，使我们不用与JDBC API 打交道，就可以 完成对数据库的持久化操作。

- 原始 JDBC 的操作问题分析 

  > 1) 频繁创建和销毁数据库的连接会造成系统资源浪费从而影响系统性能。 2) sql 语句在代码中硬编码，如果要修改sql 语句，就需要修改java 代码，造成代码不易维护。 
  >
  > 3) 查询操作时，需要手动将结果集中的数据封装到实体对象中。 
  >
  > 4) 增删改查操作需要参数时，需要手动将实体对象的数据设置到sql 语句的占位符。

- 原始 JDBC 的操作问题解决方案 

  > 1) 使用数据库连接池初始化连接资源。 
  >
  > 2) 将 sql 语句抽取到配置文件中。 
  >
  > 3) 使用反射、内省等底层技术，将实体与表进行属性与字段的自动映射。

<font color=red>说白了:  JDBC编写的代码多，很麻烦，Mybatis这类基于JDBC二次封装的框架，写更少的代码，做更多的事情，爽的飞起~~~</font>

#####  1.3 数据准备

```mysql
CREATE DATABASE db1;

USE db1;

CREATE TABLE student(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20),
	age INT
);

INSERT INTO student VALUES (NULL,'张三',23);
INSERT INTO student VALUES (NULL,'李四',24);
INSERT INTO student VALUES (NULL,'王五',25);

SELECT * FROM student;
```



##### 1.4 MyBatis 入门程序-示例代码

1） 数据准备。 

2） 导入 jar 包。 mybatis-3.5.3.jar       mysql-connector-java-5.1.37-bin.jar

3） 在 src 下创建映射配置文件。 

4） 在 src 下创建核心配置文件。 

5） 编写测试类完成相关API 的使用。 

6） 运行测试查看结果。



###### **1）核心配置文件 MyBatisConfig.xml**

- 放到项目src目录下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--configuration 核心根标签-->
<configuration>

    <!--environments配置数据库环境，环境可以有多个。default属性指定使用的是哪个-->
    <environments default="mysql">
        <!--environment配置数据库环境  id属性唯一标识-->
        <environment id="mysql">
            <!-- transactionManager事务管理。  type属性，采用JDBC默认的事务-->
            <transactionManager type="JDBC"></transactionManager>
            <!-- dataSource数据源信息   type属性 连接池-->
            <dataSource type="POOLED">
                <!-- property获取数据库连接的配置信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3307/lianxi" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>

    <!-- mappers引入映射配置文件 -->
    <mappers>
       <!-- mapper 引入指定的映射配置文件   resource属性指定映射配置文件的名称 -->
        <mapper resource="StudentMapper.xml"/>
    </mappers>
</configuration>
```

###### **2）映射配置文件StudentMapper.xml**

- 放到项目src目录下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--
    mapper：核心根标签
    namespace属性：名称空间
-->
<mapper namespace="StudentMapper">
    <!--
        select：查询功能的标签
        id属性：唯一标识
        resultType属性：指定结果映射对象类型
        parameterType属性：指定参数映射对象类型
    -->
    <select id="selectAll" resultType="student">
        SELECT * FROM student
    </select>

</mapper>
```

###### **3）测试代码**

```java
    /*
        查询全部
     */
    @Test
    public void selectAll() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过SqlSession工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //4.执行映射配置文件中的sql语句，并接收结果
        List<Student> list = sqlSession.selectList("StudentMapper.selectAll");

        //5.处理结果
        for (Student stu : list) {
            System.out.println(stu);
        }

        //6.释放资源
        sqlSession.close();
        is.close();
    }
```



##### 04. 快速入门小结

- Mybatis有**两类重要的配置文件**，**核心配置**文件去做一些全局的配置，比如：数据库的连接信息，事物管理，插件（分页），日志......等； 还有一个**SQL的映射文件**，该xml文件保存了SQL的映射信息，mybatis会将他们解析出来并执行；

- 框架 框架是一款半成品软件，我们可以基于框架继续开发，从而完成一些个性化的需求。 
- ORM 对象关系映射，数据和实体对象的映射。 
- MyBatis 是一个优秀的基于Java 的持久层框架，它内部封装了JDBC。 
- 入门步骤 导 jar 包。 编写映射配置文件。 编写核心配置文件。 使用相应的 API 来完成编码。



###  2. MyBatis 相关 API

##### 2.1 Resources

 org.apache.ibatis.io.Resources：加载资源的工具类。 

 **核心方法：**

| 返回值      | 方法名                               | 说明                                 |
| ----------- | ------------------------------------ | ------------------------------------ |
| InputStream | getResourceAsStream(String fileName) | 通过类加载器返回指定资源的字节输入流 |



##### 2.2 SqlSessionFactoryBuilder

org.apache.ibatis.session.SqlSessionFactoryBuilder：获取SqlSessionFactory 工厂对象的功能类。

 **核心方法：**

| 返回值            | 方法名               | 说明                                         |
| ----------------- | -------------------- | -------------------------------------------- |
| SqlSessionFactory | build(InputStreamis) | 通过指定资源字节输入流获取SqlSession工厂对象 |



##### 2.3 SqlSessionFactory

org.apache.ibatis.session.SqlSessionFactory：获取SqlSession 构建者对象的工厂接口。 

- <font color=red>openSession() 默认是false，手动提交事物，但只针对增删改，查询SQL无需理会，框架会帮我们提交</font>

**核心方法:**

| 返回值     | 方法名                         | 说明                                                         |
| ---------- | ------------------------------ | ------------------------------------------------------------ |
| SqlSession | openSession()                  | 获取SqlSession构建者对象，并开启手动提交事务                 |
| SqlSession | openSession(booleanautoCommit) | 获取SqlSession构建者对象，如果参数为true，则开启 自动提交事务 |

#####  2.4 SqlSession

org.apache.ibatis.session.SqlSession：构建者对象接口。用于执行SQL、管理事务、接口代理。

| 返回值  | 方法名                                          | 说明                           |
| ------- | ----------------------------------------------- | ------------------------------ |
| List<E> | selectList(String  statement,Object   paramter) | 执行查询语句，返回List集合     |
| T       | selectOne(String  statement,Object paramter)    | 执行查询语句，返回一个结果对象 |
| int     | insert(String statement,Object   paramter)      | 执行新增语句，返回影响行数     |
| int     | update(String statement,Object  paramter)       | 执行修改语句，返回影响行数     |
| int     | delete(String statement,Object paramter)        | 执行删除语句，返回影响行数     |
| void    | commit()                                        | 提交事务                       |
| void    | rollback()                                      | 回滚事务                       |
| T       | getMapper(Class<T> cls)                         | 获取指定接口的代理实现类对象   |
| void    | close()                                         | 释放资源                       |



##### 2.5 相关 API 小结

- Resources:   加载资源的工具类。 

- SqlSessionFactoryBuilder :  获取 SqlSessionFactory 工厂对象的功能类。 

- SqlSessionFactory:   获取 SqlSession 构建者对象的工厂接口。 指定事务的提交方式。 

- SqlSession:  构建者对象接口。 执行 SQL。 管理事务。 接口代理。

  

###  3. MyBatis 映射配置文件

##### 3.1 映射配置文件介绍

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--
    mapper：核心根标签
    namespace属性：名称空间
-->
<mapper namespace="StudentMapper">
    <!--
        select：查询功能的标签
        id属性：唯一标识
        resultType属性：指定结果映射对象类型
        parameterType属性：指定参数映射对象类型
    -->
    <select id="selectAll" resultType="student">
        SELECT * FROM student
    </select>
</mapper>    
```

##### 3.2 查询功能

**StudentMapper.xml**

```xml
<!--
标签: <select>：查询功能。 
    属性 
        id：唯一标识，配合名称空间使用。 
        parameterType：指定参数映射的对象类型。 
        resultType：指定结果映射的对象类型（增删改无此属性）。 
    SQL 获取参数 
        #{属性名}  
        
 -->
    <!-- 示例 -->
    <select id="selectAll" resultType="student">
        SELECT * FROM student
    </select>

    <select id="selectById" resultType="student" parameterType="int">
        SELECT * FROM student WHERE id = #{id}
    </select>
```

**测试**

```java
    /*
        根据id查询
     */
    @Test
    public void selectById() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //4.执行映射配置文件中的sql语句，并接收结果
        Student stu = sqlSession.selectOne("StudentMapper.selectById", 3);

        //5.处理结果
        System.out.println(stu);

        //6.释放资源
        sqlSession.close();
        is.close();
    }
```

##### 3.3 新增功能

**StudentMapper.xml**

```xml
<!--
标签: <insert>：新增功能。 
    属性 
        id：唯一标识，配合名称空间使用。 
        parameterType：指定参数映射的对象类型。 
    SQL 获取参数 
        #{属性名}     
 -->
    <!-- 示例 -->
    <insert id="insert" parameterType="student">
        INSERT INTO student VALUES (#{id},#{name},#{age})
    </insert>
```

**测试**

```java
    /*
        新增功能
     */
    @Test
    public void insert() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        //SqlSession sqlSession = sqlSessionFactory.openSession();
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.执行映射配置文件中的sql语句，并接收结果
        Student stu = new Student(5,"周七",27);
        int result = sqlSession.insert("StudentMapper.insert", stu);

        //5.提交事务
        //sqlSession.commit();

        //6.处理结果
        System.out.println(result);

        //7.释放资源
        sqlSession.close();
        is.close();
    }
```

##### 3.4 修改功能

**StudentMapper.xml**

```xml
<!--
标签: <update>：修改功能。 
    属性 
        id：唯一标识，配合名称空间使用。 
        parameterType：指定参数映射的对象类型。
    SQL 获取参数 
        #{属性名}          
 -->
    <!-- 示例 -->
    <update id="update" parameterType="student">
        UPDATE student SET name = #{name},age = #{age} WHERE id = #{id}
    </update>
```

**测试**

```java
    /*
        修改功能
     */
    @Test
    public void update() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.执行映射配置文件中的sql语句，并接收结果
        Student stu = new Student(5,"周七",37);
        int result = sqlSession.update("StudentMapper.update",stu);

        //5.提交事务
        sqlSession.commit();

        //6.处理结果
        System.out.println(result);

        //7.释放资源
        sqlSession.close();
        is.close();
    }
```

##### 3.5 删除功能

**StudentMapper.xml**

```xml
<!--
标签:  <delete>：删除功能。 
    属性 
        id：唯一标识，配合名称空间使用。 
        parameterType：指定参数映射的对象类型。
    SQL 获取参数 
        #{属性名}          
 -->
    <!-- 示例 -->
    <delete id="delete" parameterType="int">
        DELETE FROM student WHERE id = #{id}
    </delete>
```

**测试**

```java
    /*
        删除功能
     */
    @Test
    public void delete() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.执行映射配置文件中的sql语句，并接收结果
        int result = sqlSession.delete("StudentMapper.delete",5);

        //5.提交事务
        sqlSession.commit();

        //6.处理结果
        System.out.println(result);

        //7.释放资源
        sqlSession.close();
        is.close();
    }
```



##### 3.6 映射配置文件小结

```xml
<!--
映射配置文件包含了数据和对象之间的映射关系以及要执行的SQL 语句。
标签: 
    <mapper>：核心根标签。namespace 属性：名称空间
    <select>：查询功能标签。
    <insert>：新增功能标签。
    <update>：修改功能标签。
    <delete>：删除功能标签。

属性: 
    id 属性：唯一标识，配合名称空间使用 
    parameterType 属性：指定参数映射的对象类型
    resultType 属性：指定结果映射的对象类型
SQL 获取参数 
 	#{属性名}
 -->
```



###  4. MyBatis 核心配置文件

##### 4.1 核心配置文件介绍

- 核心配置文件包含了MyBatis 最核心的设置和属性信息。如数据库的连接、事务、连接池信息等

**MyBatisConfig.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--configuration 核心根标签-->
<configuration>

    <!--environments配置数据库环境，环境可以有多个。default属性指定使用的是哪个-->
    <environments default="mysql">
        <!--environment配置数据库环境  id属性唯一标识-->
        <environment id="mysql">
            <!-- transactionManager事务管理。  type属性，采用JDBC默认的事务-->
            <transactionManager type="JDBC"></transactionManager>
            <!-- dataSource数据源信息   type属性 连接池-->
            <dataSource type="POOLED">
                <!-- property获取数据库连接的配置信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3307/lianxi" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>

    <!-- mappers引入映射配置文件 -->
    <mappers>
       <!-- mapper 引入指定的映射配置文件   resource属性指定映射配置文件的名称 -->
        <mapper resource="StudentMapper.xml"/>
    </mappers>
</configuration>
```

##### 4.2 数据库连接配置文件引入

**jdbc.properties**

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://192.168.59.143:3306/db1
username=root
password=itheima
```

**MyBatisConfig.xml**

**标签**: properties 引入数据库连接配置文件
    **-  属性** 
       resource：数据库连接配置文件路径 
    **-  获取数据库连接参数** 
       ${键名}

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--configuration 核心根标签-->
<configuration>

    <!--
    标签: <properties> 引入数据库连接配置文件
    属性 
       resource：数据库连接配置文件路径 
    获取数据库连接参数 
       ${键名}
    -->
    <properties resource="jdbc.properties"/>

    <!--environments配置数据库环境，环境可以有多个。default属性指定使用的是哪个-->
    <environments default="mysql">
        <!--environment配置数据库环境  id属性唯一标识-->
        <environment id="mysql">
            <!-- transactionManager事务管理。  type属性，采用JDBC默认的事务-->
            <transactionManager type="JDBC"></transactionManager>
            <!-- dataSource数据源信息   type属性 连接池-->
            <dataSource type="POOLED">
                <!-- property获取数据库连接的配置信息 -->
                <property name="driver" value="${driver}" />
                <property name="url" value="${url}" />
                <property name="username" value="${username}" />
                <property name="password" value="${password}" />
            </dataSource>
        </environment>
    </environments>

    <!-- mappers引入映射配置文件 -->
    <mappers>
        <!-- mapper 引入指定的映射配置文件   resource属性指定映射配置文件的名称 -->
        <mapper resource="StudentMapper.xml"/>
    </mappers>
</configuration>
```



##### 4.3 起别名

在**核心配置文件**中通过标签进行配置，为**映射文件**在使用**参数和返回值类型**时可以通过别名处理。

**MyBatisConfig.xml**

- 方式一：可以自己随意定义类的别名为任意字符串
- 方式二（推荐使用）: package标签 为莫个包下（包含子包下）所有类批量起别名，name属性指向包路径，默认别名为**类名小写**
- <font color=red>推荐方式二:   因为我们的Javabean都是存放在一个package下的，配置一次，以后即使在创建一个新的Javabean类，也不用去核心配置中修改配置了</font>

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--configuration 核心根标签-->
<configuration>

    <!-- 起别名/方式一 -->
    <!--
        <typeAliases>
            <typeAlias type="com.itheima.bean.Student" alias="student"/>
        </typeAliases>
     -->
    
    <!-- 起别名/方式二, 别名默认为类名首字母小写 -->
    
    <typeAliases>
        <package name="com.itheima.bean"/>
    </typeAliases>

</configuration>
```



###### Mybatis默认对基础类型包装类提供的别名

| 别名    | 数据类型          |
| ------- | ----------------- |
| string  | java.lang.String  |
| long    | java.lang.Long    |
| int     | java.lang.Integer |
| double  | java.lang.Double  |
| boolean | java.lang.Boolean |
| .....   | .....             |



##### 4.4 核心配置文件小结

核心配置文件包含了MyBatis 最核心的设置和属性信息。如数据库的连接、事务、连接池信息等。

```xml
<!--
    <configuration>：核心根标签。 
    <properties>：引入数据库连接信息配置文件标签。
    <typeAliases>：起别名标签。 
    <environments>：配置数据库环境标签。
    <environment>：配置数据库信息标签, 可以配置多个。 
    <transactionManager>：事务管理标签。 
    <dataSource>：数据源标签。 
    <property>：数据库连接信息标签。 
    <mappers>：引入映射配置文件标签。
-->
```



###  5. MyBatis 传统方式实现 Dao 层

##### 5.1 Dao 层传统实现方式

**分层思想**：控制层(controller)、业务层(service)、持久层(dao)。 

**调用流程：**

html 页面   ------>   控制层(controller) ------>   业务层(service)  ------>  持久层(dao)  ------>  DB(Mysql)

##### 5.2 传统方式Dao-功能实现

**StudentMapperImpl.java**

```java
package com.itheima.mapper.impl;

import com.itheima.bean.Student;
import com.itheima.mapper.StudentMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;
/*
    持久层实现类
 */
public class StudentMapperImpl implements StudentMapper {

    /*
        查询全部
     */
    @Override
    public List<Student> selectAll() {
        List<Student> list = null;
        SqlSession sqlSession = null;
        InputStream is = null;
        try{
            //1.加载核心配置文件
            is = Resources.getResourceAsStream("MyBatisConfig.xml");

            //2.获取SqlSession工厂对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

            //3.通过工厂对象获取SqlSession对象
            sqlSession = sqlSessionFactory.openSession(true);

            //4.执行映射配置文件中的sql语句，并接收结果
            list = sqlSession.selectList("StudentMapper.selectAll");

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.释放资源
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

        //6.返回结果
        return list;
    }

    /*
        根据id查询
     */
    @Override
    public Student selectById(Integer id) {
        Student stu = null;
        SqlSession sqlSession = null;
        InputStream is = null;
        try{
            //1.加载核心配置文件
            is = Resources.getResourceAsStream("MyBatisConfig.xml");

            //2.获取SqlSession工厂对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

            //3.通过工厂对象获取SqlSession对象
            sqlSession = sqlSessionFactory.openSession(true);

            //4.执行映射配置文件中的sql语句，并接收结果
            stu = sqlSession.selectOne("StudentMapper.selectById",id);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.释放资源
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

        //6.返回结果
        return stu;
    }

    /*
        新增功能
     */
    @Override
    public Integer insert(Student stu) {
        Integer result = null;
        SqlSession sqlSession = null;
        InputStream is = null;
        try{
            //1.加载核心配置文件
            is = Resources.getResourceAsStream("MyBatisConfig.xml");

            //2.获取SqlSession工厂对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

            //3.通过工厂对象获取SqlSession对象
            sqlSession = sqlSessionFactory.openSession(true);

            //4.执行映射配置文件中的sql语句，并接收结果
            result = sqlSession.insert("StudentMapper.insert",stu);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.释放资源
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

        //6.返回结果
        return result;
    }

    /*
        修改功能
     */
    @Override
    public Integer update(Student stu) {
        Integer result = null;
        SqlSession sqlSession = null;
        InputStream is = null;
        try{
            //1.加载核心配置文件
            is = Resources.getResourceAsStream("MyBatisConfig.xml");

            //2.获取SqlSession工厂对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

            //3.通过工厂对象获取SqlSession对象
            sqlSession = sqlSessionFactory.openSession(true);

            //4.执行映射配置文件中的sql语句，并接收结果
            result = sqlSession.update("StudentMapper.update",stu);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.释放资源
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

        //6.返回结果
        return result;
    }

    /*
        删除功能
     */
    @Override
    public Integer delete(Integer id) {
        Integer result = null;
        SqlSession sqlSession = null;
        InputStream is = null;
        try{
            //1.加载核心配置文件
            is = Resources.getResourceAsStream("MyBatisConfig.xml");

            //2.获取SqlSession工厂对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

            //3.通过工厂对象获取SqlSession对象
            sqlSession = sqlSessionFactory.openSession(true);

            //4.执行映射配置文件中的sql语句，并接收结果
            result = sqlSession.delete("StudentMapper.delete",id);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.释放资源
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

        //6.返回结果
        return result;
    }
}

```



### 6. Log4j-日志框架

1） 导入 jar 包。 log4j-1.2.17.jar

2） 修改核心配置文件。

3）  在 src 下编写 LOG4J 配置文件  log4j.properties。

**日志级别**:  ERROR WARN INFO DEBUG

- 如果不想看DEBUG级别日志，可以在**log4j.properties**中做如下配置:

  > log4j.rootLogger=INFO , stdout

**log4j.properties**

```properties
# Global logging configuration
# ERROR WARN INFO DEBUG
log4j.rootLogger=DEBUG, stdout
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

**MyBatisConfig.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--configuration 核心根标签-->
<configuration>
    
   <!--配置LOG4J-->
    <settings>
        <setting name="logImpl" value="log4j"/>
    </settings>
    
</configuration>
```

### 7. 扩展

#### 7.1 驼峰命名法

- 数据库字段命名中存在多个单词使用下划线_ 分隔开，实体中对应的属性命名为 首个单词首字母小写，其他单词首字母大写，举例如下：

  > user表字段名:  user_name, pass_word, user_pay_status
  >
  > User实体属性对应命名: userName, passWrod, userPayStatus

在Mybatis核心配置文件中，增加如下配置:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--configuration 核心根标签-->
<configuration>
    
   <!--配置驼峰命名映射, 默认false不支持-->
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    
</configuration>
```



#### 7.2 Javabean起别名

在使用package标签批量起别名的情况下，也可以使用@Alias注解来为Javabean在重新起别名。

```java
@Alias("person")
public class Person {
    private Integer id;     //主键id
    private String name;    //人的姓名
    private Integer age;    //人的年龄

    public Person() {
    }
}
```



#### 7.3 insert操作获取自增主键值

```xml
<!-- 
属性: 
    useGeneratedKeys="true" 

    keyProperty="stuid" 
   		keyProperty属性的值要与parameterType指定的Javabean对象中主键属性命名一致。
 
public class Student {
    private Integer stuid;
}
-->
<insert id="insert" parameterType="student" useGeneratedKeys="true" keyProperty="stuid">
        INSERT INTO student VALUES (#{id},#{name},#{age})
</insert>
```

#### 7.4 Mapper.xml 入参注意事项

- 当参数是具体 数值，并且仅有一个时，#{} 中随意写，比如：

  ```xml
  sqlSession.selectOne("StudentMapper.selectById", 3);
  
  <select id="selectById" resultType="student" parameterType="int">
          SELECT * FROM student WHERE id = #{aaa} <!--bbb也行 -->
  </select>
  ```

- 当参数是Javabean对象或者map时，#{} 中必须写 javabean的属性名 或者 map 中key的名字。

  ```xml
  Student stu = new Student(8,"周七",27);
  int result = sqlSession.insert("StudentMapper.insert", stu);
  
  <insert id="insert" parameterType="com.itheima.Student">
          INSERT INTO student VALUES (#{id},#{name},#{age})
  </insert>
  
  ```

  ```xml
  Map<String, Object> map = new HashMap<>();
  map.put("id", 10);
  map.put("name", "周七");
  map.put("age", 27);
  int result = sqlSession.insert("StudentMapper.insert", map);
  
  <insert id="insert" parameterType="java.util.Map">
  	INSERT INTO student VALUES (#{id},#{name},#{age})
  </insert>
  ```

  

