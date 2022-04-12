## 注解开发

### **学习目标**：

一.将spring 框架学习的第一天内容xml 配置形式改变成用**纯注解形式开发**（学习开发中的常用注解）

二.结合学习的常用注解 将spring 框架整合mybatis 框架改造成纯注解开发

三.讲解下sping 框架的一些底层原理（了解即可）



​    **1）**注解实现类对象的注册到容器：

​          1.自己定义的类：@Component @Controller @Service @Repository 

​          2.三方类：@Bean 注意: 该注解所在的类必须被 上述注解任意一个所修饰

​     **2）**注解实现属性依赖注入:

​           1.普通属性：@Value 可以修饰属性 和 set 方法，推荐修饰属性，因为可以不提供setter方法

​           2.引用类型属性：@Autowired 类型注入   @Qualifier 指定bean id注入，@Resource 可以替代掉 @Autowired @Qualifier, 该注解不加任何属性，默认类型注入；

​      **3）**@Primary 作用 (了解)：设置类对应的bean按类型装配时优先装配，

​      <font color=red>注意：@Autowired默认按类型装配，当出现相同类型的bean，使用@Primary提高按类型自动装配的优先级，多个@Primary 会导致优先级设置无效</font>

​      **4)**  使用 @Configuration、@ComponentScan  替代  applicationContext.xml 文件

​       **5）**实现框架的整合 （包含Junit）



### 1.注解反转驱动的弊端

#### 1.1 什么是注解驱动

- 利用spring提供的注解替代掉xml文件中做的配置，干的活都是一样的

#### 1.2 注解驱动的弊端

- 为了达成注解驱动的目的，可能会将原先很简单的书写，变的更加复杂，毕竟注解也不是做各种配置都很灵活。

### 2. 常用注解

#### 2.1 启动注解功能

⚫ 启动注解扫描，加载类中配置的注解项

```xml
<context:component-scan base-package="packageName"/>
```

⚫ 说明： 

◆ 在进行包所扫描时，会对配置的包及其子包中所有文件进行扫描 

◆ 扫描过程是以文件夹递归迭代的形式进行的 

◆ 扫描过程仅读取合法的java文件 

◆ 扫描时仅读取spring可识别的注解 

◆ 扫描结束后会将可识别的有效注解转化为spring对应的资源加入IoC容器 

⚫ 注意： 

◆ 无论是注解格式还是XML配置格式，最终都是将资源加载到IoC容器中，差别仅仅是数据读取方式不同 

◆ 从加载效率上来说注解优于XML配置文件

```java
//定义bean，后面添加bean的id
@Component("userService")
//@scope默认是单例模式（singleton）即：@scope("singleton")
//1.singleton单例模式,全局有且仅有一个实例
//2.prototype原型模式,每次获取Bean的时候会有一个新的实例
@Scope("singleton")
public class UserServiceImpl implements UserService {
    
    //设定bean的生命周期-销毁
    @PreDestroy
    public void destroy(){
        System.out.println("user service destroy...");
    }
    //设定bean的生命周期-新建
    @PostConstruct
    public void init(){
        System.out.println("user service init...");
    }
}
```

```java
package com.itheima;
public class App {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");

        UserService userService = (UserService) ctx.getBean("userService");
        userService.save();
    }
}
```

#### 2.2 加载第三方资源

spring 框架 需要先扫描 到 类上的注解 @Component @Service ---> 然后才能识别@Bean(它是注解在方法上，并且方法返回一个对象值)

 ⚫ 位置：方法定义上方 

⚫ 说明： 

◆ 因为第三方bean无法在其源码上进行修改，使用@Bean解决第三方bean的引入问题 

◆ 该注解用于替代XML配置中的静态工厂与实例工厂创建bean，不区分方法是否为静态或非静态 

◆ @Bean所在的类必须被spring扫描加载并被 @Controller、@Service、@Repository  @Component所修饰，否则该注解无法生效 。

⚫ 第三方资源注解配置方式 ◆ 需要引用添加名称 ◆ 无需引用省略名称

```java
public class JDBCConfig {

    //定义bean的访问id
    @Bean("dataSource")
    public static DruidDataSource getDataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
        ds.setUsername("root");
        ds.setPassword("itheima");
        return ds;
    }
}

```

#### 2.3 非引用类型注入

⚫ 位置：定义属性上方，定义set方法上方 （用的很少）

⚫ 作用：设置对应属性的值或对方法进行传参 

⚫ 范例：

```java
@Value("${}")
private int num ;
@Value("itheima")
private String version;
```

⚫ 说明： 

◆ value值仅支持非引用类型数据，赋值时对方法的所有参数全部赋值 

◆ value值支持读取properties文件中的属性值，通过类属性将properties中数据传入类中 

◆ value值支持SpEL 

◆ @value注解如果添加在属性上方，可以省略set方法（set方法的目的是为属性赋值） 

⚫ 相关属性 

◆ value（默认）：定义对应的属性值或参数值

#### 2.4 引用类型注入

⚫ 说明： 

◆ @Autowired按类型装配, 同一个类型接口最好只有一个实现类

◆ @Autowired按类型装配, 同一个类型接口多个实现类时

​     **1）类型相同**：属性名与bean id 保持一致也可以完成注入, 如果没有bean id, 属性名为 实现类 首字母转小写 后的名称，也可注入。

​     **2）类型相同&名称与bean id匹配不上：**按照优先级@Primary完成注入

​     **3）类型相同&名称与bean id匹配不上&优先级一致：**报错

◆ @Qualifier指定自动装配的bean的id

◆ @Resource 可以替代掉 @Autowired @Qualifier 但是不推荐使用，最好还是用spring的注解的完成注入

- name：设置注入的bean的id 

- type：设置注入的bean的类型，接收的参数为Class类型
- **什么属性不指定, 默认按照类型注入**

```java
@Autowired
@Qualifier("userDao")
private UserDao userDao;

//默认类型注入  指定名称: @Resource(name = "book2")
@Resource
private BookDao bookDao;
```

@Primary 作用：设置类对应的bean按类型装配时优先装配 

```java
@Primary
public class ClassName{}
```

◆说明： 

- @Autowired默认按类型装配，当出现相同类型的bean，使用@Primary提高按类型自动装配的优先级，多个@Primary 会导致优先级设置无

#### 2.5 加载properties文件

@PropertySource ：定义类上方 ，作用：加载properties文件中的属性值 

```java
//一般只加载一个配置文件, ${key}
@PropertySource(value={"classpath:jdbc.properties","classpath:abc.properties"},ignoreResourceNotFound = true)
public class BookDaoImpl implements BookDao {
    
    @Value("${jdbc.userName}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;

}
```

#### 2.6 纯注解驱动制作 （重点）

⚫ @Configuration、@ComponentScan  定义在类上方， 作用：设置当前类为spring核心配置加载类, 替代掉applicationContext.xml

```java
@Configuration
@ComponentScan("com.itheima")
public class SpringConfig{
}
```

⚫ 说明： 

◆ 核心配合类用于替换spring核心配置文件，此类可以设置空的，不设置变量与属性 

◆ bean扫描工作使用注解@ComponentScan替代

⚫ 加载纯注解格式上下文对象，需要使用AnnotationConfigApplicationContext

```java
ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);

UserService userService = (UserService) ctx.getBean("userService");
userService.save();
```

#### 2.7 第三方bean配置与管理

⚫ @Import  定义类上方 , 作用：导入@Bean 所在的类  

⚫ 范例：

```java
//当前数组中只有一个值
@Import({JDBCConfig.class})
@ComponentScan("com.itheima")
@Configuration
public class SpringConfig {}

//被导入的类
public class JDBCConfig {
    @Bean("dataSource")
    public static DruidDataSource getDataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
        ds.setUsername("root");
        ds.setPassword("itheima");
        return ds;
    }
}
```

⚫ 说明： 

◆ @Import注解在同一个类上，仅允许添加一次，如果需要导入多个，使用数组的形式进行设定 

◆ 在被导入的类中可以继续使用@Import导入其他资源（了解） 

◆ @Bean所在的类可以使用导入的形式进入spring容器，无需声明为bean



#### 2.8 bean加载控制 (了解)

##### 2.8.1 @DependsOn("其他beanid")    

⚫ 定义在类上 ，作用：控制bean的加载顺序，使其指定bean加载完毕后再加载 

范例：

```java
@DependsOn("beanId") public class ClassName {
}
```

 **属性：**value（默认）：设置当前bean所依赖的bean的id

**注意**：

1、Spring管理的bean默认都是单例模式(singleton)

2、实例化对象应该顺序化的，比如A依赖B,B依赖C,C依赖D...

3、一个bean可以依赖多个bean，可以通过逗号(",")来定义多个依赖对象：

```java
@DependsOn("beanId1, beanId2, beanId3") public class ClassName {
}
```

##### 2.8.2 @Order("数字")  

⚫ 定义类上 ， 作用：控制@Configuration修饰类的加载顺序 

⚫ 范例：

```java
@Order(1)
public class SpringConfigClassName {
}
```

**注意**：

1. @Order 的数字值 越大优先级越低，例如 1 的优先级大于 2
2. 优先级越大的类中 @bean 修饰的方法优先注入。



##### 2.8.3 @Lazy 

**说明：**

1）用于指定单例bean实例化的时机，在没有指定此注解时，单例会在容器初始化时就被创建。而当使用此注解后，单例对象的创建时机会在该bean在被**第一次使用时创建**，并且只创建一次。第二次及以后获取使用就不再创建。

2）在实际开发场景中，并不是所有bean都要一开始就被创建的，有些可以等到使用时才创建。此时就可以使用该注解实现。

3）此注解只对单例bean有用，**原型bean时此注解不起作用。**

**实例：**

@Lazy注解作用于类上时，通常与@Component及其衍生注解配合使用。

```java
//与@component配合使用
@Component
@Lazy
public class UserService {

    public UserService(){
        System.out.println("userService创建了");
    }
}
```

@Lazy注解作用于方法上时，通常与@Bean注解配合使用。 

```java
@Configuration
@ComponentScan(basePackages = "lazyDemo")
public class SpringConfig {

    @Bean
    @Lazy
    //与@Bean注解配合使用
    public UserService userService1(){
        return new UserService();
    }
}

```



### 3. 整合第三方技术

**步骤：**

1. 修改mybatis外部配置文件格式为注解格式 
2. 业务类使用@Component声明bean，使用@Autowired注入对象
3.  建立配置文件JDBCConfig与MyBatisConfig类，并将其导入到核心配置类SpringConfig 
4. 开启注解扫描 
5. 使用AnnotationConfigApplicationContext对象加载配置项

#### App.java 

```java
import com.itheima.config.SpringConfig;
import com.itheima.domain.Account;
import com.itheima.service.AccountService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {
    public static void main(String[] args) {

        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        AccountService accountService = (AccountService) ctx.getBean("accountService");
        Account ac = accountService.findById(2);
        System.out.println(ac);
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
@Import({JDBCConfig.class,MyBatisConfig.class})
public class SpringConfig {
}

```

#### JDBCConfig.java

```java
package com.itheima.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;

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
}

```

#### MyBatisConfig.java

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



#### 注解整合Junit

1. Spring接管Junit的运行权，使用Spring专用的Junit类加载器 

2.  为Junit测试用例设定对应的spring容器

  注意：
  ◆ 从Spring5.0以后，要求Junit的版本必须是4.12及以上 ◆ Junit仅用于单元测试，不能将Junit的测试类配置成spring的bean，否则该配置将会被打包进入工 程中

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
    public void testFindById(){
        Account ac = accountService.findById(2);
//        System.out.println(ac);
        Assert.assertEquals("Jock1",ac.getName());
    }

    @Test
    public void testFindAll(){
        List<Account> list = accountService.findAll();
        Assert.assertEquals(3,list.size());
    }

}

```





### 4.  IoC底层核心原理

![image-20210221163624041](C:\Users\jiangyudeng\AppData\Roaming\Typora\typora-user-images\image-20210221163624041.png)

#### 4.1 设定组件扫描加载过滤器

@ComponentScan是告诉Spring 哪个packages 的用注解标识的类 会被spring自动扫描并且装入bean容器。

[excludeFilters](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.html#excludeFilters--)：指定不适合组件扫描的类型。

[includeFilters](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.html#includeFilters--)：指定哪些类型有资格用于组件扫描。

##### 注解过滤

- 过滤掉指定注解，让其不发挥具体功能

```java
@ComponentScan(
    value="com.itheima", // 设置基础扫描路径
    excludeFilters = // 设置过滤规则，当前为排除过滤
    @ComponentScan.Filter( // 设置过滤器
    type= FilterType.ANNOTATION, // 设置过滤方式为按照注解进行过滤
    classes=Repository.class) // 设置具体的过滤项，过滤所有 @Repository 修饰的 bean
)
```

##### 自定义过滤器

- public boolean match() 方法 返回false 不拦截，true 为拦截。

```java
//2.设置排除bean，排除的规则是自定义规则（FilterType.CUSTOM），具体的规则定义为（MyTypeFilter.class）
@ComponentScan(
        value = "com.itheima",
        excludeFilters = @ComponentScan.Filter(
                type= FilterType.CUSTOM,
                classes = MyTypeFilter.class
        )
)
public class SpringConfig {
}

/************************** 自定义过滤器 *******************/
package config.filter;

import org.springframework.core.type.ClassMetadata;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.core.type.filter.TypeFilter;

import java.io.IOException;

public class MyTypeFilter implements TypeFilter {
    @Override
    //加载的类满足要求，匹配成功
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        //通过参数获取加载的类的元数据
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        //通过类的元数据获取类的名称
        String className = classMetadata.getClassName();
        //如果加载的类名满足过滤器要求，返回匹配成功
        if(className.equals("com.itheima.service.impl.UserServiceImpl")){
            //返回true表示匹配成功，返回false表示匹配失败。此处仅确认匹配结果，不会确认是排除还是加入，排除/加入由配置项决定，与此处无关
            return true;
        }
        return false;
    }
}
```



#### 4.2 自定义导入器性

- 自定义导入器-可以帮助我们将指定类（没被@Service修饰）注入到bean容器中，功能类似@Bean。

#####  SpringConfig.java

```java
//自定义导入器
@Import(MyImportSelector.class)
public class SpringConfig {
}
```

##### import.properties

```properties
#2.加载import.properties文件中的单个类名
#className=com.itheima.dao.impl.BookDaoImpl

#3.加载import.properties文件中的多个类名
#className=com.itheima.dao.impl.BookDaoImpl,com.itheima.dao.impl.AccountDaoImpl

path=com.itheima.dao.impl.*

```

##### MyImportSelector.java

```java
package config.selector;

import org.springframework.context.annotation.ImportSelector;
import org.springframework.core.type.AnnotationMetadata;

import java.util.ResourceBundle;

public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
//      1.编程形式加载一个类
//      return new String[]{"com.itheima.dao.impl.BookDaoImpl"};

//      2.加载import.properties文件中的单个类名
//      ResourceBundle bundle = ResourceBundle.getBundle("import");
//      String className = bundle.getString("className");

//      3.加载import.properties文件中的多个类名
        ResourceBundle bundle = ResourceBundle.getBundle("import");
        String className = bundle.getString("className");
        return className.split(",");
    }
}

```



#### 4.3 自定义注册器

- 功能要比 @ComponentScan("com.itheima") 更强大，@ComponentScan("com.itheima")只会扫描包下被 @Service 等注解修饰的类注入到bean容器中,  而我们注册器配置的包扫描可以将指定包下所有类（未被注解修饰也算）都注入到bean容器中。
- 扩展的 CustomeImportBeanDefinitionRegistrar 可以给指定包下的所有类（未被@Service等注解修饰，注入到spring bean容器中。


##### SpringConfig.java

```java
//自定义注册器
@Import(MyImportBeanDefinitionRegistrar.class)
public class SpringConfig {
}
```

##### MyImportBeanDefinitionRegistrar.java

```java
package config.registrar;

import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.annotation.ClassPathBeanDefinitionScanner;
import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.core.type.filter.TypeFilter;

import java.io.IOException;

public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        //自定义注册器
        //1.开启类路径bean定义扫描器，需要参数bean定义注册器BeanDefinitionRegistry，需要制定是否使用默认类型过滤器
        ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(registry,false);
        //2.添加包含性加载类型过滤器（可选，也可以设置为排除性加载类型过滤器）
        scanner.addIncludeFilter(new TypeFilter() {
            @Override
            public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
                //所有匹配全部成功，此处应该添加实际的业务判定条件
                return true;
            }
        });
        //设置扫描路径
        scanner.scan("com.itheima");
    }
}

```



#### 4.4 bean初始化过程解析

##### bean初始化过程解析

init-medthod--@PostContruct-（注解在方法上）---> 在java bean 对象执行初始的时候执行里面的 某个方法

⚫ BeanFactoryPostProcessor 

​    在所有spring ioc 容器启动 java 对象前执行这个操作----> 打印你所需要日志；将常量信息缓存redis;增删改查 语句操作

​	◆ 作用：定义了在bean工厂对象创建后，bean对象创建前执行的动作，用于对工厂进行创建后业务处理 

​	◆ 运行时机：当前操作用于对工厂进行处理，仅运行一次 

⚫ BeanPostProcessor 

  在spring ioc 初始化的前后 进行相关操作--->日志；增删改查

​	◆ 作用：定义了所有bean初始化前后进行的统一动作，用于对bean进行创建前业务处理与创建后业务处理 

​	◆ 运行时机：当前操作伴随着每个bean的创建过程，每次创建bean均运行该操作 



InitializingBean---->spring ioc 框架启动时候就默认加载这个bean 对象



##### SpringConfig.java

```java
@Import({MyImportBeanDefinitionRegistrar.class, MyBeanFactory.class, MyBean.class})
public class SpringConfig {
}
```

##### MyBeanFactory.java

```java
package config.postprocessor;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;

public class MyBeanFactory implements BeanFactoryPostProcessor {
    @Override
    //工厂后处理bean接口核心操作
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("bean工厂制作好了，还有什么事情需要处理");
    }
}
```

##### MyBean.java

```java
package config.postprocessor;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class MyBean implements BeanPostProcessor {
    @Override
    //所有bean初始化前置操作
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("bean之前巴拉巴拉");
        System.out.println(beanName);
        return bean;
    }

    @Override
    //所有bean初始化后置操作
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("bean之后巴拉巴拉");
        return bean;
    }
}
```

### 5. 容器中注册bean对象形式

#### 1）包扫描 + 组件标注注解

- @Component @Controller @Service @Repository 
- @ComponentScan("根包名称")  //com.itheima

#### 2）@Bean

- 一般在@Configuration 配置类中注册bean对象
- 多用于注册三方框架中的对象

#### 3）spring提供的FactoryBean

- FactoryBean 是工厂类接口, 用户通过实现该接口定制实例化 bean 对象

- 实现Srping提供的FactoryBean, 并保证实现类对象可以注册到bean容器中

#### 4）@Import

- @Import(自定义类.class), @Import({类1.class,  类2.class, 批量注册类.class,  注册器类.class})

- 批量注册类： 实现ImportSelector接口
- 注册器类：实现ImportBeanDefinitionRegistrar接口



### 6. 注解整合注意事项 (必看)

**目标：**将mybatis javabean别名扫描路径 和  dao 扫描路径配置到jdbc.properties文件中

**但是：**课上 @Bean 注册 MapperScannerConfigurer存在如下问题：

- ```java
  //问题：
  MapperScannerConfigurer实现了BeanDefinitionRegistryPostProcessor使得spring容器会优先处理该bean的注册,导致当前类MyBatisConfig 还没有注入属性(@Value 未工作), 所以 下方 getMapperScannerConfigurer() 方法执行时, 所有@Value 修饰的属性都为空
  ```

- ```java
  //解决办法：
  1. 使用 @MapperScan("com.itheima.dao") 注解替代 @Bean 注册 MapperScannerConfigurer 对象
  2. mybatis-spring 1.3.1 及以下版本不支持 @MapperScan("${mybatis.basePackage}") 这种写法, 升级2.0.3及以上版本即可
  ```



#### pom.xml

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <!-- <version>1.3.1</version>-->
    <version>2.0.3</version>
</dependency>
```

#### jdbc.properties

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring_db
jdbc.username=root
jdbc.password=root

##Mybatis 相关配置
mybatis.typeAliasesPackage=com.itheima.domain
mybatis.basePackage=com.itheima.dao
```

#### MyBatisConfig.java

```java
package com.itheima.config;

import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.mybatis.spring.mapper.MapperScannerConfigurer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import javax.sql.DataSource;

/**
 * 课上 @Bean 注册 MapperScannerConfigurer存在如下问题：
 * 问题：MapperScannerConfigurer实现了BeanDefinitionRegistryPostProcessor使得spring容器会优先处理该bean的注册,导致当前类MyBatisConfig 还
 *      没有注入属性(@Value 未工作), 所以 下方 getMapperScannerConfigurer() 方法执行时, 所有@Value 修饰的属性都为空
 *
 * 解决办法：
 *      1. 使用 @MapperScan("com.itheima.dao") 注解替代 @Bean 注册 MapperScannerConfigurer 对象
 *      2. mybatis-spring 1.3.1 及以下版本不支持 @MapperScan("${mybatis.basePackage}") 这种写法, 升级2.0.3及以上版本即可
 */


@MapperScan("${mybatis.basePackage}")
//@MapperScan("com.itheima.dao")
public class MyBatisConfig {


    @Value("${mybatis.typeAliasesPackage}")
    private String typeAliasesPackage;

    @Bean
//    public SqlSessionFactoryBean getSqlSessionFactoryBean(@Qualifier("druidDataSource") DataSource dataSource) throws IOException {
    public SqlSessionFactoryBean getSqlSessionFactoryBean(@Autowired DataSource druidDataSource){

        System.out.println("typeAliasesPackage : " + typeAliasesPackage);

        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
//        sessionFactory.setTypeAliasesPackage("com.itheima.domain");
        sessionFactory.setTypeAliasesPackage(typeAliasesPackage);
        sessionFactory.setDataSource(druidDataSource);

        /*加载mybatis核心配置文件: 日志 + 分页 */
        sessionFactory.setConfigLocation(new PathMatchingResourcePatternResolver().getResource("classpath:mybatis-page.xml"));

        /* 方式二 配置分页*/
        /*Interceptor pageInterceptor = new PageInterceptor();
        Properties props = new Properties();
        props.setProperty("reasonable", "false");
        props.setProperty("params", "count=countSql");
        pageInterceptor.setProperties(props);
        Interceptor[] plugins = new Interceptor[]{pageInterceptor};
        sessionFactory.setPlugins( plugins );*/

        /*加载xml配置文件*/
//        sessionFactory.setMapperLocations( new PathMatchingResourcePatternResolver().getResources("classpath:mapper/**/*.xml") );

        return sessionFactory;
    }


    /*@Bean
    public MapperScannerConfigurer getMapperScannerConfigurer(){
        System.out.println("basePackage : " + basePackage);
        MapperScannerConfigurer msc = new MapperScannerConfigurer();
//        msc.setBasePackage("com.itheima.dao");
        msc.setBasePackage(basePackage);
        msc.setProcessPropertyPlaceHolders(true);
        return msc;
    }*/

}
```

