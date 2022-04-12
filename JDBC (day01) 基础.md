# JDBC基础

### 1. 基本概念

- Java语言中的一款操作数据库的框架，十分优秀，很多高级操作数据库框架都是基于JDBC封装出来的，jdbc 在处理不同的关系型数据库软件时，提供了统一的规范（接口) ，这使得程序员只需要熟悉一套API就可以很好的操作各种数据库软件，比如：MySQL，Oracle ，DB2,  SQLServer,  PostgreSql。

### 2. 快速入门

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class JDBCDemo01 {
    public static void main(String[] args) throws Exception{
        //1.导入jar包
        //2.注册驱动
        Class.forName("com.mysql.jdbc.Driver");

        //3.获取连接
        Connection con = DriverManager.getConnection("jdbc:mysql://192.168.59.129:3306/db2","root","itheima");

        //4.获取执行者对象
        Statement stat = con.createStatement();

        //5.执行sql语句，并且接收结果
        String sql = "SELECT * FROM user";
        ResultSet rs = stat.executeQuery(sql);

        //6.处理结果
        while(rs.next()) {
            System.out.println(rs.getInt("id") + "\t" + rs.getString("name"));
        }

        //7.释放资源
        con.close();
        stat.close();
        con.close();
    }
}
```

#####  01DriverManager驱动管理对象 

- 该类负责加载mysql-connector-java-5.1.37-bin.jar 中的类进内存，此过程我们称之为 “加载驱动”。

- mysql5以前, 我们需要通过: Class.forName("com.mysql.jdbc.Driver"); 加载驱动，在5以后的版本，驱动包会自动加载数据库驱动；推荐写上。

- API 如下

  - static Connection getConnection(String url, String user, String password); 

  - 返回值：Connection 数据库连接对象 

  - 三个参数 ：

    url:    jdbc:mysql://ip地址(域名):端口号/数据库名称 

    user：用户名 

    password：密码

  ```java
  //3.获取连接
  Connectioncon = DriverManager.getConnection("jdbc:mysql://192.168.59.129:3306/db2","root","itheima");
  
  //java服务器 与mysql 服务器之间交互数据是要走网络的，所以操作之前，我们需要得到连接对象 Connection。
  /* 创建连接的四要素: 
         协议 jdbc:mysql:  
         ip:端口  192.168.59.129:3306
         授权  账号&密码  root itheima
  */
  
  ```

  

##### 02. Connection 与MySQL建立连接对象

  ① 获取能执行SQL语句的对象 

​           获取普通执行者对象：Statement createStatement(); 

​           获取预编译执行者对象：PreparedStatementprepareStatement(String sql); 

  ② 管理事务 开启事务：setAutoCommit(booleanautoCommit);     

​          参数为false，则开启事务。 

​          提交事务：commit(); 

​          回滚事务：rollback(); 

  ③ 释放资源 立即将数据库连接对象释放：void close();

##### 03. Statement执行sql语句的对象

  ① 执行DML语句：

​           intexecuteUpdate(String sql); 

​           返回值int：返回影响的行数。 

​           参数sql：可以执行insert、update、delete语句。 

  ② 执行DQL语句：ResultSetexecuteQuery(String sql); 

​           返回值ResultSet：封装查询的结果。 

​           参数sql：可以执行select语句。 

  ③ 释放资源 立即将执行者对象释放：void close();

##### 04. ResultSet结果集对象

① 判断结果集中是否还有数据：

​           boolean   next(); 

​           有数据返回true， 并将索引向下移动一行。 

​           没有数据返回false。 

② 获取结果集中的数据：

​            XXX getXxx("列名"); 

​            XXX代表数据类型(要获取某列数据，这一列的数据类型)。 

​            例如：String getString("name");  int getInt("age"); 

③ 释放资源 立即将结果集对象释放：void close();



### 3. 案例

- 需求: 使用JDBC完成对student表的CRUD操作

##### 01_jdbc_数据准备

```mysql
-- 创建db14数据库
CREATE DATABASE db14;

-- 使用db14数据库
USE db14;


-- 创建student表
CREATE TABLE student(
 sid INT PRIMARY KEY AUTO_INCREMENT,	-- 学生id
 NAME VARCHAR(20),			-- 学生姓名
 age INT,				-- 学生年龄
 birthday DATE				-- 学生生日
);

-- 添加数据
INSERT INTO student VALUES (NULL,'张三',23,'1999-09-23'),(NULL,'李四',24,'1998-08-10'),
(NULL,'王五',25,'1996-06-06'),(NULL,'赵六',26,'1994-10-20');
```

**javabean**

```java
package com.itheima02.domain;

import java.util.Date;

public class Student {
    private Integer sid;
    private String name;
    private Integer age;
    private Date birthday;

    public Student() {
    }

    public Student(Integer sid, String name, Integer age, Date birthday) {
        this.sid = sid;
        this.name = name;
        this.age = age;
        this.birthday = birthday;
    }

    public Integer getSid() {
        return sid;
    }

    public void setSid(Integer sid) {
        this.sid = sid;
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

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return "Student{" +
                "sid=" + sid +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", birthday=" + birthday +
                '}';
    }
}

```



##### 02. 需求一，查询所有学生信息

```java
    /*
        查询所有学生信息
     */
    @Override
    public ArrayList<Student> findAll() {
        ArrayList<Student> list = new ArrayList<>();
        Connection con = null;
        Statement stat = null;
        ResultSet rs = null;
        try{

            con = JDBCUtils.getConnection();

           //3.获取执行者对象
           stat = con.createStatement();

           //4.执行sql语句，并且接收返回的结果集
           String sql = "SELECT * FROM student";
           rs = stat.executeQuery(sql);

           //5.处理结果集
           while(rs.next()) {
               Integer sid = rs.getInt("sid");
               String name = rs.getString("name");
               Integer age = rs.getInt("age");
               Date birthday = rs.getDate("birthday");

               //封装Student对象
               Student stu = new Student(sid,name,age,birthday);

               //将student对象保存到集合中
               list.add(stu);
           }

       } catch(Exception e) {
           e.printStackTrace();
       } finally {
           //6.释放资源
           JDBCUtils.close(con,stat,rs);
       }
        //将集合对象返回
        return list;
    }
```

##### 03. 需求二，id指定学生查询信息

```java
    /*
        条件查询，根据id查询学生信息
     */
    @Override
    public Student findById(Integer id) {
        Student stu = new Student();
        Connection con = null;
        Statement stat = null;
        ResultSet rs = null;
        try{

            con = JDBCUtils.getConnection();

            //3.获取执行者对象
            stat = con.createStatement();

            //4.执行sql语句，并且接收返回的结果集
            String sql = "SELECT * FROM student WHERE sid='"+id+"'";
            rs = stat.executeQuery(sql);

            //5.处理结果集
            while(rs.next()) {
                Integer sid = rs.getInt("sid");
                String name = rs.getString("name");
                Integer age = rs.getInt("age");
                Date birthday = rs.getDate("birthday");

                //封装Student对象
                stu.setSid(sid);
                stu.setName(name);
                stu.setAge(age);
                stu.setBirthday(birthday);
            }

        } catch(Exception e) {
            e.printStackTrace();
        } finally {
            //6.释放资源
            JDBCUtils.close(con,stat,rs);
        }
        //将对象返回
        return stu;
    }
```

##### 04. 需求三，添加学生信息

```java
    /*
        添加学生信息
     */
    @Override
    public int insert(Student stu) {
        Connection con = null;
        Statement stat = null;
        int result = 0;
        try{
            con = JDBCUtils.getConnection();

            //3.获取执行者对象
            stat = con.createStatement();

            //4.执行sql语句，并且接收返回的结果集
            Date d = stu.getBirthday();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            String birthday = sdf.format(d);
            String sql = "INSERT INTO student VALUES ('"+stu.getSid()+"','"+stu.getName()+"','"+stu.getAge()+"','"+birthday+"')";
            result = stat.executeUpdate(sql);

        } catch(Exception e) {
            e.printStackTrace();
        } finally {
            //6.释放资源
            JDBCUtils.close(con,stat);
        }
        //将结果返回
        return result;
    }
```

##### 05. 需求四，修改学生信息

```java
    /*
        修改学生信息
     */
    @Override
    public int update(Student stu) {
        Connection con = null;
        Statement stat = null;
        int result = 0;
        try{
            con = JDBCUtils.getConnection();

            //3.获取执行者对象
            stat = con.createStatement();

            //4.执行sql语句，并且接收返回的结果集
            Date d = stu.getBirthday();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            String birthday = sdf.format(d);
            String sql = "UPDATE student SET sid='"+stu.getSid()+"',name='"+stu.getName()+"',age='"+stu.getAge()+"',birthday='"+birthday+"' WHERE sid='"+stu.getSid()+"'";
            result = stat.executeUpdate(sql);

        } catch(Exception e) {
            e.printStackTrace();
        } finally {
            //6.释放资源
            JDBCUtils.close(con,stat);
        }
        //将结果返回
        return result;
    }
```

##### 06. 需求五，指定学生删除信息

```java
    /*
        删除学生信息
     */
    @Override
    public int delete(Integer id) {
        Connection con = null;
        Statement stat = null;
        int result = 0;
        try{
            con = JDBCUtils.getConnection();

            //3.获取执行者对象
            stat = con.createStatement();

            //4.执行sql语句，并且接收返回的结果集
            String sql = "DELETE FROM student WHERE sid='"+id+"'";
            result = stat.executeUpdate(sql);

        } catch(Exception e) {
            e.printStackTrace();
        } finally {
            //6.释放资源
            JDBCUtils.close(con,stat);
        }
        //将结果返回
        return result;
    }
```



### 4. JDBC 工具类

- 工具类可以将重复的代码进行抽取，提高代码的复用性，降低代码的冗余。
- 在java中，工具类中的方法和属性都是静态的，因为工具类其特点也是与业务无关，不存在需要new很多个对象，使用类名调用方法即可；

##### 01. 抽取工具类

- 在src目录下创建config.properties配置文件

  ```properties
  #固定写法, mysql启动类
  driverClass=com.mysql.jdbc.Driver
  #连接地址
  url=jdbc:mysql://192.168.59.129:3306/db14
  #账号
  username=root
  #密码
  password=itheima
  ```

  **JDBCUtils.java**

  ```java
  package com.itheima02.utils;
  
  import java.io.InputStream;
  import java.sql.*;
  import java.util.Properties;
  
  /*
      JDBC工具类
   */
  public class JDBCUtils {
      //1.私有构造方法
      private JDBCUtils(){}
  
      //2.声明所需要的配置变量
      private static String driverClass;
      private static String url;
      private static String username;
      private static String password;
      private static Connection con;
  
      //3.提供静态代码块。读取配置文件的信息为变量赋值，注册驱动
      static{
          try {
              //读取配置文件的信息为变量赋值
              InputStream is = JDBCUtils.class.getClassLoader().getResourceAsStream("config.properties");
              Properties prop = new Properties();
              prop.load(is);
  
              driverClass = prop.getProperty("driverClass");
              url = prop.getProperty("url");
              username = prop.getProperty("username");
              password = prop.getProperty("password");
  
              //注册驱动
              Class.forName(driverClass);
  
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  
      //4.提供获取数据库连接方法
      public static Connection getConnection() {
          try {
              con = DriverManager.getConnection(url,username,password);
          } catch (SQLException e) {
              e.printStackTrace();
          }
  
          return con;
      }
  
      //5.提供释放资源的方法
      public static void close(Connection con, Statement stat, ResultSet rs) {
          if(con != null) {
              try {
                  con.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
  
          if(stat != null) {
              try {
                  stat.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
  
          if(rs != null) {
              try {
                  rs.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
      }
  
      //6.释放  连接 和 statement
      public static void close(Connection con, Statement stat) {
          if(con != null) {
              try {
                  con.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
  
          if(stat != null) {
              try {
                  stat.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  
  ```

  ##### **优化StudentDaoImpl.java** 

  ```java
  import com.itheima02.domain.Student;
  import com.itheima02.utils.JDBCUtils;
  
  import java.sql.*;
  import java.text.SimpleDateFormat;
  import java.util.ArrayList;
  import java.util.Date;
  
  public class StudentDaoImpl implements StudentDao {
      /*
          查询所有学生信息
       */
      @Override
      public ArrayList<Student> findAll() {
          ArrayList<Student> list = new ArrayList<>();
          Connection con = null;
          Statement stat = null;
          ResultSet rs = null;
          try{
  
              con = JDBCUtils.getConnection();
  
             //3.获取执行者对象
             stat = con.createStatement();
  
             //4.执行sql语句，并且接收返回的结果集
             String sql = "SELECT * FROM student";
             rs = stat.executeQuery(sql);
  
             //5.处理结果集
             while(rs.next()) {
                 Integer sid = rs.getInt("sid");
                 String name = rs.getString("name");
                 Integer age = rs.getInt("age");
                 Date birthday = rs.getDate("birthday");
  
                 //封装Student对象
                 Student stu = new Student(sid,name,age,birthday);
  
                 //将student对象保存到集合中
                 list.add(stu);
             }
  
         } catch(Exception e) {
             e.printStackTrace();
         } finally {
             //6.释放资源
             JDBCUtils.close(con,stat,rs);
         }
          //将集合对象返回
          return list;
      }
  
      /*
          条件查询，根据id查询学生信息
       */
      @Override
      public Student findById(Integer id) {
          Student stu = new Student();
          Connection con = null;
          Statement stat = null;
          ResultSet rs = null;
          try{
  
              con = JDBCUtils.getConnection();
  
              //3.获取执行者对象
              stat = con.createStatement();
  
              //4.执行sql语句，并且接收返回的结果集
              String sql = "SELECT * FROM student WHERE sid='"+id+"'";
              rs = stat.executeQuery(sql);
  
              //5.处理结果集
              while(rs.next()) {
                  Integer sid = rs.getInt("sid");
                  String name = rs.getString("name");
                  Integer age = rs.getInt("age");
                  Date birthday = rs.getDate("birthday");
  
                  //封装Student对象
                  stu.setSid(sid);
                  stu.setName(name);
                  stu.setAge(age);
                  stu.setBirthday(birthday);
              }
  
          } catch(Exception e) {
              e.printStackTrace();
          } finally {
              //6.释放资源
              JDBCUtils.close(con,stat,rs);
          }
          //将对象返回
          return stu;
      }
  
      /*
          添加学生信息
       */
      @Override
      public int insert(Student stu) {
          Connection con = null;
          Statement stat = null;
          int result = 0;
          try{
              con = JDBCUtils.getConnection();
  
              //3.获取执行者对象
              stat = con.createStatement();
  
              //4.执行sql语句，并且接收返回的结果集
              Date d = stu.getBirthday();
              SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
              String birthday = sdf.format(d);
              String sql = "INSERT INTO student VALUES ('"+stu.getSid()+"','"+stu.getName()+"','"+stu.getAge()+"','"+birthday+"')";
              result = stat.executeUpdate(sql);
  
          } catch(Exception e) {
              e.printStackTrace();
          } finally {
              //6.释放资源
              JDBCUtils.close(con,stat);
          }
          //将结果返回
          return result;
      }
  
      /*
          修改学生信息
       */
      @Override
      public int update(Student stu) {
          Connection con = null;
          Statement stat = null;
          int result = 0;
          try{
              con = JDBCUtils.getConnection();
  
              //3.获取执行者对象
              stat = con.createStatement();
  
              //4.执行sql语句，并且接收返回的结果集
              Date d = stu.getBirthday();
              SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
              String birthday = sdf.format(d);
              String sql = "UPDATE student SET sid='"+stu.getSid()+"',name='"+stu.getName()+"',age='"+stu.getAge()+"',birthday='"+birthday+"' WHERE sid='"+stu.getSid()+"'";
              result = stat.executeUpdate(sql);
  
          } catch(Exception e) {
              e.printStackTrace();
          } finally {
              //6.释放资源
              JDBCUtils.close(con,stat);
          }
          //将结果返回
          return result;
      }
  
      /*
          删除学生信息
       */
      @Override
      public int delete(Integer id) {
          Connection con = null;
          Statement stat = null;
          int result = 0;
          try{
              con = JDBCUtils.getConnection();
  
              //3.获取执行者对象
              stat = con.createStatement();
  
              //4.执行sql语句，并且接收返回的结果集
              String sql = "DELETE FROM student WHERE sid='"+id+"'";
              result = stat.executeUpdate(sql);
  
          } catch(Exception e) {
              e.printStackTrace();
          } finally {
              //6.释放资源
              JDBCUtils.close(con,stat);
          }
          //将结果返回
          return result;
      }
  }
  
  ```

#####  02.  Student表的CRUD操作整合页面    

​     **详见java项目**:  \08-JDBC（双元）\day01_jdbc基础\代码\JDBC基础网页版\





### 5. SQL注入攻击

- 就是利用sql语句的漏洞来对系统进行攻击

  > 用户通过web页面输入框，输入可以破坏java构建SQL的文本内容，达到串改的目的

##### 01_jdbc_sql注入攻击数据准备

```mysql
-- 创建用户表
CREATE TABLE USER(
	uid VARCHAR(50) PRIMARY KEY,	-- 用户id
	ucode VARCHAR(50),		-- 用户标识
	loginname VARCHAR(100),		-- 登录用户名
	PASSWORD VARCHAR(100),		-- 登录密码
	username VARCHAR(100),		-- 用户名
	gender VARCHAR(10),		-- 用户性别
	birthday DATE,			-- 出生日期
	dutydate DATE                   -- 入职日期
);

-- 添加一条测试数据
INSERT INTO USER VALUES ('11111111', 'zhangsan001', 'zhangsan', '1234', '张三', '男', '2008-10-28', '2018-10-28');

```

##### 02. SQL注入的原理

- SQL注入攻击的原理

  >  1) 按照正常道理来说，我们在密码处输入的所有内容，都应该认为是密码的组成 , 但是现在Statement对象在执行sql语句时，将密码的一部分内容当做查询条件来执行了

##### 03. SQL注入攻击的解决 

- PreparedStatement 预编译执行者对象

  > 1)  在执行sql语句之前，将sql语句进行提前编译。明确sql语句的格式后，就不会改变了。剩余的内容都会认为是参数！
  >
  > 2)  SQL语句中的参数使用?作为占位符

- 为?占位符赋值的方法：setXxx(参数1,参数2);

  > 1) Xxx代表：数据类型
  >
  > 2) 参数1：?的位置编号(编号从1开始) 
  >
  > 3) 参数2：?的实际参数

- 执行SQL语句  

  > 1) 执行insert、update、delete语句：intexecuteUpdate(); 
  >
  > 2) 执行select语句：ResultSetexecuteQuery();



### 6. JDBC 事务

#####  01. JDBC如何管理事务 

- 管理事务的功能类：Connection  开启事务：

  > 1)  setAutoCommit(booleanautoCommit);  参数为false，则开启事务。 
  >
  > 2)  提交事务：commit(); 
  >
  > 3)  回滚事务：rollback();

<font color=red>注意：事务的管理需要在业务层实现，因为dao层的功能要给很多模块提供功能的支撑，而有些模块是不需要事务的。</font>

##### 02. java事物操作示例代码

```java
       /*
       张三给李四转账500元
    */
    public void 转账() {

        //获取数据库连接对象
        Connection con = JDBCUtils.getConnection();
        Statement stat = null;
        try {

            //开启事务
            con.setAutoCommit(false);

            //1.获取执行者对象
            stat = con.createStatement();

            //2.构建SQL, 张三账户-500
            String sql_zhagnsan = "UPDATE account SET money=money-500 WHERE NAME='张三'";

            //3.执行SQL：sql_zhagnsan
            stat.executeUpdate(sql_zhagnsan);


            //可人为出错
            int a = 4/0;

            //4.构建SQL, 李四账户+500
            String sql_lisi = "UPDATE account SET money=money+500 WHERE NAME='李四'";

            //5.执行SQL：sql_lisi
            stat.executeUpdate(sql_lisi);

            //提交事务
            con.commit();

        }catch (Exception e){
            //回滚事务
            try {
                con.rollback();
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
            e.printStackTrace();
        } finally {
            //释放资源
            JDBCUtils.close(con,stat);
        }
    }
```

