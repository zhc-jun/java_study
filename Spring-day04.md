##  事务

### **学习目标**：

   1）理解事务的概念

​    **2）** 基于xml或者注解实现 事物

​    **3）** 能够说出事物的传播行为

​    **4）**练一练 redisTemplete 常用的API

#### 注意事项: 

- <font color=red>spring 是使用 aop 来代理事务控制 ，是针对于接口或类的，所以在同一个 service 实现类中两个方法的调用，传播机制是不生效的</font>

- <font color=red>@Transaction 注解可以用在接口上，也可以用在类上。spring团队推荐我们用在类上。</font>

- <font color=red>只有当service方法 发生 RuntimeException时 才可以触发事物回滚，其他异常默认不回滚，比如：Exception 和 IOException 都不回滚，可以使用 @Transaction 中 rollbackFor = {Exception.class, IOException.class} 来人为加入触发回滚的异常。</font>

  

### 1. 事务回顾  

- 保障数据库 即使在异常状态下仍能保持数据一致性（要么操作前状态，要么操作后状态）
- 当出现并发访问数据库时，在多个访问间进行相互隔离，防止并发访问操作结果互相干扰

#### **事务特征ACID**

 	◆ 原子性（Atomicity）指事务是一个不可分割的整体，其中的操作要么全执行或全不执行 

​	 ◆ 一致性（Consistency）事务前后数据的完整性必须保持一致 

​	 ◆ 隔离性（Isolation）事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的 事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离 

​	 ◆ 持久性（Durability）持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性 的，接下来即使数据库发生故障也不应该对其有任何影响

#### 事务隔离级别

展示隔离级别： select @@session.tx_isolation;

不同隔离级别带来的问题如下: 

|         事务隔离级别         | 脏读 | 不可重复读 | 幻读 |
| :--------------------------: | :--: | :--------: | :--: |
| 读未提交（read-uncommitted） |  是  |     是     |      |
|  读已提交（read-committed）  |  否  |     是     |  是  |
| 可重复读（repeatable-read）  |  否  |     否     |  是  |
|    串行化（serializable）    |  否  |     否     |  否  |

**脏读:**

- 事物A读到了事物B未提交的数据，如果事物B回滚了，对A事物来说查到的数据就是不正确的，这种现象我们称之为：“脏读”。

- 解决：RU隔离级别存在脏读问题，修改隔离级别为: RC 或以上。

- 1、读未提交：

  　　　　（1）打开一个客户端A，并设置当前事务模式为read uncommitted（未提交读），查询表account的初始值：

   ![img](https://images2015.cnblogs.com/blog/1183794/201706/1183794-20170615225939087-367776221.png)

  　　　　（2）在客户端A的事务提交之前，打开另一个客户端B，更新表account：

   ![img](https://images2015.cnblogs.com/blog/1183794/201706/1183794-20170615230218306-862399438.png)

   

  　　　　（3）这时，虽然客户端B的事务还没提交，但是客户端A就可以查询到B已经更新的数据：

   ![img](https://images2015.cnblogs.com/blog/1183794/201706/1183794-20170615230427790-2059251412.png)

  　　　　（4）一旦客户端B的事务因为某种原因回滚，所有的操作都将会被撤销，那客户端A查询到的数据其实就是脏数据：

**不可重复读**:  

- 事物A读到了事物B修改后提交的数据，导致事物A两次执行一模一样的查询SQL得到的数据是不一样的，这种现象称之为： “不可重复读”。

- 解决：RU隔离级别存在不可重复读问题, 修改隔离级别为：RR 或 以上。

- 2、读已提交

  　　　　（1）打开一个客户端A，并设置当前事务模式为read committed（未提交读），查询表account的所有记录：

   ![img](https://images2015.cnblogs.com/blog/1183794/201706/1183794-20170615231437353-1441361659.png)

  　　　　（2）在客户端A的事务提交之前，打开另一个客户端B，更新表account：

   ![img](https://images2015.cnblogs.com/blog/1183794/201706/1183794-20170615231920696-48081094.png)

  　　　　（3）这时，客户端B的事务还没提交，客户端A不能查询到B已经更新的数据，解决了脏读问题：

   ![img](https://images2015.cnblogs.com/blog/1183794/201706/1183794-20170615232203978-179631977.png)

  　　　　（4）客户端B的事务提交

  ![img](https://images2015.cnblogs.com/blog/1183794/201706/1183794-20170615232506650-1677223761.png)

  　　　　（5）客户端A执行与上一步相同的查询，结果 与上一步不一致，即产生了不可重复读的问题

**幻读**

- 事物A查询 用户赵六, 没有查到，想要新增用户赵六时，mysql提示错误：“该用户已存在”，执行删除赵六操作时，mysql提示成功了。导致原因：“事物B在事物A插入之前，提前将赵六插入到了表中，进而导致事物A出现了幻觉”，这种现象称之为：“幻读”。

- 解决：RR这种mysql默认的隔离界别只存在幻读问题，一般我们业务系统可以无视此影响，当然也可以修改隔离级别为：serializable。强烈不推荐修改隔离界别，因为:serializable 存在锁表的操作，会十分的影响性能，导致数据库的并发不高。

- 3、可重复读

  （1）打开一个客户端A，并设置当前事务模式为repeatable read，查询表account的所有记录

  ![img](https://images2015.cnblogs.com/blog/1183794/201706/1183794-20170615233320290-1840487787.png)

  　　　　（2）在客户端A的事务提交之前，打开另一个客户端B，更新表account并提交

  ![img](https://images2015.cnblogs.com/blog/1183794/201706/1183794-20170615233526103-1495989601.png)

  　　　　（3）在客户端A查询表account的所有记录，与步骤（1）查询结果一致，没有出现不可重复读的问题

  ![img](https://images2015.cnblogs.com/blog/1183794/201706/1183794-20170615233858087-1000794949.png)

  　　　　（4）在客户端A，接着执行update balance = balance - 50 where id = 1，balance没有变成400-50=350，lilei的balance值用的是步骤（2）中的350来算的，所以是300，数据的一致性倒是没有被破坏。可重复读的隔离级别下使用了MVCC机制，select操作不会更新版本号，是快照读（历史版本）；insert、update和delete会更新版本号，是当前读（当前版本）。

  ![img](https://images2018.cnblogs.com/blog/1183794/201809/1183794-20180903014747581-1570431335.png)

  （5）重新打开客户端B，插入一条新数据后提交

  ![img](https://images2018.cnblogs.com/blog/1183794/201809/1183794-20180903015202958-625875608.png)

  （6）在客户端A查询表account的所有记录，没有 查出 新增数据，所以没有出现幻读

  ![img](https://images2018.cnblogs.com/blog/1183794/201809/1183794-20180903015501418-1606127377.png)




### 2. 事务管理

#### 2.1 Spring事务核心对象

⚫ 事务开启在业务层或者 数据层并无太大差别，当业务中包含多个数据层的调用时，需要在业务层开启事务，对数据层中多个操 作进行组合并归属于同一个事务进行处理 

⚫ Spring为业务层提供了整套的事务解决方案 

◆ PlatformTransactionManager ◆ TransactionDefinition ◆ TransactionStatus



##### PlatformTransactionManager

⚫ 此接口定义了事务的基本操作

◆ 获取事务 ：

```java
TransactionStatus getTransaction(TransactionDefinition definition)
```

◆ 提交事务 ：

```java
void commit(TransactionStatus status) 
```

◆ 回滚事务 ：

```java
void rollback(TransactionStatus status)
```



##### TransactionDefinition

⚫ 此接口定义了事务的基本信息 

◆ 获取事务定义名称

```java
int getPropagationBehavior()
```

◆ 获取事务的读写属性

```java
int getIsolationLevel()
```

◆ 获取事务隔离级别

```java
int getTimeout()
```

◆ 获取事务超时时间

```java
boolean isReadOnly()
```

◆ 获取事务传播行为特征

```java
String getName()
```



##### TransactionStatus

⚫ 此接口定义了事务在执行过程中某个时间点上的状态信息及对应的状态操作 

◆ 获取事务是否处于新开启事务状态

```java
boolean isNewTransaction()
```

◆ 获取事务是否处于已完成状态

```java
boolean isCompleted()
```

◆ 获取事务是否处于回滚状态

```java
boolean isRollbackOnly()
```

◆ 刷新事务状态

```java
void flush()
```

◆ 获取事务是否具有回滚存储点

```java
boolean hasSavepoint()
```

◆ 设置事务处于回滚状态

```java
void setRollbackOnly()
```



#### 2.2 事务控制方式

- 编程式

- 声明式（XML） 

- 声明式（注解）

  

#### 2.3 编程式事物-案例

⚫ 银行转账业务说明 银行转账操作中，涉及从A账户到B账户的资金转移操作。数据层仅提供单条数据的基础操作，未 设计多账户间的业务操作。

##### TxAdvice.java

```java
package com.itheima.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import javax.sql.DataSource;

public class TxAdvice {

    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public Object transactionManager(ProceedingJoinPoint pjp) throws Throwable {
        //开启事务
        PlatformTransactionManager ptm = new DataSourceTransactionManager(dataSource);
        //事务定义
        TransactionDefinition td = new DefaultTransactionDefinition();
        //事务状态
        TransactionStatus ts = ptm.getTransaction(td);

        Object ret = pjp.proceed(pjp.getArgs());

        //提交事务
        ptm.commit(ts);

        return ret;
    }

}

```

##### applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:property-placeholder location="classpath:*.properties"/>

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <!--<property name="dataSource" ref="dataSource"/>-->
    </bean>

    <bean class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="typeAliasesPackage" value="com.itheima.domain"/>
    </bean>

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.itheima.dao"/>
    </bean>


    <!--###########################################-->

    <bean id="txAdvice" class="com.itheima.aop.TxAdvice">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <aop:config>
        <aop:pointcut id="pt" expression="execution(* *..transfer(..))"/>
        <aop:aspect ref="txAdvice">
            <aop:around method="transactionManager" pointcut-ref="pt"/>
        </aop:aspect>
    </aop:config>


</beans>
```

##### AccountServiceImpl.java

```java 
package com.itheima.service.impl;


import com.itheima.dao.AccountDao;
import com.itheima.service.AccountService;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import javax.sql.DataSource;

public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;
    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    public void transfer(String outName, String inName, Double money) {
        accountDao.inMoney(outName,money);
        int i = 1/0;
        accountDao.outMoney(inName,money);
    }

}
```

##### App.java

```java
import com.itheima.service.AccountService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        AccountService accountService = (AccountService) ctx.getBean("accountService");
        accountService.transfer("Jock1","Jock2",100D);
    }
}
```



#### 2.4 声明式事务

##### 2.4.1 aop:advice与aop:advisor

⚫ aop:advice配置的通知类可以是普通java对象，不实现接口，也不使用继承关系 

⚫ aop:advisor配置的通知类必须实现通知接口 

◆ MethodBeforeAdvice ◆ AfterReturningAdvice ◆ ThrowsAdvice ◆ ……



#####  2.4.2 tx:advice 

- 归属：beans标签  

- 作用：专用于声明事务通知

- 基本属性： 

   ◆ id ：用于配置aop时指定通知器的id 

   ◆ transaction-manager ：指定事务管理器bean

  ```xml
  <!--适用于 spring jdbc 或者 mybatis 事物管理-->
  <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
      <property name="dataSource" ref="dataSource"/>
  </bean>
  
  <!--定义事务管理的通知类-->
  <tx:advice id="txAdvice" transaction-manager="txManager">
  </tx:advice>
  ```

##### 2.4.3 tx:attributes 

- 归属：tx:advice标签   
- 作用：定义通知属性 

```xml
<!-- 配置哪些方法要加入事务控制 -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
    <!-- 让所有的方法都加入事务管理，为了提高效率，可以把一些查询之类的方法设置为只读的事务-->
    <tx:method name="*" propagation="REQUIRED" read-only="true" />
    <!--  以下方法都是可能设计修改的方法，就无法设置为只读 -->
    <tx:method name="add*" propagation="REQUIRED" />
    <tx:method name="del*" propagation="REQUIRED" />
    <tx:method name="update*" propagation="REQUIRED" />
    <tx:method name="save*" propagation="REQUIRED" />
    <tx:method name="clear*" propagation="REQUIRED" />
    </tx:attributes>
</tx:advice>
```

##### 2.4.4 tx:method 

- 归属：tx:attribute标签    
- 作用：设置具体的事务属性 
- 说明：通常事务属性会配置多个，包含1个读写的全事务属性，1个只读的查询类事务属性

- 属性

  1）name="*" ： 待添加事务的方法名表达式（支持*号通配符），例如get* 、* 

  2）read-only="false"  ：设置事务的读写属性，true为只读，false为读写 

  3）timeout="-1" ： 设置事务超时时长，单位秒 

  4）isolation="DEFAULT"  设置事务隔离级别，该隔离级设定是基于Spring的设定，非数据库端 

  5）no-rollback-for=""  设置事务中不回滚的异常，多个异常间使用，分割 

  6）rollback-for=""  设置事务中必回滚的异常，多个异常间使用，分割 

  7）propagation="REQUIRED"  设置事务的传播行为

```xml
<!-- 配置哪些方法要加入事务控制 -->
<tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
    <!-- 让所有的方法都加入事务管理，为了提高效率，可以把一些查询之类的方法设置为只读的事务-->
    <tx:method name="*" propagation="REQUIRED" read-only="true" />
    <!--  以下方法都是可能设计修改的方法，就无法设置为只读 -->
    <tx:method name="add*" propagation="REQUIRED" />
    <tx:method name="del*" propagation="REQUIRED" />
    <tx:method name="update*" propagation="REQUIRED" />
    <tx:method name="save*" propagation="REQUIRED" />
    <tx:method name="clear*" propagation="REQUIRED" />
    </tx:attributes>
</tx:advice>
```

##### 2.4.5 示例代码

###### applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:property-placeholder location="classpath:*.properties"/>

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <bean class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="typeAliasesPackage" value="com.itheima.domain"/>
    </bean>

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.itheima.dao"/>
    </bean>

    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
    </bean>
    
    <!--适用于 spring jdbc 或者 mybatis 事物管理-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置哪些方法要加入事务控制 -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <!-- 让所有的方法都加入事务管理，为了提高效率，可以把一些查询之类的方法设置为只读的事务 -->
            <tx:method name="*" propagation="REQUIRED" read-only="true" />
            <!-- 以下方法都是可能设计修改的方法，就无法设置为只读 -->
            <tx:method name="add*" propagation="REQUIRED" />
            <tx:method name="del*" propagation="REQUIRED" />
            <tx:method name="update*" propagation="REQUIRED" />
            <tx:method name="save*" propagation="REQUIRED" />
            <tx:method name="clear*" propagation="REQUIRED" />
        </tx:attributes>
    </tx:advice>

    <aop:config>
        <!-- 事务不应该在DAO层处理，而应该在service，这也就是Spring所提供的一个非常方便的工具，声明式事务 -->
        <aop:pointcut id="pt" expression="execution(* com.itheima.service.*Service.*(..))"/>
        <aop:pointcut id="pt2" expression="execution(* com.itheima.dao.*.b(..))"/>
        <!--< aop:advisor>大多用于事务管理-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pt"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pt2"/>
    </aop:config>

</beans>
```

###### AccountServiceImpl.java

```java 
package com.itheima.service.impl;


import com.itheima.dao.AccountDao;
import com.itheima.service.AccountService;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import javax.sql.DataSource;

public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;
    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    public void transfer(String outName, String inName, Double money) {
        accountDao.inMoney(outName,money);
        int i = 1/0;
        accountDao.outMoney(inName,money);
    }

}
```

###### App.java

```java
import com.itheima.service.AccountService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        AccountService accountService = (AccountService) ctx.getBean("accountService");
        accountService.transfer("Jock1","Jock2",100D);
    }
}
```



#### 2.5 事物传播行为

**说明**：一般用得比较多的是 `PROPAGATION_REQUIRED `， `REQUIRES_NEW`；

`REQUIRES_NEW` 一般用在子方法需要单独事务, 不受父事物影响自身提交。

##### 2.5.1 生效条件: 

- <font color=red>spring 是使用 aop 来代理事务控制 ，是针对于接口或类的，所以在同一个 service 实现类中两个方法的调用，传播机制是不生效的</font>

##### 2.5.2 REQUIRED (默认)

- 支持当前事务，如果当前没有事务，则新建事务
- 如果当前存在事务，则加入当前事务，合并成一个事务

##### 2.5.3 REQUIRES_NEW

- 新建事务，如果当前存在事务，则把当前事务挂起
- 这个方法会独立提交事务，不受调用者的事务影响，父级异常，它也是正常提交

##### 2.5.4 NESTED

- 如果当前存在事务，它将会成为父级事务的一个子事务，方法结束后并没有提交，只有等父事务结束才提交
- 如果当前没有事务，则新建事务
- 如果它异常，父级可以捕获它的异常而不进行回滚，正常提交
- 但如果父级异常，它必然回滚，这就是和 `REQUIRES_NEW` 的区别

##### 2.5.5 SUPPORTS

- 如果当前存在事务，则加入事务
- 如果当前不存在事务，则以非事务方式运行，这个和不写没区别

##### 2.5.6 NOT_SUPPORTED

- 以非事务方式运行
- 如果当前存在事务，则把当前事务挂起

##### 2.5.7 MANDATORY

- 如果当前存在事务，则运行在当前事务中
- 如果当前无事务，则抛出异常，也即父级方法必须有事务

##### 2.5.8 NEVER

- 以非事务方式运行，如果当前存在事务，则抛出异常，即父级方法必须无事务



#### 2.6 声明式事务（注解）

##### @Transactional  

- 位置：方法定义上方，类定义上方，接口定义上方

- 作用：设置当前类/接口中所有方法或具体方法开启事务，并指定相关事务属性 

- 范例：

  ```java
  @Transactional(
      readOnly = false,
      timeout = -1,
      isolation = Isolation.DEFAULT,
      rollbackFor = {ArithmeticException.class, IOException.class},
      noRollbackFor = {},
      propagation = Propagation.REQUIRES_NEW
  )
  ```



##### tx:annotation-driven 

- 归属：beans标签 

- 作用：开启事务注解驱动，并指定对应的事务管理器 

- 范例：

  ```xml
  <!--   TX格式   -->
  <!--适用于 spring jdbc 或者 mybatis 事物管理-->
  <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
      <property name="dataSource" ref="dataSource"/>
  </bean>
  
  <tx:annotation-driven transaction-manager="txManager"/>
  ```



##### 示例代码

###### applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:property-placeholder location="classpath:*.properties"/>

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <bean class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="typeAliasesPackage" value="com.itheima.domain"/>
    </bean>

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.itheima.dao"/>
    </bean>

    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
    </bean>

    <!--                  TX格式                  -->
    <!--适用于 spring jdbc 或者 mybatis 事物管理-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <tx:annotation-driven transaction-manager="txManager"/>
    
</beans>
```

###### AccountServiceImpl.java

```java
package com.itheima.service.impl;

import com.itheima.dao.AccountDao;
import com.itheima.domain.Account;
import com.itheima.service.AccountService;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import javax.sql.DataSource;
import java.io.IOException;
import java.util.List;

//对当前类的所有方法添加事务，通常配置在接口上

//@Transactional
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;
    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    public void transfer(String outName, String inName, Double money) {
        accountDao.inMoney(outName,money);
        int i = 1/0;
        accountDao.outMoney(inName,money);
    }

}
```

###### App.java

```java
import com.itheima.service.AccountService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        AccountService accountService = (AccountService) ctx.getBean("accountService");
        accountService.transfer("Jock1","Jock2",100D);
    }
}
```



#### 2.7 声明式事务（注解驱动）

- @EnableTransactionManagement
- @Transactional

##### UserServiceTest.java

```java
package com.itheima.service;

import com.itheima.config.SpringConfig;
import com.itheima.domain.Account;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.List;

//设定spring专用的类加载器
@RunWith(SpringJUnit4ClassRunner.class)
//设定加载的spring上下文对应的配置
@ContextConfiguration(classes = SpringConfig.class)
public class UserServiceTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void testTransfer(){
        accountService.transfer("Jock1","Jock2",100D);
    }

}
```

##### SpringConfig.java

```java
package com.itheima.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.PropertySource;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@Configuration
@ComponentScan("com.itheima")
@PropertySource("classpath:jdbc.properties")
@Import({JDBCConfig.class,MyBatisConfig.class})
//开启注解事物支持
@EnableTransactionManagement
public class SpringConfig {
}
```

##### JDBCConfig.java

```java
package com.itheima.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;

public class JDBCConfig {

    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;

    @Bean("dataSource")
    public DataSource getDataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        return ds;
    }

    /**
     * 注入事物管理对象
     * @param dataSource
     * 支持 spirng jdbc 或者 mybatis 事物管理
     * @return
     */
    @Bean
    public PlatformTransactionManager getTransactionManager(DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);
    }

}
```

##### MyBatisConfig.java

```java
package com.itheima.config;

import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.mapper.MapperScannerConfigurer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;

import javax.sql.DataSource;

public class MyBatisConfig {

    @Bean
    public SqlSessionFactoryBean getSqlSessionFactoryBean(@Autowired DataSource dataSource){
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        ssfb.setTypeAliasesPackage("com.itheima.domain");
        ssfb.setDataSource(dataSource);
        return ssfb;
    }

    @Bean
    public MapperScannerConfigurer getMapperScannerConfigurer(){
        MapperScannerConfigurer msc = new MapperScannerConfigurer();
        msc.setBasePackage("com.itheima.dao");
        return msc;
    }

}
```

##### AccountServiceImpl.java

```java
package com.itheima.service.impl;


import com.itheima.dao.AccountDao;
import com.itheima.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;

    @Transactional
    public void transfer(String outName, String inName, Double money) {
        accountDao.inMoney(outName,money);
        int i = 1/0;
        accountDao.outMoney(inName,money);
    }
}
```



### 3. 模板对象 JdbcTemplet

**Spring模板对象**

- TransactionTemplate

- JdbcTemplate

- RedisTemplate

- RabbitTemplate

- JmsTemplate

- HibernateTemplate

- RestTemplate

  

#### AccountServiceTest.java

```java
package com.itheima.service;

import com.itheima.config.SpringConfig;
import com.itheima.domain.Account;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.List;

import static org.junit.Assert.*;
//设定spring专用的类加载器
@RunWith(SpringJUnit4ClassRunner.class)
//设定加载的spring上下文对应的配置
@ContextConfiguration(classes = SpringConfig.class)
public class AccountServiceTest {
    @Autowired
    private AccountService accountService;

    @Test
    public void testSave() {
        Account account = new Account();
        account.setName("阿尔萨斯");
        account.setMoney(999.99d);
        accountService.save(account);
    }

    @Test
    public void testDelete() {
        accountService.delete(6);
    }

    @Test
    public void testUpdate() {
        Account account = new Account();
        account.setId(7);
        account.setName("itheima");
        account.setMoney(6666666666.66d);
        accountService.update(account);
    }

    @Test
    public void testFindNameById() {
        String name = accountService.findNameById(2);
        System.out.println(name);
    }

    @Test
    public void testFindById() {
        Account account = accountService.findById(2);
        System.out.println(account);
    }

    @Test
    public void testFindAll() {
        List<Account> list = accountService.findAll();
        System.out.println(list);
    }

    @Test
    public void testFindAll1() {
        List<Account> list = accountService.findAll(1, 2);
        System.out.println(list);
    }

    @Test
    public void testGetCount() {
        Long count = accountService.getCount();
        System.out.println(count);
    }
}
```

#### SpringConfig.java

```java
package com.itheima.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.PropertySource;

@Configuration
@ComponentScan("com.itheima")
@PropertySource("classpath:jdbc.properties")
@Import(JDBCConfig.class)
public class SpringConfig {
}

```

#### JDBCConfig.java

```java
package com.itheima.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;

import javax.sql.DataSource;

public class JDBCConfig {
    
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;

    @Bean("dataSource")
    public DataSource getDataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        return ds;
    }

    //注册JdbcTemplate模块对象bean
    @Bean("jdbcTemplate")
    public JdbcTemplate getJdbcTemplate(@Autowired DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }

    @Bean("jdbcTemplate2")
    public NamedParameterJdbcTemplate getJdbcTemplate2(@Autowired DataSource dataSource){
        return new NamedParameterJdbcTemplate(dataSource);
    }

}
```

#### AccountServiceImpl.java

```java
package com.itheima.service.impl;


import com.itheima.dao.AccountDao;
import com.itheima.domain.Account;
import com.itheima.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;

    public void save(Account account) {
        accountDao.save(account);
    }

    public void delete(Integer id) {
        accountDao.delete(id);
    }

    public void update(Account account){
        accountDao.update(account);
    }

    public String findNameById(Integer id) {
        return accountDao.findNameById(id);
    }

    public Account findById(Integer id) {
        return accountDao.findById(id);
    }

    public List<Account> findAll() {
        return accountDao.findAll();
    }

    public List<Account> findAll(int pageNum, int preNum) {
        return accountDao.findAll(pageNum,preNum);
    }

    public Long getCount() {
        return accountDao.getCount();
    }
}
```

#### AccountDaoImpl.java

```java
package com.itheima.dao.impl;

import com.itheima.dao.AccountDao;
import com.itheima.domain.Account;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Repository;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

//dao注册为bean
@Repository("accountDao")
public class AccountDaoImpl implements AccountDao {

    //注入模板对象
    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void save(Account account) {
        String sql = "insert into account(name,money)values(?,?)";
        jdbcTemplate.update(sql,account.getName(),account.getMoney());
    }

    public void delete(Integer id) {
        String sql = "delete from account where id = ?";
        jdbcTemplate.update(sql,id);
    }

    public void update(Account account) {
        String sql = "update account set name = ? , money = ? where id = ?";
        jdbcTemplate.update(sql, account.getName(),account.getMoney(),account.getId());
    }

    public String findNameById(Integer id) {
        String sql = "select name from account where id = ? ";
        //单字段查询可以使用专用的查询方法，必须制定查询出的数据类型，例如name为String类型
        return jdbcTemplate.queryForObject(sql,String.class,id );
    }

    public Account findById(Integer id) {
        String sql = "select * from account where id = ? ";
        //支持自定义行映射解析器
        RowMapper<Account> rm = new RowMapper<Account>() {
            public Account mapRow(ResultSet rs, int rowNum) throws SQLException {
                Account account = new Account();
                account.setId(rs.getInt("id"));
                account.setName(rs.getString("name"));
                account.setMoney(rs.getDouble("money"));
                return account;
            }
        };
        return jdbcTemplate.queryForObject(sql,rm,id);
    }

    public List<Account> findAll() {
        String sql = "select * from account";
        //使用spring自带的行映射解析器，要求必须是标准封装
        return jdbcTemplate.query(sql,new BeanPropertyRowMapper<Account>(Account.class));
    }

    public List<Account> findAll(int pageNum, int preNum) {
        String sql = "select * from account limit ?,?";
        //分页数据通过查询参数赋值
        return jdbcTemplate.query(sql,new BeanPropertyRowMapper<Account>(Account.class),(pageNum-1)*preNum,preNum);
    }

    public Long getCount() {
        String sql = "select count(id) from account ";
        //单字段查询可以使用专用的查询方法，必须制定查询出的数据类型，例如数据总量为Long类型
        return jdbcTemplate.queryForObject(sql,Long.class);
    }
}
```

#### jdbc.properties

```java
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring_db
jdbc.username=root
jdbc.password=itheima
```

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>Spring_day04_05_JDBCTemplate</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!--无mybatis配置坐标-->
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
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```



### 4.  模板对象  RedisTemplate

#### AccountServiceTest.java

```java
package com.itheima.service;

import com.itheima.config.SpringConfig;
import com.itheima.domain.Account;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import redis.clients.jedis.Jedis;

import java.util.List;

import static org.junit.Assert.*;
//设定spring专用的类加载器
@RunWith(SpringJUnit4ClassRunner.class)
//设定加载的spring上下文对应的配置
@ContextConfiguration(classes = SpringConfig.class)
public class AccountServiceTest {
    @Autowired
    private AccountService accountService;

    @Test
    public void test(){
        Jedis jedis = new Jedis("192.168.40.130",6378);
        jedis.set("name","itheima");
        jedis.close();
    }

    @Test
    public void save(){
        Account account = new Account();
        account.setName("Jock");
        account.setMoney(666.66);
    }

    @Test
    public void changeMoney() {
        accountService.changeMoney(1,200D);
    }

    @Test
    public void findMondyById() {
        Double money = accountService.findMondyById(1);
        System.out.println(money);
    }
    
}
```

#### SpringConfig.java

```java
package com.itheima.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration
@ComponentScan("com.itheima")
@Import(RedisConfig.class)
public class SpringConfig {
}

```

#### RedisConfig.java

```java
package com.itheima.config;

import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.PropertySource;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.jedis.JedisClientConfiguration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@PropertySource("redis.properties")
public class RedisConfig {

    @Value("${redis.host}")
    private String hostName;

    @Value("${redis.port}")
    private Integer port;

//    @Value("${redis.password}")
//    private String password;

    @Value("${redis.maxActive}")
    private Integer maxActive;
    @Value("${redis.minIdle}")
    private Integer minIdle;
    @Value("${redis.maxIdle}")
    private Integer maxIdle;
    @Value("${redis.maxWait}")
    private Integer maxWait;



    @Bean
    //配置RedisTemplate
    public RedisTemplate createRedisTemplate(RedisConnectionFactory redisConnectionFactory){
        //1.创建对象
        RedisTemplate redisTemplate = new RedisTemplate();
        //2.设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        //3.设置redis生成的key的序列化器，对key编码进行处理
        RedisSerializer keyStringSerializer = new StringRedisSerializer();
        RedisSerializer valueStringSerializer = new GenericJackson2JsonRedisSerializer();
        /**
         * 如果不配置Serializer，那么存储的时候缺省使用String，如果用User类型存储
         * ，那么会提示错误User can't cast to String！！
         */
        redisTemplate.setKeySerializer(keyStringSerializer);
        redisTemplate.setValueSerializer(valueStringSerializer);
        redisTemplate.setHashKeySerializer(keyStringSerializer);
        redisTemplate.setHashValueSerializer(valueStringSerializer);
        //4.返回
        return redisTemplate;
    }

    @Bean
    //配置Redis连接工厂
    public RedisConnectionFactory createRedisConnectionFactory(RedisStandaloneConfiguration redisStandaloneConfiguration,GenericObjectPoolConfig genericObjectPoolConfig){
        //1.创建配置构建器，它是基于池的思想管理Jedis连接的
        JedisClientConfiguration.JedisPoolingClientConfigurationBuilder builder = (JedisClientConfiguration.JedisPoolingClientConfigurationBuilder)JedisClientConfiguration.builder();
        //2.设置池的配置信息对象
        builder.poolConfig(genericObjectPoolConfig);
        //3.创建Jedis连接工厂
        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory(redisStandaloneConfiguration,builder.build());
        //4.返回
        return jedisConnectionFactory;
    }

    @Bean
    //配置spring提供的Redis连接池信息
    public GenericObjectPoolConfig createGenericObjectPoolConfig(){
        //1.创建Jedis连接池的配置对象
        GenericObjectPoolConfig genericObjectPoolConfig = new GenericObjectPoolConfig();
        //2.设置连接池信息
        genericObjectPoolConfig.setMaxTotal(maxActive);
        genericObjectPoolConfig.setMinIdle(minIdle);
        genericObjectPoolConfig.setMaxIdle(maxIdle);
        genericObjectPoolConfig.setMaxWaitMillis(maxWait);
        //3.返回
        return genericObjectPoolConfig;
    }


    @Bean
    //配置Redis标准连接配置对象
    public RedisStandaloneConfiguration createRedisStandaloneConfiguration(){
        //1.创建Redis服务器配置信息对象
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        //2.设置Redis服务器地址，端口和密码（如果有密码的话）
        redisStandaloneConfiguration.setHostName(hostName);
        redisStandaloneConfiguration.setPort(port);
//        redisStandaloneConfiguration.setPassword(RedisPassword.of(password));
        //3.返回
        return redisStandaloneConfiguration;
    }

}
```

#### AccountServiceImpl.java

```java
package com.itheima.service.impl;


import com.itheima.domain.Account;
import com.itheima.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;

@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Autowired
    private RedisTemplate redisTemplate;

    @Override
    public void save(Account account) {
    }

    @Override
    public void changeMoney(Integer id, Double money) {
        //等同于redis中set account:id:1 100
        redisTemplate.opsForValue().set("account:id:"+id,money);
    }

    @Override
    public Double findMondyById(Integer id) {
        //等同于redis中get account:id:1
        Object money = redisTemplate.opsForValue().get("account:id:" + id);
        return new Double(money.toString());
    }
}
//        redisTemplate.type()
//        redisTemplate.persist()
//        redisTemplate.move()
//        redisTemplate.hasKey()
//        redisTemplate.getExpire()
//        redisTemplate.expire()
//        redisTemplate.delete()
//        redisTemplate.rename();
//
//        redisTemplate.opsForValue().;
//        redisTemplate.opsForHash().;
//        redisTemplate.opsForList().;
//        redisTemplate.opsForSet().;
//        redisTemplate.opsForZSet();
//
//
//        redisTemplate.boundValueOps().;
//
//        redisTemplate.slaveOf();
//        redisTemplate.slaveOfNoOne();
//
//        redisTemplate.opsForCluster()
```

#### redis.properties

```properties
# redis服务器主机地址
redis.host=192.168.40.130
#redis服务器主机端口
redis.port=6378

#redis服务器登录密码
#redis.password=itheima

#最大活动连接
redis.maxActive=20
#最大空闲连接
redis.maxIdle=10
#最小空闲连接
redis.minIdle=0
#最大等待时间
redis.maxWait=-1
```



### 5 底层原理解析-设计模式

#### 策略模式应用

⚫ 策略模式（Strategy Pattern）使用不同策略的对象实现不同的行为方式，策略对象的变化导致行为的 变化。

```java

```

