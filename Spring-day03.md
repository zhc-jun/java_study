## AOP

### **学习目标**：

​    **1）** 基于xml或者注解实现 aop 切面

​    **2）** 能够说出aop有多少种通知类型，区别是什么

​    **3）**完成案例



### 1. AOP简介 

- AOP（Aspect Oriented Programming）称为面向切面编程，在程序开发中主要用来解决一些系统层面上的问题，比如日志，事务，权限, 登录拦截验证等，Struts2或springmvc 的拦截器设计就是基于AOP的思想，是个比较经典的例子。
- 在不改变原有的逻辑的基础上，增加一些额外的功能。代理也是这个功能，读写分离也能用aop来做。

**优点：**

AOP不改变原有的逻辑代码，只对其在切面做增强，所以有如下好处：

⚫ 提高代码的可重用性 ⚫ 业务代码编码更简洁 ⚫ 业务代码维护更高效 ⚫ 业务功能扩展更便捷



### 2. 入门案例

#### 2.1 aop相关概念

⚫ Joinpoint(连接点)：就是方法 

⚫ Pointcut(切入点)：就是挖掉共性功能的方法 

⚫ Advice(通知)：就是共性功能，最终以一个方法的形式呈现 

⚫ Aspect(切面)：就是共性功能与挖的位置的对应关系 

⚫ Target(目标对象)：就是挖掉功能的方法对应的类产生的对象，这种对象是无法直接完成最终工作的 

⚫ Weaving(织入)：就是将挖掉的功能回填的动态过程 

⚫ Proxy(代理)：目标对象无法直接完成工作，需要对其进行功能回填，通过创建原始对象的代理对象实现 

⚫ Introduction(引入/引介) ：就是对原始对象无中生有的添加成员变量或成员方法



#### 2.1 Advice通知类型介绍

- Before:在目标方法被调用之前做增强处理,@Before只需要指定切入点表达式即可

- AfterReturning:在目标方法正常完成后做增强,@AfterReturning除了指定切入点表达式后，还可以指定一个返回值形参名returning,代表目标方法的返回值

- AfterThrowing:主要用来处理程序中未处理的异常,@AfterThrowing除了指定切入点表达式后，还可以指定一个throwing的返回值形参名,可以通过该形参名

来访问目标方法中所抛出的异常对象

- After:在目标方法完成之后做增强，无论目标方法时候成功完成。@After可以指定一个切入点表达式

- Around:环绕通知,在目标方法完成前后做增强处理,环绕通知是最重要的通知类型,像事务,日志等都是环绕通知,注意编程中核心是一个ProceedingJoinPoint



#### 2.2 示例代码

pom.xml

⚫ 说明： 

◆ 因为第三方bean无法在其源码上进行修改，使用@Bean解决第三方bean的引入问题 

◆ 该注解用于替代XML配置中的静态工厂与实例工厂创建bean，不区分方法是否为静态或非静态 

◆ @Bean所在的类必须被spring扫描加载并被 @Controller、@Service、@Repository  @Component所修饰，否则该注解无法生效 。

⚫ 第三方资源注解配置方式 ◆ 需要引用添加名称 ◆ 无需引用省略名称

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.1.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.4</version>
    </dependency>
</dependencies>
```

applicationContext.xml

```xml
<!--3.开启AOP命名空间-->
<bean id="userService" class="com.itheima.service.impl.UserServiceImpl"/>
<!--2.配置共性功能成功spring控制的资源-->
<bean id="myAdvice" class="com.itheima.aop.AOPAdvice"/>

<!--4.配置AOP-->
<aop:config>
    <!--5.配置切入点-->
    <aop:pointcut id="pt" expression="execution(* *..*(..))"/>
    <!--6.配置切面（切入点与通知的关系）-->
    <aop:aspect ref="myAdvice">
        <!--7.配置具体的切入点对应通知中那个操作方法-->
        <aop:before method="myfunction" pointcut-ref="pt"/>
    </aop:aspect>
</aop:config>
```

AOPAdvice.java

```java
package com.itheima.aop;
//1.制作通知类，在类中定义一个方法用于完成共性功能
public class AOPAdvice {

    public void myfunction(){
        System.out.println("共性功能");
    }
}
```

UserService.java    UserServiceImpl.java

```java
package com.itheima.service;

public interface UserService {
    public void save();
}
//实现类
package com.itheima.service.impl;
import com.itheima.service.UserService;
public class UserServiceImpl implements UserService {

    public void save(){
        //0.将共性功能抽取出来
        //System.out.println("共性功能");
        System.out.println("user service running...");
    }

}
```



### 3. AOP配置 （XML）

#### 3.1 aop:config 标签

- 作用：设置AOP
- 说明：一个beans标签中可以配置多个aop:config标签

```xml
<bean id="myAdvice" class="com.itheima.aop.AOPAdvice"/>

<!---AOP基本配置-->
<!---AOP基本配置-->
<aop:config>
     <aop:pointcut id="pt" expression="execution(* *..*(..))"/>
     <aop:aspect ref="myAdvice">
         <aop:before method="before" pointcut-ref="pt"/>
     </aop:aspect>
</aop:config>
```

#### 3.2 aop:aspect 标签

- 作用：设置具体的AOP通知对应的切入点 
- 归属：aop:config标签
- 说明：一个aop:config标签中可以配置多个aop:aspect标签 
- 基本属性： ◆ ref ：通知所在的bean的id

```xml
<bean id="myAdvice" class="com.itheima.aop.AOPAdvice"/>
<!---AOP基本配置-->
<aop:config>
     <aop:pointcut id="pt" expression="execution(* *..*(..))"/>
     <aop:aspect ref="myAdvice">
         <aop:before method="before" pointcut-ref="pt"/>
     </aop:aspect>
</aop:config>
```

#### 3.3 aop:pointcut 标签

- 作用：设置切入点 
- 归属：aop:config标签、aop:aspect标签
- 说明： 一个aop:config标签中可以配置多个aop:pointcut标签，且该标签可以配置在aop:aspect标签内
- 基本属性： ◆ id ：识别切入点的名称 ◆ expression ：切入点表达式

```xml
<bean id="myAdvice" class="com.itheima.aop.AOPAdvice"/>
<!---AOP基本配置-->
<aop:config>
     <aop:pointcut id="pt" expression="execution(* *..*(..))"/>
     <aop:aspect ref="myAdvice">
         <aop:before method="before" pointcut-ref="pt"/>
     </aop:aspect>
</aop:config>
```

#### 3.4 切入点

- 切入点描述的是某个方法 
- 切入点表达式是一个快速匹配方法描述的通配格式，类似于正则表达式

**切入点表达式的组成**

- 关键字（访问修饰符 返回值 包名.类名.方法名（参数）异常名）

  execution（public User com.itheima.service.UserService.findById（int））

  ◆ 关键字：描述表达式的匹配模式（参看关键字列表） 

  ◆ 访问修饰符：方法的访问控制权限修饰符 

  ◆ 类名：方法所在的类（此处可以配置接口名称） 

  ◆ 异常：方法定义中指定抛出的异常 

- 切入点描述的是某个方法 
- 切入点表达式是一个快速匹配方法描述的通配格式，类似于正则表达式

**切入点表达式——通配符**

⚫ * ：单个独立的任意符号，可以独立出现，也可以作为前缀或者后缀的匹配符出现

```
◆ 匹配com.itheima包下的任意包中的UserService类或接口中所有find开头的带有一个参数的方法 
execution（public * com.itheima.*.UserService.find*（*））
```

⚫ ..：**多个**连续的任意符号，可以独立出现，常用于简化包名与参数的书写

```
◆ 匹配com包下的任意包中的UserService类或接口中所有名称为findById的方法 
execution（public User com..UserService.findById（..））
```

⚫ +：专用于匹配子类类型, 语法：至少出现一个

```
execution(* *..*Service+.*(..))
```

**切入点表达式——范例**

```
execution(* *(..)) 
execution(* *..*(..)) 
execution(* *..*.*(..)) 
execution(public * *..*.*(..)) 
execution(int *..*.*(..)) 
execution(void *..*.*(..)) 
execution(void com..*.*(..)) 
execution(void com..service.*.*(..)) 
execution(void com.itheima.service.*.*(..)) 
execution(void com.itheima.service.User*.*(..)) 
execution(void com.itheima.service.*Service.*(..)) 
execution(void com.itheima.service.UserService.*(..)) execution(User com.itheima.service.UserService.find*(..)) execution(User com.itheima.service.UserService.*Id(..)) 
execution(User com.itheima.service.UserService.findById(..)) execution(User com.itheima.service.UserService.findById(int)) execution(User com.itheima.service.UserService.findById(int,int)) execution(User com.itheima.service.UserService.findById(int,*)) execution(User com.itheima.service.UserService.findById(*,int)) execution(public User com.itheima.service.UserService.findById()) execution(List com.itheima.service.*Service+.findAll(..))
```

**切入点的三种配置方式**

- 公共，局部，私有，一般使用公共切入点即可

```xml
<!--切入点三种配置方式-->
<aop:config>
    <!---公共切入点-->
    <aop:pointcut id="pt" expression="execution(* *..*(..))"/>
	<aop:aspect ref="myAdvice">
        <!--局部切入点-->
        <aop:pointcut id="pt2" expression="execution(* *..*(..))"/>
        <!--私有切入点-->
        <aop:before method="before" pointcut="execution(* *..*(..))"/>
    </aop:aspect>
</aop:config>
```



#### 3.5 通知类型

⚫ AOP的通知类型共5种 

◆ 前置通知：原始方法执行前执行，如果通知中抛出异常，阻止原始方法运行 应用：数据校验 

◆ 后置通知：原始方法执行后执行，无论原始方法中是否出现异常，都将执行通知 应用：现场清理 

◆ 返回后通知：原始方法正常执行完毕并返回结果后执行，如果原始方法中抛出异常，无法执行 应用：返回值相关数据处理 

◆ 抛出异常后通知：原始方法抛出异常后执行，如果原始方法没有抛出异常，无法执行 应用：对原始方法中出现的异常信息进行处理 

◆ 环绕通知：在原始方法执行前后均有对应执行，还可以阻止原始方法的执行 应用：十分强大，可以做任何事情



##### 3.5.1 aop:before 标签

- 作用：设置前置通知 

- 归属：aop:aspect标签 

- 说明：一个aop:aspect标签中可以配置多个aop:before标签

- 基本属性： 

  ◆ method ：在通知类中设置当前通知类别对应的方法 

  ◆ pointcut ：设置当前通知对应的切入点表达式，与pointcut-ref属性冲突 

  ◆ pointcut-ref：设置当前通知对应的切入点id，与pointcut属性冲突

```xml
<bean id="myAdvice" class="com.itheima.aop.AOPAdvice"/>
<!---AOP基本配置-->
<aop:config>
     <aop:pointcut id="pt" expression="execution(* *..*(..))"/>
     <aop:aspect ref="myAdvice">
         <aop:before method="before" pointcut-ref="pt"/>
     </aop:aspect>
</aop:config>
```

##### 3.5.2 aop:after  标签

- 作用：设置后置通知 

- 归属：aop:aspect标签 

- 说明：一个aop:aspect标签中可以配置多个aop:after标签 

- 基本属性： 

  ◆ method ：在通知类中设置当前通知类别对应的方法 

  ◆ pointcut ：设置当前通知对应的切入点表达式，与pointcut-ref属性冲突 

  ◆ pointcut-ref：设置当前通知对应的切入点id，与pointcut属性冲突

```xml
<bean id="myAdvice" class="com.itheima.aop.AOPAdvice"/>
<!--五种通知类型-->
<aop:config>
    <aop:pointcut id="pt" expression="execution(* *..*(..))"/>
    <aop:aspect ref="myAdvice">
        <aop:after method="after" pointcut-ref="pt"/>
    </aop:aspect>
</aop:config>
```

##### 3.5.3 aop:after-returning   标签

- 作用：设置返回后通知 

- 归属：aop:aspect标签 

- 说明：一个aop:aspect标签中可以配置多个aop:after-returning标签 

- 基本属性： 

  ◆ method ：在通知类中设置当前通知类别对应的方法 

  ◆ pointcut ：设置当前通知对应的切入点表达式，与pointcut-ref属性冲突 

  ◆ pointcut-ref：设置当前通知对应的切入点id，与pointcut属性冲突

```xml
<bean id="myAdvice" class="com.itheima.aop.AOPAdvice"/>
<!--五种通知类型-->
<aop:config>
    <aop:pointcut id="pt" expression="execution(* *..*(..))"/>
    <aop:aspect ref="myAdvice">
        <aop:after-returning method="afterReturing" pointcut-ref="pt"/>
    </aop:aspect>
</aop:config>
```

##### 3.5.4 aop:after-throwing    标签

- 作用：设置抛出异常后通知 

- 归属：aop:aspect标签 

- 说明：一个aop:aspect标签中可以配置多个aop:after-throwing标签 

- 基本属性： 

  ◆ method ：在通知类中设置当前通知类别对应的方法 

  ◆ pointcut ：设置当前通知对应的切入点表达式，与pointcut-ref属性冲突 

  ◆ pointcut-ref：设置当前通知对应的切入点id，与pointcut属性冲突

```xml
<bean id="myAdvice" class="com.itheima.aop.AOPAdvice"/>
<!--五种通知类型-->
<aop:config>
    <aop:pointcut id="pt" expression="execution(* *..*(..))"/>
    <aop:aspect ref="myAdvice">
        <aop:after-throwing method="afterThrowing" pointcut-ref="pt"/>
        <aop:around method="around" pointcut-ref="pt"/>
    </aop:aspect>
</aop:config>
```

##### 3.5.5 aop:around  标签

- 作用：设置环绕通知 

- 归属：aop:aspect标签 

- 说明：一个aop:aspect标签中可以配置多个aop:around标签 

- 基本属性： 

  ◆ method ：在通知类中设置当前通知类别对应的方法 

  ◆ pointcut ：设置当前通知对应的切入点表达式，与pointcut-ref属性冲突 

  ◆ pointcut-ref：设置当前通知对应的切入点id，与pointcut属性冲突

```xml
<bean id="myAdvice" class="com.itheima.aop.AOPAdvice"/>
<!--五种通知类型-->
<aop:config>
    <aop:pointcut id="pt" expression="execution(* *..*(..))"/>
    <aop:aspect ref="myAdvice">
        <aop:around method="around" pointcut-ref="pt"/>
    </aop:aspect>
</aop:config>
```

###### 环绕通知注意事项

⚫ 环绕通知是在原始方法的前后添加功能，在环绕通知中，存在对原始方法的显式调用

```java
    public Object around(ProceedingJoinPoint pjp) {
        System.out.println("around before...");
        Object ret = null;
        try {
            //对原始方法的调用
            ret = pjp.proceed();
        } catch (Throwable throwable) {
            System.out.println("around...exception...."+throwable.getMessage());
        }
        System.out.println("around after..."+ret);
        return ret;
    }
```

⚫ 环绕通知方法相关说明： 

​	◆ 方法须设定Object类型的返回值，否则会拦截原始方法的返回。如果原始方法返回值类型为 void，通知方法也可以设定返回值类型为void，最终返回null 

​	◆ 方法需在第一个参数位置设定ProceedingJoinPoint对象，通过该对象调用proceed()方法，实 现对原始方法的调用。如省略该参数，原始方法将无法执行 	◆ 使用proceed()方法调用原始方法时，因无法预知原始方法运行过程中是否会出现异常，强制抛 出Throwable对象，封装原始方法中可能出现的异常信息



#### 3.6 通知顺序（了解）

⚫ 当同一个切入点配置了多个通知时，通知会存在运行的先后顺序，该顺序以通知配置的顺序为准



#### 3.7 通知获取参数数据

##### 原始方法

```java
public void save(int i,int m){
    System.out.println("user service running..."+i+","+m);
}
```

##### 通知方法

```java
public void before1(int x,int y){
     System.out.println("before(int)..."+x+","+y);
}  
```

##### 方式一 JoinPoint

```java
public void after(JoinPoint jp){
    Object[] args = jp.getArgs();
    System.out.println("after..."+args[0]);
}
```

##### 方式二 xml配置

```java
<aop:before  method="before1"
     pointcut="execution(* *..*(int,int)) &amp;&amp; args(x,y)"/>
```

#### 3.8 通知获取返回值数据

##### xml 配置

```xml
<!--通知获取原始方法返回值信息-->
<aop:config>
    <aop:pointcut id="pt" expression="execution(* *..*(..))"/>
    <aop:aspect ref="myAdvice">
        <aop:after-returning method="afterReturing" pointcut-ref="pt" returning="ret"/>
        <aop:around method="around" pointcut-ref="pt"/>
    </aop:aspect>
</aop:config>
```

##### AOPAdvice.java

```java
public void afterReturing(Object ret){
	System.out.println("afterReturing..."+ret);
}

public Object around(ProceedingJoinPoint pjp) {
    System.out.println("around before...");
    Object ret = null;
    try {
        //对原始方法的调用
        ret = pjp.proceed();
    } catch (Throwable throwable) {
        System.out.println("around...exception...."+throwable.getMessage());
    }
    System.out.println("around after..."+ret);
    return ret;
}
```

#### 3.9 通知获取异常数据

- 如果 环绕通知存在, 异常就会被捕获, 导致 after-throwing 无法触发

##### xml 配置

```xml
<!--通知获取原始方法异常信息-->
<aop:config>
    <aop:pointcut id="pt" expression="execution(* *..*(..))"/>
    <aop:aspect ref="myAdvice">
        <aop:after-throwing method="afterThrowing" pointcut-ref="pt" throwing="t"/>
        <!--如果 环绕通知存在, 异常就会被捕获, 导致 after-throwing 无法触发 -->
        <!--<aop:around method="around" pointcut-ref="pt"/>-->
    </aop:aspect>
</aop:config>
```

##### AOPAdvice.java

```java
public void afterThrowing(Throwable t){
    System.out.println("afterThrowing..."+t.getMessage());
}

public Object around(ProceedingJoinPoint pjp) {
    System.out.println("around before...");
    Object ret = null;
    try {
        //对原始方法的调用
        ret = pjp.proceed();
    } catch (Throwable throwable) {
        System.out.println("around...exception...."+throwable.getMessage());
    }
    System.out.println("around after..."+ret);
    return ret;
}
```



### 4.  AOP配置 （注解）

#### 4.1 注解开发AOP

⚫ 在XML格式基础上 

​	◆ 导入坐标（伴随spring-context坐标导入已经依赖导入完成） 

​	◆ 开启AOP注解支持 

​	◆ 配置切面@Aspect 

​	◆ 定义专用的切入点方法，并配置切入点@Pointcut 

​	◆ 为通知方法配置通知类型及对应切入点@Before

##### applicationContext.xml

```xml
<context:component-scan base-package="com.itheima"/>
<aop:aspectj-autoproxy/>
```

##### AOPAdvice.java

- @Aspect 替代 aop:aspect 标签
- @Pointcut 替代 aop:pointcut 标签
- @Before @After @AfterReturning @AfterThrowing @Around 替代掉 aop:before  aop:after  aop:after-returning  aop:after-throwing  aop:around 标签

```java
package com.itheima.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class AOPAdvice {

    @Pointcut("execution(* *..*(..))")
    public void pt() {
    }

    @Before("pt()")
    public void before() {
        System.out.println("前置before...");
    }

    @After("pt()")
    public void after() {
        System.out.println("后置after...");
    }

    @AfterReturning("pt()")
    public void afterReturing() {
        System.out.println("返回后afterReturing...");
    }

    @AfterThrowing("pt()")
    public void afterThrowing() {
        System.out.println("抛出异常后afterThrowing...");
    }

    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("环绕前around before...");
        Object ret = pjp.proceed();
        System.out.println("环绕后around after...");
        return ret;
    }

}

```

#### 4.2 AOP注解开发通知执行顺序（了解）

⚫ AOP使用XML配置情况下，通知的执行顺序由配置顺序决定，在注解情况下由于不存在配置顺序的概念，参照通知所配置的方法名字符串对应的编码值顺序，可以简单理解为字母排序 

​	◆ 同一个通知类中，相同通知类型以方法名排序为准 

​	◆ 不同通知类中，以类名排序为准 

​	◆ 使用@Order注解通过变更bean的加载顺序改变通知的加载顺序 

⚫ 企业开发经验 

​	◆ 通知方法名由3部分组成，分别是前缀、顺序编码、功能描述 

​	◆ 前缀为固定字符串，例如baidu、itheima等，无实际意义 

​	◆ 顺序编码为6位以内的整数，通常3位即可，不足位补0 

​	◆ 功能描述为该方法对应的实际通知功能，例如exception、strLenCheck 

​	◆ 控制通知执行顺序使用顺序编码控制，使用时做一定空间预留 

​		◼ 003使用，006使用，预留001、002、004、005、007、008 

​		◼ 使用时从中段开始使用，方便后期做前置追加或后置追加 

​		◼ 最终顺序以运行顺序为准，以测试结果为准，不以设定规则为准



#### 4.3 AOP注解驱动

- @EnableAspectJAutoProxy 开启注解支持，功能等价: aop:aspectj-autoproxy 标签

##### SpringConfig.java

```java
@Configuration
@ComponentScan("com.itheima")
//开启AOP功能
@EnableAspectJAutoProxy
public class SpringConfig {
}

```

##### UserServiceImpl.java

```java
package com.itheima.service.impl;
import com.itheima.service.UserService;
import org.springframework.stereotype.Service;

@Service("userService")
public class UserServiceImpl implements UserService {

    public int save(int i,int m){
        System.out.println("user service running..."+i+","+m);
        return 100;
    }
}
```

##### UserServiceTest.java

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfig.class)
public class UserServiceTest {
    @Autowired
    private UserService userService;

    @Test
    public void testSave(){
        int ret = userService.save(888, 666);
        Assert.assertEquals(100,ret);
    }
}

```



### 5 综合案例-性能监控

⚫ 测量接口执行效率：接口方法执行前后获取执行时间，求出执行时长 

​		◆ System.currentTimeMillis( ) 

⚫ 对项目进行监控：项目中所有接口方法，AOP思想，执行期动态织入代码 

​		◆ 环绕通知 

​		◆ proceed()方法执行前后获取系统时间

#### RunTimeMonitorAdvice.java

```java
package com.itheima.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class RunTimeMonitorAdvice {

    //切入点，监控业务层接口
    @Pointcut("execution(* com.itheima.service.*Service.find*(..))")
    public void pt(){}

    @Around("pt()")
    public Object runtimeAround(ProceedingJoinPoint pjp) throws Throwable {
        //获取执行签名信息
        Signature signature = pjp.getSignature();
        //通过签名获取执行类型（接口名）
        String className = signature.getDeclaringTypeName();
        //通过签名获取执行操作名称（方法名）
        String methodName = signature.getName();

        //执行时长累计值
        long sum = 0L;

        for (int i = 0; i < 10000; i++) {
            //获取操作前系统时间beginTime
            long startTime = System.currentTimeMillis();
            //原始操作调用
            pjp.proceed(pjp.getArgs());
            //获取操作后系统时间endTime
            long endTime = System.currentTimeMillis();
            sum += endTime-startTime;
        }
        //打印信息
        System.out.println(className+":"+methodName+"   (万次）run:"+sum+"ms");
        return null;
    }
}

```

#### 案例后续思考与设计

⚫ 测量真实性

​		 ◆ 开发测量是隔离性反复执行某个操作，是理想情况，上线测量差异过大 

​		◆ 上线测量服务器性能略低于单机开发测量 

​		◆ 上线测量基于缓存的性能查询要优于数据库查询测量 

​		◆ 上线测量接口的性能与最终对外提供的服务性能差异过大 

​		◆ 当外部条件发生变化（硬件），需要进行回归测试，例如数据库迁移 

⚫ 测量结果展示 

​		◆ 测量结果无需每一个都展示，需要设定检测阈值 

​		◆ 阈值设定要根据业务进行区分，一个复杂的查询与简单的查询差异化很大 

​		◆ 阈值设定需要做独立的配置文件或通过图形工具配置（工具级别的开发） 

​		◆ 配合图形界面展示测量结果



### 6.  AOP底层原理

 **JDK动态代理和CGLIB字节码生成的区别？**
	 1）JDK动态代理只能对实现了接口的类生成代理，而不能针对类
 	2）CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法
  因为是继承，所以该类或方法最好不要声明成final 

​     3）JDK动态代理生成新的代理类，CGLIB会 对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。 

​     <font color=red>4） spring默认使用jdk动态代理，如果类没有接口，则使用cglib。 </font>

#### 6.1 装饰模式

##### UserServiceImplDecorator.java

```java
package base.decorator;

import com.itheima.service.UserService;

public class UserServiceImplDecorator implements UserService {

    private UserService userService;

    public UserServiceImplDecorator(UserService userService){
        this.userService  = userService;
    }

    public void save() {
        //原始调用
        userService.save();
        //增强功能（后置）
        System.out.println("刮大白");
    }
}
```

##### UserServiceImpl.java

```java
package com.itheima.service.impl;

import com.itheima.service.UserService;

public class UserServiceImpl implements UserService{
    public void save() {
        System.out.println("水泥墙");
    }
}
```

##### App.java

```java
package base.decorator;

import com.itheima.service.UserService;
import com.itheima.service.impl.UserServiceImpl;

public class App {
    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
        UserService userService1 = new UserServiceImplDecorator(userService);
        userService1.save();
    }
}

```

#### 6.2 JDK 动态代理

##### UserServiceJDKProxy.java

```java
package base.proxy;

import com.itheima.service.UserService;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class UserServiceJDKProxy {

    public static UserService createUserServiceJDKProxy(final UserService userService){
        //获取被代理对象的类加载器
        ClassLoader cl = userService.getClass().getClassLoader();
        //获取被代理对象实现的接口
        Class[] classes = userService.getClass().getInterfaces();
        //对原始方法执行进行拦截并增强
        InvocationHandler ih = new InvocationHandler() {
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                //原始调用
                Object ret = method.invoke(userService, args);
                //后置增强内容
                System.out.println("刮大白2");
                System.out.println("贴壁纸2");
                return ret;
            }
        };
        //使用原始被代理对象创建新的代理对象
        UserService service = (UserService )Proxy.newProxyInstance(cl,classes,ih);
        return service;
    }

}
```

##### UserServiceImpl.java

```java
package com.itheima.service.impl;

import com.itheima.service.UserService;

public class UserServiceImpl implements UserService{
    public void save() {
        System.out.println("水泥墙");
    }
}
```

##### App.java

```java
package base.proxy;

import com.itheima.service.UserService;
import com.itheima.service.impl.UserServiceImpl;

public class App {
    public static void main(String[] args) {
        UserService userService  = new UserServiceImpl();
        UserService userService1 = UserServiceJDKProxy.createUserServiceJDKProxy(userService);
        userService1.save();
    }
}

```

#### 6.3 Cglib 动态代理

##### UserServiceCglibProxy.java

```java
package base.cglib;

import com.itheima.service.UserService;
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class UserServiceCglibProxy {

    public static UserService createUserServiceCglibProxy(Class clazz){
        //创建Enhancer对象（可以理解为内存中动态创建了一个类的字节码）
        Enhancer enhancer = new Enhancer();
        //设置Enhancer对象的父类是指定类型UserServerImpl
        enhancer.setSuperclass(clazz);
        //设置回调方法
        enhancer.setCallback(new MethodInterceptor() {
            public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                //通过调用父类的方法实现对原始方法的调用
                Object ret = methodProxy.invokeSuper(o, args);
                //后置增强内容，与JDKProxy区别：JDKProxy仅对接口方法做增强，cglib对所有方法做增强，包括Object类中的方法
                if(method.getName().equals("save")) {
                    System.out.println("刮大白3");
                    System.out.println("贴墙纸3");
                }
                return ret;
            }
        });
        //使用Enhancer对象创建对应的对象
        return (UserService) enhancer.create();
    }
}

```

##### UserServiceImpl.java

```java
package com.itheima.service.impl;

import com.itheima.service.UserService;

public class UserServiceImpl implements UserService{
    public void save() {
        System.out.println("水泥墙");
    }
}
```

##### App.java

```java
package base.cglib;

import com.itheima.service.UserService;
import com.itheima.service.impl.UserServiceImpl;

public class App {

    public static void main(String[] args) {
        UserService userService = UserServiceCglibProxy.createUserServiceCglibProxy(UserServiceImpl.class);
        userService.save();
    }
}
```

#### 6.4 代理模式的选择

⚫ Spirng可以通过配置的形式控制使用的代理形式，默认使用jdkproxy，通过配置可以修改为使用cglib 

**◆ XML配置**

```xml
<!--XMP 配置 AOP-->
<aop:config proxy-target-class="false">
</aop:config>
```

**◆ XML注解支持**

```java
<!-注解配置 AOP-->
<aop:aspectj-autoproxy proxy-target-class="false"/>
```

**◆ 注解驱动**

```java
// 注解驱动
@EnableAspectJAutoProxy(proxyTargetClass = true)
```

#### 6.5 织入时机

- 编译期：.java 编译 为 .class 过程中进行织入

  1）运行时速度快，2）灵活性差，3）编译即锁定

- 加载期：.class 文件加载进内存之前进行织入

  1）运行时速度快，2）灵活性中，3）多次加载可变更实现

- 运行期：字节码对象在内存运行时进行织入

  1）运行时速度慢，2）灵活性强，3）每次运行均可改变实现

- **spring 是在加载期完成代码的织入**