# JDBC高级

### 1. 数据库连接池

#####  01数据库连接池的概念

- 计算机领域里，计算机与自己算计之间通信，需要走网络连接的协议，都势必会在建立连接上耗费掉一些资源，比如：TCP的三次握手和四次挥手。
- 连接池就是将 建立的网络 “连接” 管理起来，达到<font color=red>**重复利用**</font>的效果，来节省建立连接和销毁连接所消耗的资源。
- 一个优秀的数据库连接池技术，势必可以对连接进行管理的，比如：池子中需要提前准备多少个连接，空闲的连接有多少，池子最多可以装载多少个连接，正在使用的连接有多少，繁忙的连接又有多少，当一个新线程需要一个空闲的连接操作数据库时，连接池要合理的对当前线程的诉求进行处理。

### 2. 自定义数据库连接池

##### 01. DataSource 接口概述

>  1） javax.sql.DataSource接口：数据源(数据库连接池)。Java 官方提供的数据库连接池规范(接口) 
>
>  2） 如果想完成数据库连接池技术，就必须实现DataSource 接口 
>
>  3） 核心功能：获取数据库连接对象：Connection getConnection();

##### 02. 自定义数据库连接池 

> ① 定义一个类，实现DataSource 接口。 
>
> ② 定义一个容器，用于保存多个Connection 连接对象。 
>
> ③ 定义静态代码块，通过JDBC 工具类获取 10 个连接保存到容器中。 
>
> ④ 重写 getConnection 方法，从容器中获取一个连接并返回。 
>
> ⑤ 定义 getSize 方法，用于获取容器的大小并返回。
>
> **实例代码如下:** 
>
> ```java
> package com.itheima01;
> 
> import com.itheima.utils.JDBCUtils;
> 
> import javax.sql.DataSource;
> import java.io.PrintWriter;
> import java.lang.reflect.InvocationHandler;
> import java.lang.reflect.Method;
> import java.lang.reflect.Proxy;
> import java.sql.Connection;
> import java.sql.SQLException;
> import java.sql.SQLFeatureNotSupportedException;
> import java.util.ArrayList;
> import java.util.Collections;
> import java.util.List;
> import java.util.logging.Logger;
> 
> public class MyDataSource implements DataSource {
>     //1.准备一个容器。用于保存多个数据库连接对象
>     private static List<Connection> pool = Collections.synchronizedList(new ArrayList<>());
> 
>     //2.定义静态代码块,获取多个连接对象保存到容器中
>     static{
>         for(int i = 1; i <= 10; i++) {
>             Connection con = JDBCUtils.getConnection();
>             pool.add(con);
>         }
>     }
>     
>     //3.重写getConnection方法，用于返回一个连接对象
>     @Override
>     public Connection getConnection() throws SQLException {
>         if(pool.size() > 0) {
>             Connection con = pool.remove(0);
>             //通过自定义的连接对象 对原有的连接对象进行包装
>             //MyConnection2 myCon = new MyConnection2(con,pool);
>             MyConnection3 myCon = new MyConnection3(con,pool);
>             return myCon;
>         }else {
>             throw new RuntimeException("连接数量已用尽");
>         }
>     }
> 
>     //4.提供一个获取连接池大小的方法
>     public int getSize() {
>         return pool.size();
>     }
> 
>     @Override
>     public Connection getConnection(String username, String password) throws SQLException {
>         return null;
>     }
> 
>     @Override
>     public <T> T unwrap(Class<T> iface) throws SQLException {
>         return null;
>     }
> 
>     @Override
>     public boolean isWrapperFor(Class<?> iface) throws SQLException {
>         return false;
>     }
> 
>     @Override
>     public PrintWriter getLogWriter() throws SQLException {
>         return null;
>     }
> 
>     @Override
>     public void setLogWriter(PrintWriter out) throws SQLException {
> 
>     }
> 
>     @Override
>     public void setLoginTimeout(int seconds) throws SQLException {
> 
>     }
> 
>     @Override
>     public int getLoginTimeout() throws SQLException {
>         return 0;
>     }
> 
>     @Override
>     public Logger getParentLogger() throws SQLFeatureNotSupportedException {
>         return null;
>     }
> }
> 
> ```
>
> 

##### 03. 测试代码

```java
package com.itheima01;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class MyDataSourceTest {
    public static void main(String[] args) throws Exception{
        //1.创建连接池对象
        MyDataSource dataSource = new MyDataSource();

        System.out.println("使用之前的数量：" + dataSource.getSize());

        //2.通过连接池对象获取连接对象
        Connection con = dataSource.getConnection();
        System.out.println(con.getClass());

        //3.查询学生表的全部信息
        String sql = "SELECT * FROM student";
        PreparedStatement pst = con.prepareStatement(sql);

        //4.执行sql语句，接收结果集
        ResultSet rs = pst.executeQuery();

        //5.处理结果集
        while(rs.next()) {
            System.out.println(rs.getInt("sid") + "\t" + rs.getString("name") + "\t" + rs.getInt("age") + "\t" + rs.getDate("birthday"));
        }

        //6.释放资源
        rs.close();
        pst.close();
        con.close();    // 用完以后，关闭连接

        System.out.println("使用之后的数量：" + dataSource.getSize());
    }
}

```



### 3. 归还连接-继承方式

- 继承方式归还数据库连接的思想。 

  > 1) 通过打印连接对象，发现DriverManager 获取的连接实现类是JDBC4Connection 
  >
  > 2) 那我们就可以自定义一个类，继承JDBC4Connection 这个类，重写 close() 方法，完成连接对象的归还

- 继承方式归还数据库连接的实现步骤。 

  > ① 定义一个类，继承JDBC4Connection 。 
  >
  > ② 定义 Connection 连接对象和连接池容器对象的成员变量。 
  >
  > ③ 通过有参构造方法完成对成员变量的赋值。 
  >
  > ④ 重写 close 方法，将连接对象添加到池中。

- 继承方式归还数据库连接存在的问题。 

  >  1) 通过查看 JDBC 工具类获取连接的方法发现：我们虽然自定义了一个子类，完成了归还连接的操作。但是 DriverManager 获取的还是JDBC4Connection 这个对象，并不是我们的子类对象，而我们又不能整体 去修改驱动包中类的功能，所继承这种方式<font color=red>行不通！</font>



### 4. 归还连接-装饰设计模式

- 装饰设计模式归还数据库连接的思想。

  > 1)  我们可以自定义一个类，实现Connection 接口。这样就具备了和JDBC4Connection 相同的行为了 
  >
  > 
  >
  > 2)  重写 close() 方法，完成连接的归还。其余的功能还调用mysql 驱动包实现类原有的方法即可

- 装饰设计模式归还数据库连接的实现步骤。 

  > ① 定义一个类，实现Connection 接口 
  >
  > ② 定义 Connection 连接对象和连接池容器对象的成员变量 
  >
  > ③ 通过有参构造方法完成对成员变量的赋值 
  >
  > ④ 重写 close() 方法，将连接对象添加到池中 
  >
  > ⑤ 剩余方法，只需要调用mysql 驱动包的连接对象完成即可 
  >
  > ⑥ 在自定义连接池中，将获取的连接对象通过自定义连接对象进行包装
  >
  > 

- 装饰设计模式归还数据库连接存在的问题。

  > 实现 Connection 接口后，有大量的方法需要在自定义类中进行重写

##### 01示例代码

```java
package com.itheima02;

import java.sql.*;
import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.concurrent.Executor;

/*
    1.定义一个类，实现Connection接口
    2.定义连接对象和连接池容器对象的成员变量
    3.通过有参构造方法为成员变量赋值
    4.重写close方法，完成归还连接
    5.剩余方法，还是调用原有的连接对象中的功能即可
 */
//1.定义一个类，实现Connection接口
public class MyConnection2 implements Connection{

    //2.定义连接对象和连接池容器对象的成员变量
    private Connection con;
    private List<Connection> pool;

    //3.通过有参构造方法为成员变量赋值
    public MyConnection2(Connection con,List<Connection> pool){
        this.con = con;
        this.pool = pool;
    }

    //4.重写close方法，完成归还连接
    @Override
    public void close() throws SQLException {
        pool.add(con);
    }

    //5.剩余方法，还是调用原有的连接对象中的功能即可
    @Override
    public Statement createStatement() throws SQLException {
        return con.createStatement();
    }

    @Override
    public PreparedStatement prepareStatement(String sql) throws SQLException {
        return con.prepareStatement(sql);
    }

    @Override
    public CallableStatement prepareCall(String sql) throws SQLException {
        return con.prepareCall(sql);
    }

    @Override
    public String nativeSQL(String sql) throws SQLException {
        return con.nativeSQL(sql);
    }

    @Override
    public void setAutoCommit(boolean autoCommit) throws SQLException {
        con.setAutoCommit(autoCommit);
    }

    @Override
    public boolean getAutoCommit() throws SQLException {
        return con.getAutoCommit();
    }

    @Override
    public void commit() throws SQLException {
        con.commit();
    }

    @Override
    public void rollback() throws SQLException {
        con.rollback();
    }

    @Override
    public boolean isClosed() throws SQLException {
        return con.isClosed();
    }

    @Override
    public DatabaseMetaData getMetaData() throws SQLException {
        return con.getMetaData();
    }

    @Override
    public void setReadOnly(boolean readOnly) throws SQLException {
        con.setReadOnly(readOnly);
    }

    @Override
    public boolean isReadOnly() throws SQLException {
        return con.isReadOnly();
    }

    @Override
    public void setCatalog(String catalog) throws SQLException {
        con.setCatalog(catalog);
    }

    @Override
    public String getCatalog() throws SQLException {
        return con.getCatalog();
    }

    @Override
    public void setTransactionIsolation(int level) throws SQLException {
        con.setTransactionIsolation(level);
    }

    @Override
    public int getTransactionIsolation() throws SQLException {
        return con.getTransactionIsolation();
    }

    @Override
    public SQLWarning getWarnings() throws SQLException {
        return con.getWarnings();
    }

    @Override
    public void clearWarnings() throws SQLException {
        con.clearWarnings();
    }

    @Override
    public Statement createStatement(int resultSetType, int resultSetConcurrency) throws SQLException {
        return con.createStatement(resultSetType,resultSetConcurrency);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency) throws SQLException {
        return con.prepareStatement(sql,resultSetType,resultSetConcurrency);
    }

    @Override
    public CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency) throws SQLException {
        return con.prepareCall(sql,resultSetType,resultSetConcurrency);
    }

    @Override
    public Map<String, Class<?>> getTypeMap() throws SQLException {
        return con.getTypeMap();
    }

    @Override
    public void setTypeMap(Map<String, Class<?>> map) throws SQLException {
        con.setTypeMap(map);
    }

    @Override
    public void setHoldability(int holdability) throws SQLException {
        con.setHoldability(holdability);
    }

    @Override
    public int getHoldability() throws SQLException {
        return con.getHoldability();
    }

    @Override
    public Savepoint setSavepoint() throws SQLException {
        return con.setSavepoint();
    }

    @Override
    public Savepoint setSavepoint(String name) throws SQLException {
        return con.setSavepoint(name);
    }

    @Override
    public void rollback(Savepoint savepoint) throws SQLException {
        con.rollback(savepoint);
    }

    @Override
    public void releaseSavepoint(Savepoint savepoint) throws SQLException {
        con.releaseSavepoint(savepoint);
    }

    @Override
    public Statement createStatement(int resultSetType, int resultSetConcurrency, int resultSetHoldability) throws SQLException {
        return con.createStatement(resultSetType,resultSetConcurrency,resultSetHoldability);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability) throws SQLException {
        return con.prepareStatement(sql,resultSetType,resultSetConcurrency,resultSetHoldability);
    }

    @Override
    public CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability) throws SQLException {
        return con.prepareCall(sql,resultSetType,resultSetConcurrency,resultSetHoldability);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int autoGeneratedKeys) throws SQLException {
        return con.prepareStatement(sql,autoGeneratedKeys);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int[] columnIndexes) throws SQLException {
        return con.prepareStatement(sql,columnIndexes);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, String[] columnNames) throws SQLException {
        return con.prepareStatement(sql,columnNames);
    }

    @Override
    public Clob createClob() throws SQLException {
        return con.createClob();
    }

    @Override
    public Blob createBlob() throws SQLException {
        return con.createBlob();
    }

    @Override
    public NClob createNClob() throws SQLException {
        return con.createNClob();
    }

    @Override
    public SQLXML createSQLXML() throws SQLException {
        return con.createSQLXML();
    }

    @Override
    public boolean isValid(int timeout) throws SQLException {
        return con.isValid(timeout);
    }

    @Override
    public void setClientInfo(String name, String value) throws SQLClientInfoException {
        con.setClientInfo(name,value);
    }

    @Override
    public void setClientInfo(Properties properties) throws SQLClientInfoException {
        con.setClientInfo(properties);
    }

    @Override
    public String getClientInfo(String name) throws SQLException {
        return con.getClientInfo(name);
    }

    @Override
    public Properties getClientInfo() throws SQLException {
        return con.getClientInfo();
    }

    @Override
    public Array createArrayOf(String typeName, Object[] elements) throws SQLException {
        return con.createArrayOf(typeName,elements);
    }

    @Override
    public Struct createStruct(String typeName, Object[] attributes) throws SQLException {
        return con.createStruct(typeName,attributes);
    }

    @Override
    public void setSchema(String schema) throws SQLException {
        con.setSchema(schema);
    }

    @Override
    public String getSchema() throws SQLException {
        return con.getSchema();
    }

    @Override
    public void abort(Executor executor) throws SQLException {
        con.abort(executor);
    }

    @Override
    public void setNetworkTimeout(Executor executor, int milliseconds) throws SQLException {
        con.setNetworkTimeout(executor,milliseconds);
    }

    @Override
    public int getNetworkTimeout() throws SQLException {
        return con.getNetworkTimeout();
    }

    @Override
    public <T> T unwrap(Class<T> iface) throws SQLException {
        return con.unwrap(iface);
    }

    @Override
    public boolean isWrapperFor(Class<?> iface) throws SQLException {
        return con.isWrapperFor(iface);
    }
}

```



### 5. 归还连接-适配器设计模式

- 适配器设计模式归还数据库连接的思想。

  > 1) 我们可以提供一个适配器类，实现Connection 接口，将所有方法进行实现(除了close方法) 
  >
  > 2) 自定义连接类只需要继承这个适配器类，重写需要改进的close() 方法即可

- 适配器设计模式归还数据库连接的实现步骤。

  > ① 定义一个适配器类，实现Connection 接口。 
  >
  > ② 定义 Connection 连接对象的成员变量。 
  >
  > ③ 通过有参构造方法完成对成员变量的赋值。 
  >
  > ④ 重写所有方法(除了close )，调用mysql驱动包的连接对象完成即可。 ⑤ 定义一个连接类，继承适配器类。 
  >
  > ⑥ 定义 Connection 连接对象和连接池容器对象的成员变量，并通过有参构造进行赋值。 
  >
  > ⑦ 重写 close() 方法，完成归还连接。 
  >
  > ⑧ 在自定义连接池中，将获取的连接对象通过自定义连接对象进行包装。
  >
  > 

- 适配器设计模式归还数据库连接存在的问题。 

  > 自定义连接类虽然很简洁了，但适配器类还是我们自己编写的，也比较的麻烦



##### 01示例代码

- 这种思想跟我们学过的servlet技术中，Servlet 接口与HttpServlet处理方式是一样的，我们定义自己的servlet去继承HttpServlet重写doXX方法。

**MyAdapter.java**

```java
package com.itheima02;

import java.sql.*;
import java.util.Map;
import java.util.Properties;
import java.util.concurrent.Executor;
/*
    1.定义一个适配器类。实现Connection接口
    2.定义连接对象的成员变量
    3.通过有参构造为变量赋值
    4.重写所有的抽象方法(除了close)
 */
public abstract class MyAdapter implements Connection {

    //2.定义连接对象的成员变量
    private Connection con;

    //3.通过有参构造为变量赋值
    public MyAdapter(Connection con) {
        this.con = con;
    }


    //4.重写所有的抽象方法(除了close)
    @Override
    public Statement createStatement() throws SQLException {
        return con.createStatement();
    }

    @Override
    public PreparedStatement prepareStatement(String sql) throws SQLException {
        return con.prepareStatement(sql);
    }

    @Override
    public CallableStatement prepareCall(String sql) throws SQLException {
        return con.prepareCall(sql);
    }

    @Override
    public String nativeSQL(String sql) throws SQLException {
        return con.nativeSQL(sql);
    }

    @Override
    public void setAutoCommit(boolean autoCommit) throws SQLException {
        con.setAutoCommit(autoCommit);
    }

    @Override
    public boolean getAutoCommit() throws SQLException {
        return con.getAutoCommit();
    }

    @Override
    public void commit() throws SQLException {
        con.commit();
    }

    @Override
    public void rollback() throws SQLException {
        con.rollback();
    }

    @Override
    public boolean isClosed() throws SQLException {
        return con.isClosed();
    }

    @Override
    public DatabaseMetaData getMetaData() throws SQLException {
        return con.getMetaData();
    }

    @Override
    public void setReadOnly(boolean readOnly) throws SQLException {
        con.setReadOnly(readOnly);
    }

    @Override
    public boolean isReadOnly() throws SQLException {
        return con.isReadOnly();
    }

    @Override
    public void setCatalog(String catalog) throws SQLException{
        con.setCatalog(catalog);
    }

    @Override
    public String getCatalog() throws SQLException {
        return con.getCatalog();
    }

    @Override
    public void setTransactionIsolation(int level) throws SQLException {
        con.setTransactionIsolation(level);
    }

    @Override
    public int getTransactionIsolation() throws SQLException {
        return con.getTransactionIsolation();
    }

    @Override
    public SQLWarning getWarnings() throws SQLException {
        return con.getWarnings();
    }

    @Override
    public void clearWarnings() throws SQLException {
        con.clearWarnings();
    }

    @Override
    public Statement createStatement(int resultSetType, int resultSetConcurrency) throws SQLException {
        return con.createStatement(resultSetType,resultSetConcurrency);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency) throws SQLException {
        return con.prepareStatement(sql,resultSetType,resultSetConcurrency);
    }

    @Override
    public CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency) throws SQLException {
        return con.prepareCall(sql,resultSetType,resultSetConcurrency);
    }

    @Override
    public Map<String, Class<?>> getTypeMap() throws SQLException {
        return con.getTypeMap();
    }

    @Override
    public void setTypeMap(Map<String, Class<?>> map) throws SQLException {
        con.setTypeMap(map);
    }

    @Override
    public void setHoldability(int holdability) throws SQLException {
        con.setHoldability(holdability);
    }

    @Override
    public int getHoldability() throws SQLException {
        return con.getHoldability();
    }

    @Override
    public Savepoint setSavepoint() throws SQLException {
        return con.setSavepoint();
    }

    @Override
    public Savepoint setSavepoint(String name) throws SQLException {
        return con.setSavepoint(name);
    }

    @Override
    public void rollback(Savepoint savepoint) throws SQLException {
        con.rollback(savepoint);
    }

    @Override
    public void releaseSavepoint(Savepoint savepoint) throws SQLException {
        con.releaseSavepoint(savepoint);
    }

    @Override
    public Statement createStatement(int resultSetType, int resultSetConcurrency, int resultSetHoldability) throws SQLException {
        return con.createStatement(resultSetType,resultSetConcurrency,resultSetHoldability);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability) throws SQLException {
        return con.prepareStatement(sql,resultSetType,resultSetConcurrency,resultSetHoldability);
    }

    @Override
    public CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability) throws SQLException {
        return con.prepareCall(sql,resultSetType,resultSetConcurrency,resultSetHoldability);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int autoGeneratedKeys) throws SQLException {
        return con.prepareStatement(sql,autoGeneratedKeys);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int[] columnIndexes) throws SQLException {
        return con.prepareStatement(sql,columnIndexes);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, String[] columnNames) throws SQLException {
        return con.prepareStatement(sql,columnNames);
    }

    @Override
    public Clob createClob() throws SQLException {
        return con.createClob();
    }

    @Override
    public Blob createBlob() throws SQLException {
        return con.createBlob();
    }

    @Override
    public NClob createNClob() throws SQLException {
        return con.createNClob();
    }

    @Override
    public SQLXML createSQLXML() throws SQLException {
        return con.createSQLXML();
    }

    @Override
    public boolean isValid(int timeout) throws SQLException {
        return con.isValid(timeout);
    }

    @Override
    public void setClientInfo(String name, String value) throws SQLClientInfoException {
        con.setClientInfo(name,value);
    }

    @Override
    public void setClientInfo(Properties properties) throws SQLClientInfoException {
        con.setClientInfo(properties);
    }

    @Override
    public String getClientInfo(String name) throws SQLException {
        return con.getClientInfo(name);
    }

    @Override
    public Properties getClientInfo() throws SQLException {
        return con.getClientInfo();
    }

    @Override
    public Array createArrayOf(String typeName, Object[] elements) throws SQLException {
        return con.createArrayOf(typeName,elements);
    }

    @Override
    public Struct createStruct(String typeName, Object[] attributes) throws SQLException {
        return con.createStruct(typeName,attributes);
    }

    @Override
    public void setSchema(String schema) throws SQLException {
        con.setSchema(schema);
    }

    @Override
    public String getSchema() throws SQLException {
        return con.getSchema();
    }

    @Override
    public void abort(Executor executor) throws SQLException {
        con.abort(executor);
    }

    @Override
    public void setNetworkTimeout(Executor executor, int milliseconds) throws SQLException {
        con.setNetworkTimeout(executor,milliseconds);
    }

    @Override
    public int getNetworkTimeout() throws SQLException {
        return con.getNetworkTimeout();
    }

    @Override
    public <T> T unwrap(Class<T> iface) throws SQLException {
        return con.unwrap(iface);
    }

    @Override
    public boolean isWrapperFor(Class<?> iface) throws SQLException {
        return con.isWrapperFor(iface);
    }
}

```

**MyConnection3.java**

```java
package com.itheima02;

import java.sql.Connection;
import java.util.List;

/*
    1.定义一个类，继承适配器类
    2.定义连接对象和连接池容器对象的成员变量
    3.通过有参构造为变量赋值
    4.重写close方法，完成归还连接
 */
//1.定义一个类，继承适配器类
public class MyConnection3 extends MyAdapter {

    //2.定义连接对象和连接池容器对象的成员变量
    private Connection con;
    private List<Connection> pool;

    //3.通过有参构造为变量赋值
    public MyConnection3(Connection con,List<Connection> pool){
        super(con);
        this.con = con;
        this.pool = pool;
    }

    //4.重写close方法，完成归还连接
    @Override
    public void close() {
        pool.add(con);
    }
}

```



### 6. 归还连接-动态代理模式

##### 01动态代理

- 在不改变目标对象方法的情况下对方法进行增强 

> 1) 组成 被代理对象：真实的对象 代理对象：内存中的一个对象 
>
> 2) 要求 代理对象必须和被代理对象实现相同的接口
>
> 3) 实现 Proxy.newProxyInstance()



##### 02动态代理实例代码

Student.java

```java
public class Student implements StudentInterface{
    public void eat(String name) {
        System.out.println("学生吃" + name);
    }

    public void study() {
        System.out.println("在家自学");
    }
}

```

StudentInterface.java

```java
package com.proxy;

public interface StudentInterface {
    void eat(String name);
    void study();
}

```

Test.java

```java
package com.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class Test {


    public static void main(String[] args) {
        Student stu = new Student();
        /*stu.eat("米饭");
        stu.study();*/

        /*
            要求：在不改动Student类中任何的代码的前提下，通过study方法输出一句话：来黑马学习
            类加载器：和被代理对象使用相同的类加载器
            接口类型Class数组：和被代理对象使用相同接口
            代理规则：完成代理增强的功能
         */
        StudentInterface proxyStu = (StudentInterface) Proxy.newProxyInstance(stu.getClass().getClassLoader(), new Class[]{StudentInterface.class}, new InvocationHandler() {
            /*
                执行Student类中所有的方法都会经过invoke方法
                对method方法进行判断
                    如果是study，则对其增强
                    如果不是，还调用学生对象原有的功能即可
             */
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                if(method.getName().equals("study")) {
                    System.out.println("来黑马学习");
                    return null;
                }else {
                    return method.invoke(stu,args);
                }
            }
        });

        proxyStu.eat("米饭");
        proxyStu.study();

    }
}

```



##### 03动态代理方式归还数据库连接

- 动态代理方式归还数据库连接的思想。 

  > 1) 我们可以通过 Proxy 来完成对 Connection 实现类对象的代理
  >
  > 2) 代理过程中判断如果执行的是close 方法，就将连接归还池中。如果是其他方法则调用连接对象原来 的功能即可 

- 动态代理方式归还数据库连接的实现步骤。

  > ① 定义一个类，实现DataSource 接口 
  >
  > ② 定义一个容器，用于保存多个Connection连接对象 
  >
  > ③ 定义静态代码块，通过JDBC 工具类获取 10 个连接保存到容器中 
  >
  > ④ 重写 getConnection 方法，从容器中获取一个连接 
  >
  > ⑤ 通过 Proxy 代理，如果是close 方法，就将连接归还池中。如果是其他方法则调用原有功能 
  >
  > ⑥ 定义 getSize 方法，用于获取容器的大小并返回

- 动态代理方式归还数据库连接存在的问题。 

  > 我们自己写的连接池技术不够完善，功能也不够强大

##### 04动态代理方式归还连接-示例代码

MyDataSource.java

```java
package com.itheima01;

import com.itheima.utils.JDBCUtils;

import javax.sql.DataSource;
import java.io.PrintWriter;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.SQLFeatureNotSupportedException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.logging.Logger;

public class MyDataSource implements DataSource {
    //1.准备一个容器。用于保存多个数据库连接对象
    private static List<Connection> pool = Collections.synchronizedList(new ArrayList<>());

    //2.定义静态代码块,获取多个连接对象保存到容器中
    static{
        for(int i = 1; i <= 10; i++) {
            Connection con = JDBCUtils.getConnection();
            pool.add(con);
        }
    }

    //4.提供一个获取连接池大小的方法
    public int getSize() {
        return pool.size();
    }

    /*
        动态代理方式
     */
    @Override
    public Connection getConnection() throws SQLException {
        if(pool.size() > 0) {
            Connection con = pool.remove(0);

            Connection proxyCon = (Connection) Proxy.newProxyInstance(con.getClass().getClassLoader(), new Class[]{Connection.class}, new InvocationHandler() {
                /*
                    执行Connection实现类连接对象所有的方法都会经过invoke
                    如果是close方法，归还连接
                    如果不是，直接执行连接对象原有的功能即可
                 */
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    if(method.getName().equals("close")) {
                        //归还连接
                        pool.add(con);
                        return null;
                    }else {
                        return method.invoke(con,args);
                    }
                }
            });

            return proxyCon;
        }else {
            throw new RuntimeException("连接数量已用尽");
        }
    }

    //3.重写getConnection方法，用于返回一个连接对象
    /*@Override
    public Connection getConnection() throws SQLException {
        if(pool.size() > 0) {
            Connection con = pool.remove(0);
            //通过自定义的连接对象 对原有的连接对象进行包装
            //MyConnection2 myCon = new MyConnection2(con,pool);
            MyConnection3 myCon = new MyConnection3(con,pool);
            return myCon;
        }else {
            throw new RuntimeException("连接数量已用尽");
        }
    }*/

    @Override
    public Connection getConnection(String username, String password) throws SQLException {
        return null;
    }

    @Override
    public <T> T unwrap(Class<T> iface) throws SQLException {
        return null;
    }

    @Override
    public boolean isWrapperFor(Class<?> iface) throws SQLException {
        return false;
    }

    @Override
    public PrintWriter getLogWriter() throws SQLException {
        return null;
    }

    @Override
    public void setLogWriter(PrintWriter out) throws SQLException {

    }

    @Override
    public void setLoginTimeout(int seconds) throws SQLException {

    }

    @Override
    public int getLoginTimeout() throws SQLException {
        return 0;
    }

    @Override
    public Logger getParentLogger() throws SQLFeatureNotSupportedException {
        return null;
    }
}

```



### 7. 开源数据库连接池

- c3p0  和 druid 都是不错的数据库连接池开源框架

  

#####  01C3P0 数据库连接池

   ① 导入 jar 包。 

   ② 导入配置文件到src 目录下。 

   ③ 创建 C3P0 连接池对象。 

   ④ 获取数据库连接进行使用。

<font color=red>   注意：C3P0 的配置文件会自动加载，但是必须叫c3p0-config.xml 或 c3p0-config.properties 。</font>

​    ① 导入 jar 包。 

​    ② 编写配置文件，放在src 目录下。 

​    ③ 通过 Properties 集合加载配置文件。 

​    ④ 通过 Druid 连接池工厂类获取数据库连接池对象。 

​    ⑤ 获取数据库连接进行使用。

<font color=red>注意：Druid 不会自动加载配置文件，需要我们手动加载，但是文件的名称可以自定义。</font>

**c3p0-config.xml**

```xml
<c3p0-config>
  <!-- 使用默认的配置读取连接池对象 -->
  <default-config>
  	<!--  连接参数 -->
    <property name="driverClass">com.mysql.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://192.168.59.129:3306/db14</property>
    <property name="user">root</property>
    <property name="password">itheima</property>
    
    <!-- 连接池参数 -->
    <!--初始化的连接数量-->
    <property name="initialPoolSize">5</property>
    <!--最大连接数量-->
    <property name="maxPoolSize">10</property>
    <!--超时时间-->
    <property name="checkoutTimeout">3000</property>
  </default-config>

  <named-config name="otherc3p0"> 
    <!--  连接参数 -->
    <property name="driverClass">com.mysql.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost:3306/db15</property>
    <property name="user">root</property>
    <property name="password">itheima</property>
    
    <!-- 连接池参数 -->
    <property name="initialPoolSize">5</property>
    <property name="maxPoolSize">8</property>
    <property name="checkoutTimeout">1000</property>
  </named-config>
</c3p0-config>
```

**c3p0-config.properties**

```properties
c3p0.JDBC.url=jdbc:mysql://localhost:3306/ms_cms?characterEncoding=utf8
c3p0.DriverClass=com.mysql.jdbc.Driver
c3p0.user=root
c3p0.password=
c3p0.initialPoolSize=10
c3p0.maxPoolSize=20
c3p0.minPoolSize=5
c3p0.checkoutTimeout=3000
```

**C3P0Test1.java**

```java
import com.mchange.v2.c3p0.ComboPooledDataSource;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class C3P0Test1 {
    public static void main(String[] args) throws Exception{
        //1.创建c3p0的数据库连接池对象
        DataSource dataSource = new ComboPooledDataSource();

        //2.通过连接池对象获取数据库连接
        Connection con = dataSource.getConnection();

        //3.执行操作
        String sql = "SELECT * FROM student";
        PreparedStatement pst = con.prepareStatement(sql);

        //4.执行sql语句，接收结果集
        ResultSet rs = pst.executeQuery();

        //5.处理结果集
        while(rs.next()) {
            System.out.println(rs.getInt("sid") + "\t" + rs.getString("name") + "\t" + rs.getInt("age") + "\t" + rs.getDate("birthday"));
        }

        //6.释放资源
        rs.close();
        pst.close();
        con.close();
    }
}
```

**C3P0Test2.java**

```java
package com.itheima03;

import com.mchange.v2.c3p0.ComboPooledDataSource;

import javax.sql.DataSource;
import java.sql.Connection;

public class C3P0Test2 {
    public static void main(String[] args) throws Exception{
        //1.创建c3p0的数据库连接池对象
        DataSource dataSource = new ComboPooledDataSource();

        //2.测试
        for(int i = 1; i <= 11; i++) {
            Connection con = dataSource.getConnection();
            System.out.println(i + ":" + con);

            if(i == 5) {
                con.close();
            }
        }
    }
}

```



#####  02阿里Druid 数据库连接池

​    ① 导入 jar 包。 

​    ② 编写配置文件，放在src 目录下。 

​    ③ 通过 Properties 集合加载配置文件。 

​    ④ 通过 Druid 连接池工厂类获取数据库连接池对象。 

​    ⑤ 获取数据库连接进行使用。

<font color=red>     注意：Druid 不会自动加载配置文件，需要我们手动加载，但是文件的名称可以自定义,  propertis 文件中key命名都是固定的</font>

**druid.properties**

```properties
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://192.168.59.129:3306/db14
username=root
password=itheima
# 初始化连接数量
initialSize=5
# 最大连接数量
maxActive=10
# 超时时间
maxWait=3000
```

**DruidTest1.java**

```java
package com.itheima04;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.Properties;

/*
    1.通过Properties集合，加载配置文件
    2.通过Druid连接池工厂类获取数据库连接池对象
    3.通过连接池对象获取数据库连接进行使用
 */
public class DruidTest1 {
    public static void main(String[] args) throws Exception{
        //获取配置文件的流对象
        InputStream is = DruidTest1.class.getClassLoader().getResourceAsStream("druid.properties");

        //1.通过Properties集合，加载配置文件
        Properties prop = new Properties();
        prop.load(is);

        //2.通过Druid连接池工厂类获取数据库连接池对象
        DataSource dataSource = DruidDataSourceFactory.createDataSource(prop);

        //3.通过连接池对象获取数据库连接进行使用
        Connection con = dataSource.getConnection();


        String sql = "SELECT * FROM student";
        PreparedStatement pst = con.prepareStatement(sql);

        //4.执行sql语句，接收结果集
        ResultSet rs = pst.executeQuery();

        //5.处理结果集
        while(rs.next()) {
            System.out.println(rs.getInt("sid") + "\t" + rs.getString("name") + "\t" + rs.getInt("age") + "\t" + rs.getDate("birthday"));
        }

        //6.释放资源
        rs.close();
        pst.close();
        con.close();
    }
}

```

**DruidTest2.java**

```java
package com.itheima04;

import com.itheima.utils.DataSourceUtils;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class DruidTest2 {
    public static void main(String[] args) throws Exception{
        //1.通过连接池工具类获取一个数据库连接
        Connection con = DataSourceUtils.getConnection();

        String sql = "SELECT * FROM student";
        PreparedStatement pst = con.prepareStatement(sql);

        //2.执行sql语句，接收结果集
        ResultSet rs = pst.executeQuery();

        //3.处理结果集
        while(rs.next()) {
            System.out.println(rs.getInt("sid") + "\t" + rs.getString("name") + "\t" + rs.getInt("age") + "\t" + rs.getDate("birthday"));
        }

        //4.释放资源
        DataSourceUtils.close(con,pst,rs);
    }
}

```



##### 03基于Druid连接池-工具类

**DataSourceUtils.java**

```java
package com.itheima.utils;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

/*
    数据库连接池的工具类
 */
public class DataSourceUtils {
    //1.私有构造方法
    private DataSourceUtils(){}

    //2.声明数据源变量
    private static DataSource dataSource;

    //3.提供静态代码块，完成配置文件的加载和获取数据库连接池对象
    static{
        try{
            //完成配置文件的加载
            InputStream is = DataSourceUtils.class.getClassLoader().getResourceAsStream("druid.properties");

            Properties prop = new Properties();
            prop.load(is);

            //获取数据库连接池对象
            dataSource = DruidDataSourceFactory.createDataSource(prop);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //4.提供一个获取数据库连接的方法
    public static Connection getConnection() {
        Connection con = null;
        try {
            con = dataSource.getConnection();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return con;
    }

    //5.提供一个获取数据库连接池对象的方法
    public static DataSource getDataSource() {
        return dataSource;
    }

    //6.释放资源
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

##### 03黑马zou仿写Druid数据库连接池(简陋版)

​    ① 导入 jar 包。 

​    ② 编写配置文件，放在src 目录下。 

​    ③ 通过 Properties 集合加载配置文件。 

​    ④ 通过 Itheima连接池管理类类获取数据库连接池对象。 

​    ⑤ 获取数据库连接进行使用。

<font color=red>     注意：Itheima不会自动加载配置文件，需要我们手动加载，但是文件的名称可以自定义,  propertis 文件中key命名都是固定的</font>

**itheima-zou.properties**

```properties
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://192.168.23.129:3306/db4
username=root
password=root
# 初始化连接数量
initialSize=5
# 最大连接数量
maxActive=10
# 超时时间
maxWait=3000
```

**ItheimaZouTest1.java**

```java
package com.ithiemazou;

import com.itheima.zou.pool.ItheimaDataSourceManager;

import javax.sql.DataSource;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.Properties;

/**
 *     1.通过Properties集合，加载配置文件
 *     2.通过Ithiema连接池管理者类获取数据库连接池对象
 *     3.通过连接池对象获取数据库连接进行使用
 */
public class ItheimaZouTest1 {


    public static void main(String[] args) {
        /**
         * 测试自定义连接池
         */
        try{
            //通过类加载器返回配置文件的字节流
            InputStream is = ItheimaZouTest1.class.getClassLoader().getResourceAsStream("itheima-zou.properties");

            //创建Properties集合，加载流对象的信息
            Properties prop = new Properties();
            prop.load(is);

            //1.创建连接池对象
            DataSource dataSource = ItheimaDataSourceManager.createDataSource(prop);

            //2.通过连接池对象获取连接对象
            Connection con = dataSource.getConnection();
            System.out.println(con.getClass());

            //3.查询学生表的全部信息
            String sql = "SELECT * FROM student";
            PreparedStatement pst = con.prepareStatement(sql);

            //4.执行sql语句，接收结果集
            ResultSet rs = pst.executeQuery();

            //5.处理结果集
            while(rs.next()) {
                System.out.println(rs.getInt("sid") + "\t" + rs.getString("name") + "\t" + rs.getInt("age") + "\t" + rs.getDate("birthday"));
            }

            //6.释放资源
            rs.close();
            pst.close();
            con.close();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }


}

```

**ItheimaZouTest2.java**

```java
package com.ithiemazou;

import com.itheima.zou.pool.ItheimaDataSourceManager;

import javax.sql.DataSource;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

/**
 * 测试当线程大于5个时, 第6个要等待
 */
public class ItheimaZouTest2 {


    public static void main(String[] args) {

        try{
            //通过类加载器返回配置文件的字节流
            InputStream is = ItheimaZouTest2.class.getClassLoader().getResourceAsStream("itheima-zou.properties");

            //创建Properties集合，加载流对象的信息
            Properties prop = new Properties();
            prop.load(is);

            //1.创建连接池对象
            final DataSource dataSource = ItheimaDataSourceManager.createDataSource(prop);

            for (int i = 0; i <6 ; i++) {

                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        //2.通过连接池对象获取连接对象
                        Connection con = null;
                        try {
                            con = dataSource.getConnection();
//                            con.close();
                            System.out.println(con);
                        } catch (SQLException e) {
                            e.printStackTrace();
                        }

                    }
                }).start();
            }

            //阻塞当前线程
            synchronized (ItheimaZouTest2.class){
                ItheimaZouTest2.class.wait();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


}

```



### 8. 自定义JDBC框架-模板（templete）

- 分析案例中的重复代码。

  > 定义必要的信息、获取数据库的连接、释放资源都是重复的代码，而我们最终的核心功能仅仅只是执行一条 sql 语句，所以我们可以抽取出一个JDBC 模板类，来封装一些方法(update、query)，专门帮我们执行增删 改查的 sql 语句。<font color=red>将之前那些重复的操作，都抽取到模板类中的方法里，就能大大简化我们的使用步骤！</font>

##### 01源信息

**1） DataBaseMetaData ：数据库的源信息（了解）** 

**java.sql.DataBaseMetaData ： 封装了整个数据库的综合信息**。 

例如： 

​      String getDatabaseProductName()：获取数据库产品的名称 

​      int getDatabaseProductVersion()：获取数据库产品的版本号

**2） ParameterMetaData：参数的源信息** 

​      java.sql.ParameterMetaData：封装的是预编译执行者对象中每个参数的类型和属性，这个对象可以通过预 编译执行者对象中的getParameterMetaData() 方法来获取 

**核心功能：** int getParameterCount()用于获取 sql 语句中参数的个数



**3）ResultSetMetaData：结果集的源信息**

​     java.sql.ResultSetMetaData：封装的是结果集对象中列的类型和属性，这个对象可以通过结果集对象中的 getMetaData()方法来获取

**核心功能：**int getColumnCount() 用于获取列的总数

​                   String getColumnName(int i) 用于获取列名



##### 02框架的编写

###### 1) 用于执行增删改功能的update() 方法 

> ① 定义所需成员变量(数据源、数据库连接、执行者、结果集)。 
>
> ② 定义有参构造，为数据源对象赋值。 
>
> ③ 定义 update() 方法，参数：sql 语句、sql 语句所需参数。 
>
> ④ 定义 int 类型变量，用于接收sql 语句执行后影响的行数。 
>
> ⑤ 通过数据源获取一个数据库连接。 
>
> ⑥ 通过数据库连接对象获取执行者对象并对sql 语句预编译。 
>
> ⑦ 通过执行者对象获取sql 语句中参数的源信息对象。 
>
> ⑧ 通过参数源信息对象获取sql 语句中参数的个数。 
>
> ⑨ 判断参数个数是否一致。 
>
> ⑩ 为 sql 语句中 ? 占位符赋值。 
>
> ⑪ 执行 sql 语句并接收结果。 
>
> ⑫ 释放资源。 
>
> ⑬ 返回结果。

**JDBCTemplate.java**

```java
package com.itheima05;

import com.itheima.utils.DataSourceUtils;
import com.itheima05.handler.ResultSetHandler;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.ParameterMetaData;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.ArrayList;
import java.util.List;

/*
    JDBC框架类
 */
public class JDBCTemplate {

    //1.定义参数变量(数据源、连接对象、执行者对象、结果集对象)
    private DataSource dataSource;
    private Connection con;
    private PreparedStatement pst;
    private ResultSet rs;

    //2.通过有参构造为数据源赋值
    public JDBCTemplate(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    /*
        用于执行增删改功能的方法
     */
    //3.定义update方法。参数：sql语句、sql语句中的参数
    public int update(String sql,Object...objs) {
        //4.定义int类型变量，用于接收增删改后影响的行数
        int result = 0;

        try{
            //5.通过数据源获取一个数据库连接
            con = dataSource.getConnection();

            //6.通过数据库连接对象获取执行者对象，并对sql语句进行预编译
            pst = con.prepareStatement(sql);

            //7.通过执行者对象获取参数的源信息对象
            ParameterMetaData parameterMetaData = pst.getParameterMetaData();
            //8.通过参数源信息对象获取参数的个数
            int count = parameterMetaData.getParameterCount();

            //9.判断参数数量是否一致
            if(count != objs.length) {
                throw new RuntimeException("参数个数不匹配");
            }

            //10.为sql语句占位符赋值
            for(int i = 0; i < objs.length; i++) {
                pst.setObject(i+1,objs[i]);
            }

            //11.执行sql语句并接收结果
            result = pst.executeUpdate();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //12.释放资源
            DataSourceUtils.close(con,pst);
        }

        //13.返回结果
        return result;
    }
}

```

###### **2) 测试update**()方法

① 定义测试类，模拟dao 层。

② 测试执行 insert 语句。 

③ 测试执行 update 语句。 

④ 测试执行 delete 语句。

**JDBCTemplateTest.java**

```java
package com.itheima05;

import com.itheima.utils.DataSourceUtils;
import com.itheima05.domain.Student;
import com.itheima05.handler.BeanHandler;
import com.itheima05.handler.BeanListHandler;
import com.itheima05.handler.ScalarHandler;
import org.junit.Test;

import java.util.List;

/*
    模拟dao层
 */
public class JDBCTemplateTest {

    private JDBCTemplate template = new JDBCTemplate(DataSourceUtils.getDataSource());

    

    @Test
    public void delete() {
        //删除数据的测试
        String sql = "DELETE FROM student WHERE name=?";
        int result = template.update(sql, "周七");
        System.out.println(result);
    }

    @Test
    public void update() {
        //修改数据的测试
        String sql = "UPDATE student SET age=? WHERE name=?";
        Object[] params = {37,"周七"};
        int result = template.update(sql, params);
        System.out.println(result);
    }

    @Test
    public void insert() {
        //新增数据的测试
        String sql = "INSERT INTO student VALUES (?,?,?,?)";
        Object[] params = {5,"周七",27,"1997-07-07"};
        int result = template.update(sql, params);
        if(result != 0) {
            System.out.println("添加成功");
        }else {
            System.out.println("添加失败");
        }
    }
}

```



###### 3) 用于执行查询功能的方法介绍。 

- 查询一条记录并封装对象的方法：queryForObject() 

- 查询多条记录并封装集合的方法：queryForList() 

- 查询聚合函数并返回单条数据的方法：queryForScalar()



###### **4) 处理结果集接口**&实现类

**ResultSetHandler.java**

```java
import java.sql.ResultSet;

/*
    用于处理结果集方式的接口
 */
public interface ResultSetHandler<T> {

    /**
     *
     * @param <T> T 这个方法的返回值的类型可以在你调用方法时候或者实现类中进行设定
     * @param rs
     * @return
     */
    <T> T handler(ResultSet rs);
}


```

###### 5) 单行多列查询结果集实现类BeanHandler.java

```java
package com.itheima05.handler;

import java.beans.PropertyDescriptor;
import java.lang.reflect.Method;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;

/*
    实现类1：用于将查询到的一条记录，封装为Student对象并返回
 */
//1.定义一个类，实现ResultSetHandler接口
public class BeanHandler<T> implements ResultSetHandler<T>{
    //2.定义Class对象类型变量
    private Class<T> beanClass;

    //3.通过有参构造为变量赋值
    public BeanHandler(Class<T> beanClass) {
        this.beanClass = beanClass;
    }

    //4.重写handler方法。用于将一条记录封装到自定义对象中
    @Override
    public T handler(ResultSet rs) {
        //5.声明自定义对象类型
        T bean = null;
        try {
            //6.创建传递参数的对象，为自定义对象赋值
            bean = beanClass.newInstance();

            //7.判断结果集中是否有数据
            if(rs.next()) {
                //8.通过结果集对象获取结果集源信息的对象
                ResultSetMetaData metaData = rs.getMetaData();
                //9.通过结果集源信息对象获取列数
                int count = metaData.getColumnCount();

                //10.通过循环遍历列数
                for(int i = 1; i <= count; i++) {
                    //11.通过结果集源信息对象获取列名
                    String columnName = metaData.getColumnName(i);

                    //12.通过列名获取该列的数据
                    Object value = rs.getObject(columnName);

                    //13.创建属性描述器对象，将获取到的值通过该对象的set方法进行赋值
                    PropertyDescriptor pd = new PropertyDescriptor(columnName.toLowerCase(),beanClass);
                    //获取可以调用set方法的对象
                    Method writeMethod = pd.getWriteMethod();
                    //执行set方法，给成员变量赋值
                    writeMethod.invoke(bean,value);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        //14.返回封装好的对象
        return bean;
    }
}

```



###### 6) 查询一条记录的方法queryForObject()  

① 定义方法 queryForObject()，参数：sql 语句、处理结果集接口、sql 语句中的参数。 

② 声明自定义类型对象。 

③ 通过数据源获取数据库连接对象。 

④ 通过数据库连接对象获取执行者对象并对sql语句进行预编译。 

⑤ 通过执行者对象获取参数源信息的对象。 

⑥ 通过参数源信息对象获取参数的个数。 

⑦ 判断参数数量是否一致。 

⑧ 为 sql 语句中 ? 占位符赋值。 

⑨ 执行 sql 语句并接收结果集。 

⑩ 通过结果集接口对结果进行处理。 

⑪ 释放资源。 

⑫ 返回结果。

**JDBCTemplate.java**

```java
package com.itheima05;

import com.itheima.utils.DataSourceUtils;
import com.itheima05.handler.ResultSetHandler;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.ParameterMetaData;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.ArrayList;
import java.util.List;

/*
    JDBC框架类
 */
public class JDBCTemplate {

    //1.定义参数变量(数据源、连接对象、执行者对象、结果集对象)
    private DataSource dataSource;
    private Connection con;
    private PreparedStatement pst;
    private ResultSet rs;

    //2.通过有参构造为数据源赋值
    public JDBCTemplate(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    /*
        查询方法：用于将一条记录封装成自定义对象并返回
     */
    public <T> T queryForObject(String sql, ResultSetHandler<T> rsh, Object...objs){
        T obj = null;

        try{
            //通过数据源获取一个数据库连接
            con = dataSource.getConnection();

            //通过数据库连接对象获取执行者对象，并对sql语句进行预编译
            pst = con.prepareStatement(sql);

            //通过执行者对象获取参数的源信息对象
            ParameterMetaData parameterMetaData = pst.getParameterMetaData();
            //通过参数源信息对象获取参数的个数
            int count = parameterMetaData.getParameterCount();

            //判断参数数量是否一致
            if(count != objs.length) {
                throw new RuntimeException("参数个数不匹配");
            }

            //为sql语句占位符赋值
            for(int i = 0; i < objs.length; i++) {
                pst.setObject(i+1,objs[i]);
            }

            //执行sql语句并接收结果
            rs = pst.executeQuery();

            //通过BeanHandler方式对结果进行处理
            obj = rsh.handler(rs);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //释放资源
            DataSourceUtils.close(con,pst,rs);
        }

        //返回结果
        return obj;
    }

    
}

```

###### 7) 测试-queryForObject()  

```java
    @Test
    public void queryForObject() {
        //查询一条记录并封装自定义对象的测试
        String sql = "SELECT * FROM student WHERE sid=?";
        Student stu = template.queryForObject(sql,new BeanHandler<>(Student.class), 1);
        System.out.println(stu);
    }
```



###### 8) 多行多列查询结果集实现类BeanListHandler.java

① 定义BeanListHandler<T> 类实现 ResultSetHandler<T> 接口。 

② 定义Class 对象类型的变量。

③ 定义有参构造为变量赋值。 

④ 重写handler 方法，用于将结果集中的所有记录封装到集合中并返回。 

⑤ 创建List 集合对象。 

⑥ 遍历结果集对象。 

⑦ 创建传递参数的对象。 

⑧ 通过结果集对象获取结果集的源信息对象。 

⑨ 通过结果集源信息对象获取列数。 

⑩ 通过循环遍历列数。

⑪ 通过结果集源信息获取列名。 

⑫ 通过列名获取该列的数据。 

⑬ 创建属性描述器对象，将获取到的值通过对象的set() 方法进行赋值。 

⑭ 将封装好的对象添加到集合中。 

⑮ 返回集合对象。

**BeanListHandler.java**

```java
package com.itheima05.handler;

import java.beans.PropertyDescriptor;
import java.lang.reflect.Method;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.util.ArrayList;
import java.util.List;

/*
    实现类2：用于将查询到的多条记录，封装为Student对象并添加到集合返回
 */
//1.定义一个类，实现ResultSetHandler接口
public class BeanListHandler<T> implements ResultSetHandler<T>{
    //2.定义Class对象类型变量
    private Class<T> beanClass;

    //3.通过有参构造为变量赋值
    public BeanListHandler(Class<T> beanClass) {
        this.beanClass = beanClass;
    }

    //4.重写handler方法。用于将多条记录封装到自定义对象中并添加到集合返回
    @Override
    public List<T> handler(ResultSet rs) {
        //5.声明集合对象类型
        List<T> list = new ArrayList<>();
        try {
            //6.判断结果集中是否有数据
            while(rs.next()) {
                //7.创建传递参数的对象，为自定义对象赋值
                T bean = beanClass.newInstance();

                //8.通过结果集对象获取结果集源信息的对象
                ResultSetMetaData metaData = rs.getMetaData();
                //9.通过结果集源信息对象获取列数
                int count = metaData.getColumnCount();

                //10.通过循环遍历列数
                for(int i = 1; i <= count; i++) {
                    //11.通过结果集源信息对象获取列名
                    String columnName = metaData.getColumnName(i);

                    //12.通过列名获取该列的数据
                    Object value = rs.getObject(columnName);

                    //13.创建属性描述器对象，将获取到的值通过该对象的set方法进行赋值
                    PropertyDescriptor pd = new PropertyDescriptor(columnName.toLowerCase(),beanClass);
                    //获取set方法
                    Method writeMethod = pd.getWriteMethod();
                    //执行set方法，给成员变量赋值
                    writeMethod.invoke(bean,value);
                }
                //将对象保存到集合中
                list.add(bean);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        //14.返回封装好的对象
        return list;
    }
}

```



###### 9) 查询多条记录的方法 queryForList()

① 定义方法 queryForList()，参数：sql 语句、处理结果集接口、sql 语句中的参数。 

② 创建集合对象。 

③ 通过数据源获取数据库连接对象。 

④ 通过数据库连接对象获取执行者对象并对sql语句进行预编译。 

⑤ 通过执行者对象获取参数源信息的对象。 

⑥ 通过参数源信息对象获取参数的个数。 

⑦ 判断参数数量是否一致。 

⑧ 为 sql 语句中 ? 占位符赋值。 

⑨ 执行 sql 语句并接收结果集。 

⑩ 通过结果集接口对结果进行处理。 

⑪ 释放资源。 

⑫ 返回结果。

**JDBCTemplate.java**

```java
package com.itheima05;

import com.itheima.utils.DataSourceUtils;
import com.itheima05.handler.ResultSetHandler;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.ParameterMetaData;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.ArrayList;
import java.util.List;

/*
    JDBC框架类
 */
public class JDBCTemplate {

    //1.定义参数变量(数据源、连接对象、执行者对象、结果集对象)
    private DataSource dataSource;
    private Connection con;
    private PreparedStatement pst;
    private ResultSet rs;

    //2.通过有参构造为数据源赋值
    public JDBCTemplate(DataSource dataSource) {
        this.dataSource = dataSource;
    }


    /*
        查询方法：用于将多条记录封装成自定义对象并添加到集合返回
     */
    public <T> List<T> queryForList(String sql, ResultSetHandler<T> rsh, Object...objs){
        List<T> list = new ArrayList<>();

        try{
            //通过数据源获取一个数据库连接
            con = dataSource.getConnection();

            //通过数据库连接对象获取执行者对象，并对sql语句进行预编译
            pst = con.prepareStatement(sql);

            //通过执行者对象获取参数的源信息对象
            ParameterMetaData parameterMetaData = pst.getParameterMetaData();
            //通过参数源信息对象获取参数的个数
            int count = parameterMetaData.getParameterCount();

            //判断参数数量是否一致
            if(count != objs.length) {
                throw new RuntimeException("参数个数不匹配");
            }

            //为sql语句占位符赋值
            for(int i = 0; i < objs.length; i++) {
                pst.setObject(i+1,objs[i]);
            }

            //执行sql语句并接收结果
            rs = pst.executeQuery();

            //通过BeanListHandler方式对结果进行处理
            list = rsh.handler(rs);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //释放资源
            DataSourceUtils.close(con,pst,rs);
        }

        //返回结果
        return list;
    }

  
}

```

###### 9) 测试-queryForList()

```java
    @Test
    public void queryForList() {
        //查询所有学生信息的测试
        String sql = "SELECT * FROM student";
        List<Student> list = template.queryForList(sql, new BeanListHandler<>(Student.class));
        for(Student stu : list) {
            System.out.println(stu);
        }
    }
```



###### 10) 聚合函数实现类ScalarHandler.java 

① 定义 ScalarHandler<T> 类实现 ResultSetHandler<T> 接口。 

② 重写 handler() 方法，用于返回一个聚合函数的查询结果。 

③ 定义 Long 类型变量。 

④ 判断结果集对象是否有数据。 

⑤ 通过结果集对象获取结果集源信息的对象。 

⑥ 通过结果集源信息对象获取第一列的列名。 

⑦ 通过列名获取该列的数据。 

⑧ 将结果返回。

**ScalarHandler.java**

```java
package com.itheima05.handler;

import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.util.List;

/*
    1.定义一个类，实现ResultSetHandler接口
    2.重写handler方法
    3.定义一个Long类型变量
    4.判断结果集对象中是否还有数据
    5.获取结果集源信息的对象
    6.获取第一列的列名
    7.根据列名获取该列的值
    8.返回结果
 */
//1.定义一个类，实现ResultSetHandler接口
public class ScalarHandler<T> implements ResultSetHandler<T> {

    //2.重写handler方法
    @Override
    public List<T> handler(ResultSet rs) {
        //3.定义一个Long类型变量
        Long value = null;
        try{
            //4.判断结果集对象中是否还有数据
            if(rs.next()) {
                //5.获取结果集源信息的对象
                ResultSetMetaData metaData = rs.getMetaData();
                //6.获取第一列的列名
                String columnName = metaData.getColumnName(1);
                //7.根据列名获取该列的值
               value = rs.getLong(columnName);
            }
        }catch (Exception e) {
            e.printStackTrace();
        }
        //8.返回结果
        return value;
    }
}

```



###### 11) 查询聚合函数的方法 queryForScalar()

① 定义方法 queryForScalar()，参数：sql 语句、处理结果集接口、sql 语句中的参数。 

② 创建 Long 类型变量。 

③ 通过数据源获取数据库连接对象。 

④ 通过数据库连接对象获取执行者对象并对sql 语句进行预编译

⑤ 通过执行者对象获取参数源信息的对象。 

⑥ 通过参数源信息对象获取参数的个数。 

⑦ 判断参数数量是否一致。 

⑧ 为 sql 语句中 ? 占位符赋值。 

⑨ 执行 sql 语句并接收结果集。 

⑩ 通过结果集接口对结果进行处理。 

⑪ 释放资源。 

⑫ 返回结果。

**JDBCTemplate.java**

```java
package com.itheima05;

import com.itheima.utils.DataSourceUtils;
import com.itheima05.handler.ResultSetHandler;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.ParameterMetaData;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.ArrayList;
import java.util.List;

/*
    JDBC框架类
 */
public class JDBCTemplate {

    //1.定义参数变量(数据源、连接对象、执行者对象、结果集对象)
    private DataSource dataSource;
    private Connection con;
    private PreparedStatement pst;
    private ResultSet rs;

    //2.通过有参构造为数据源赋值
    public JDBCTemplate(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    /*
        查询方法：用于将聚合函数的查询结果进行返回
     */
    public Long queryForScalar(String sql, ResultSetHandler<Long> rsh, Object...objs){
        Long value = null;

        try{
            //通过数据源获取一个数据库连接
            con = dataSource.getConnection();

            //通过数据库连接对象获取执行者对象，并对sql语句进行预编译
            pst = con.prepareStatement(sql);

            //通过执行者对象获取参数的源信息对象
            ParameterMetaData parameterMetaData = pst.getParameterMetaData();
            //通过参数源信息对象获取参数的个数
            int count = parameterMetaData.getParameterCount();

            //判断参数数量是否一致
            if(count != objs.length) {
                throw new RuntimeException("参数个数不匹配");
            }

            //为sql语句占位符赋值
            for(int i = 0; i < objs.length; i++) {
                pst.setObject(i+1,objs[i]);
            }

            //执行sql语句并接收结果
            rs = pst.executeQuery();

            //通过ScalarHandler方式对结果进行处理
            value = rsh.handler(rs);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //释放资源
            DataSourceUtils.close(con,pst,rs);
        }

        //返回结果
        return value;
    }

    
}

```



###### 12) 测试-queryForScalar()

```java
    @Test
    public void queryForScalar() {
        //查询聚合函数的测试
        String sql = "SELECT COUNT(*) FROM student";
        Long value = template.queryForScalar(sql,new ScalarHandler<Long>());
        System.out.println(value);
    }
```

