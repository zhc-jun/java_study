## SpringBoot-day02

### **学习目标**：

​    **1）**能说出 springboot的Condition的作用？常见的Condition？以及他们的判断条件？

​             **答：**Condition 是在Spring 4.0 增加的条件判断功能, 根据条件选择性控制类的对象是否注册到IOC容器中。

​             **答：**@ConditionalOnMissingBean : 对象已注册过, 则不注册。

​                     @ConditionalOnClass  ：判断类是否存在，存在则完成注册。

​                     @ConditionalOnProperty ：根据application.properties 中key value 与注解中一致，则完成注册。 

​    **2）**能够根据pom.xml切换springboot项目不同的内置容器（tomcat，jetty, Undertow）。

​    **3）**知道在springboot中，@Enable * 类的注解底层都是基于@Import注解来指定某个类被spring处理, 注册到IOC容器

​    **4）**知道@SpringBootApplication 内部的 @EnableAutoConfiguration 内部的@Import({AutoConfigurationImportSelector.class}) 实际上是一个spring的 ImportSelector接口的实现类, 同时它会将<font color=red>所有jar包中的META-INF/spring.factories配置的 "包名.类名" 装载到String[] 中</font>,   稍后一并注册到 IOC容器中。

​    **5）**可以利用@SpringBootApplication内部机制，仿写自己的redis-starter模块

​    **6）**知道springboot有监听事件的一回事，最好结合springboot 执行流程选择合适的监听事件来加载自己想要处理的数据。

​    **7）**知道springboot admin 是基于actuator 实现的，随便点一点 server端的UI页面，看看即可。

​    **8）**知道springboot 既可以 jar 启动 也可以war部署到web 容器中。



### 1.  SpringBoot 原理分析  



源码注解→编译时注解→运行时注解】

#### 1.1 SpringBoot 自动配置

##### Condition

Condition 是在Spring 4.0 增加的条件判断功能，通过这个可以功能可以实现选择性的创建Bean 操作。



##### 案例一：需求

在 Spring 的 IOC 容器中有一个 User 的 Bean，现要求： 

1. 导入Jedis坐标后，加载该Bean，没导入，则不加载。 

###### ClassCondition.java

```java
package com.itheima.springbootcondition.condtion;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

import java.util.Map;

public class ClassCondition implements Condition {
    /**
     *
     * @param context 上下文对象。用于获取环境，IOC容器，ClassLoader对象
     * @param metadata 注解元对象。 可以用于获取注解定义的属性值
     * @return
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        
        //1.需求： 导入Jedis坐标后创建Bean
        //思路：判断redis.clients.jedis.Jedis.class文件是否存在
        boolean flag = true;
        try {
            Class<?> cls = Class.forName("redis.clients.jedis.Jedis");
        } catch (ClassNotFoundException e) {
            flag = false;
        }
        return flag;
   }
       
}
```

###### UserConfig.java

```java
package com.itheima.springbootcondition.config;

import com.itheima.springbootcondition.condtion.ClassCondition;
import com.itheima.springbootcondition.condtion.ConditionOnClass;
import com.itheima.springbootcondition.domain.User;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

@Configuration
public class UserConfig {

    @Bean
    @Conditional(ClassCondition.class)
    public User user(){
        return new User();
    }

}
```

###### SpringbootConditionApplication.java

```java
package com.itheima.springbootcondition;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class SpringbootConditionApplication {

    public static void main(String[] args) {
        //启动SpringBoot的应用，返回Spring的IOC容器
        ConfigurableApplicationContext context = SpringApplication.run(SpringbootConditionApplication.class, args);

        Object user = context.getBean("user");
        System.out.println(user);

    }

}
```

###### User.java

```java
package com.itheima.springbootcondition.domain;

public class User {
}
```

###### pom.xml

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
    <artifactId>springboot-condition</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-condition</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!--排除tomcat依赖-->
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <!--引入jetty的依赖-->
        <dependency>
            <artifactId>spring-boot-starter-jetty</artifactId>
            <groupId>org.springframework.boot</groupId>
        </dependency>


       <!-- <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.4</version>
        </dependency>-->
<!--

        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
-->

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



##### 案例二：需求

1. 在案例一基础上，将类的判断定义为动态的。判断哪个字节码文件存在可以动态指定。
2. 思路：将写死的 “**包名.类名**” 暴露出去，交给开发人员自行设定。



###### ConditionOnClass.java

- 使用@Conditional(ClassCondition.class)修饰自己定义的注解，我们的注解也会有@Conditional()同样的功能
- @Target({ElementType.TYPE, ElementType.METHOD})：可以修饰类和方法上
- @Retention(RetentionPolicy.RUNTIME)：  用于描述一个注解存在的生命周期, 注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在 
-  @Documented 注解较少用，简单了解即可。
  该注解主要判断是否可以生成到 API文档中 ==》即生成API文档的时 检验 

```java
package com.itheima.springbootcondition.condtion;

import org.springframework.context.annotation.Conditional;
import java.lang.annotation.*;

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ClassCondition.class)
public @interface ConditionOnClass {
    String[] value();
}

```

###### UserConfig.java

```java
package com.itheima.springbootcondition.config;

import com.itheima.springbootcondition.condtion.ConditionOnClass;
import com.itheima.springbootcondition.domain.User;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

@Configuration
public class UserConfig {

    @Bean
    @ConditionOnClass("com.alibaba.fastjson.JSON")
    public User user(){
        return new User();
    }

}

```

###### ClassCondition.java

```java
package com.itheima.springbootcondition.condtion;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

import java.util.Map;

public class ClassCondition implements Condition {
    /**
     *
     * @param context 上下文对象。用于获取环境，IOC容器，ClassLoader对象
     * @param metadata 注解元对象。 可以用于获取注解定义的属性值
     * @return
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
       
        //2.需求： 导入通过注解属性值value指定坐标后创建Bean
        //获取注解属性值  value
        Map<String, Object> map = metadata.getAnnotationAttributes(ConditionOnClass.class.getName());
        //System.out.println(map);
        //属性名value
        String[] value = (String[]) map.get("value");

        boolean flag = true;
        try {
            for (String className : value) {
                Class<?> cls = Class.forName(className);
            }
        } catch (ClassNotFoundException e) {
            flag = false;
        }
        return flag;
    }
    
}
```



##### 案例三：需求

1. 使用@ConditionalOnProperty() 注解，当application.properties 配置文件中key = value 存在时，就会创建对应的对象。
2. springboot也提供了 @ConditionalOnClass(name = "com.alibaba.fastjson.JSON")

###### UserConfig.java

```java
package com.itheima.springbootcondition.config;

import com.itheima.springbootcondition.condtion.ClassCondition;
import com.itheima.springbootcondition.condtion.ConditionOnClass;
import com.itheima.springbootcondition.domain.User;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

@Configuration
public class UserConfig {

    @Bean
    /**
     * springboot提供的
     */
    @ConditionalOnClass(name = "com.alibaba.fastjson.JSON")
    public User user(){
        return new User();
    }

    @Bean
    @ConditionalOnProperty(name = "itcast",havingValue = "itheima")
    public User user2(){
        return new User();
    }

}
```

###### application.properties

```properties
itcast=itheima
```



##### Condition – 小结

​	⚫ 自定义条件： 

​		① 定义条件类：自定义类实现Condition接口，重写matches 方法，在 matches 方法中进行逻辑判断，返回 boolean值。 matches 方法两个参数：

​			    • context：上下文对象，可以获取属性值，获取类加载器，获取BeanFactory等。 

​				• metadata：元数据对象，用于获取注解属性。 

​		② 判断条件： 在初始化Bean时，使用 @Conditional(条件类.class)注解

​	⚫ SpringBoot 提供的常用条件注解： 

​				• ConditionalOnProperty：判断配置文件中是否有对应属性和值才初始化Bean 

​				• ConditionalOnClass：判断环境中是否有对应字节码文件才初始化Bean 

​				• ConditionalOnMissingBean：判断环境中没有对应Bean才初始化Bean





### 2. 切换内置web服务器

SpringBoot的web环境中默认使用tomcat作为内置服务器，其实SpringBoot提供了4中内置服务器供我们选择，我们可 以很方便的进行切换。

   1）Jetty   2）Netty（暂不支持）    3）Tomcat     4）Undertow



#### Tomcat

pom.xml

- 该jar传递依赖了 tomcat jar

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```



#### Jetty

pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

#### Undertow

pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- 移除掉默认支持的 Tomcat -->
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- 添加 Undertow 容器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```



### 3.  @Enable*注解

SpringBoot中提供了很多Enable开头的注解，这些注解都是用于动态启用某些功能的。而其底层原理是使用**@Import**注 解导入一些配置类，实现Bean的动态加载。

**思考**： SpringBoot 工程是否可以直接获取jar包中定义的Bean？答：需要采取一定的手段才可以。

<font color=red>默认情况下, springboot 会自动扫描 @SpringBootApplication注解修饰的类所在的包以及子包下所有被@Import @Configration @Service @Component @Controller @Repository修饰的类。</font>

#### EnableUser.java

- 自定义注解

```java
package com.itheima.config;

import org.springframework.context.annotation.Import;

import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(UserConfig.class)
public @interface EnableUser {
}
```

#### UserConfig.java

- 该篇省略 User.java  和 Role.java 两个Javabean 代码展示

```java
package com.itheima.config;

import com.itheima.domain.Role;
import com.itheima.domain.User;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

//@Configuration
public class UserConfig {

    @Bean
    public User user() {
        return new User();
    }

    @Bean
    public Role role() {
        return new Role();
    }
}
```

#### SpringbootEnableApplication.java

- @EnableUser 注解底层也就是封装@Import注解来完成配置类的注入IOC

```java
package com.itheima.springbootenable;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.Bean;
import redis.clients.jedis.Jedis;

import java.util.Map;

/**
 * @ComponentScan 扫描范围：当前引导类所在包及其子包
 *
 * com.itheima.springbootenable
 * com.itheima.config
 * //1.使用@ComponentScan扫描com.itheima.config包
 * //2.可以使用@Import注解，加载类。这些类都会被Spring创建，并放入IOC容器
 * //3.可以对Import注解进行封装。
 */

//@ComponentScan("com.itheima.config")
//@Import(UserConfig.class)
@EnableUser
@SpringBootApplication
public class SpringbootEnableApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringbootEnableApplication.class, args);

        Object user = context.getBean("user");
        System.out.println(user);

    }
}
```



### 4. @Import注解

@Enable*底层依赖于@Import注解导入一些类，使用@Import导入的类会被Spring加载到IOC容器中。而@Import提供4中用 法：

​	    ① 导入Bean 

​		② 导入配置类 

​		③ 导入 ImportSelector 实现类。一般用于加载配置文件中的类 

​		④ 导入 ImportBeanDefinitionRegistrar 实现类。



#### SpringbootEnableApplication.java

```java
package com.itheima.springbootenable;

import com.itheima.config.MyImportBeanDefinitionRegistrar;
import com.itheima.domain.Role;
import com.itheima.domain.User;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Import;

/**
 * Import4中用法：
 *  1. 导入Bean
 *  2. 导入配置类
 *  3. 导入ImportSelector的实现类。
 *  4. 导入ImportBeanDefinitionRegistrar实现类
 */

//@Import(User.class)
//@Import(UserConfig.class)
//@Import(MyImportSelector.class)
@Import({MyImportBeanDefinitionRegistrar.class})

@SpringBootApplication
public class SpringbootEnableApplication {

    public static void main(String[] args) {
    
        ConfigurableApplicationContext context = SpringApplication.run(SpringbootEnableApplication.class, args);

        User user = context.getBean(User.class);
        System.out.println(user);

    }
}
```

#### UserConfig.java

```java
package com.itheima.config;

        import com.itheima.domain.Role;
        import com.itheima.domain.User;
        import org.springframework.context.annotation.Bean;
        import org.springframework.context.annotation.Configuration;

//@Configuration
public class UserConfig {

    @Bean
    public User user() {
        return new User();
    }

    @Bean
    public Role role() {
        return new Role();
    }
}
```

#### MyImportSelector.java

```java
package com.itheima.config;

import org.springframework.context.annotation.ImportSelector;
import org.springframework.core.type.AnnotationMetadata;

public class MyImportSelector implements ImportSelector {

    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"com.itheima.domain.User", "com.itheima.domain.Role"};
    }
}
```

#### MyImportBeanDefinitionRegistrar.java

```java
package com.itheima.config;

import com.itheima.domain.User;
import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
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

        //1.方式一
        /*AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(User.class).getBeanDefinition();
        registry.registerBeanDefinition("user", beanDefinition);*/


        //2.方式二-批量注册
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
        scanner.scan("com.itheima.domain");

    }
}
```



### 5. @EnableAutoConfiguration

​	⚫ @EnableAutoConfiguration 注解内部使用 @Import(AutoConfigurationImportSelector.class)来加载配置类。
​	⚫ 配置文件位置：META-INF/spring.factories，该配置文件中定义了大量的配置类，当 SpringBoot 应用启动时，会自动加载 这些配置类，初始化Bean 

​	⚫ 并不是所有的Bean都会被初始化，在配置类中使用Condition来加载满足条件的Bean

源码

#### EnableAutoConfiguration.java

```java 
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.boot.autoconfigure;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.context.annotation.Import;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}

```



#### AutoConfigurationImportSelector.java

- 如下代码为该类一个方法，它会加载所有jar包中META-INF/spring.factories文件中所有配置好的 **“包名.类名”** 注册到IOC容器中。
- 日后我们可以基于此机制，制作自己的starter jar包

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
```

#### spring.factories

- META-INF/spring.factories

- 预览, springboot的一些 带有 autoconfigure的jar包中, 都存在这类文件，完成自动装配

```properties
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener

..........此处省略很多类似上方的配置

```



### 6. 自定义starter

#### 案例一：需求

自定义redis-starter。要求当导入redis坐标时，SpringBoot自动创建Jedis的Bean。

​	① 创建 redis-spring-boot-starter 模块

​	② 在 redis-spring-boot-starter 模块中初始化Jedis 的 Bean。并定义META-INF/spring.factories 文件 

​	③ 在测试模块中引入自定义的redis-starter 依赖，测试获取 Jedis 的Bean，操作 redis。 

<font color=red>说明: @EnableAutoConfiguration 底层的@Import({AutoConfigurationImportSelector.class}) 底层会加载所有jar包中: META-INF/spring.factories文件中所有配置好的 **“包名.类名”** 注册到IOC容器中</font>



#### redis-spring-boot-starter模块

- 该模块直接配置starter需要做的工作

##### RedisAutoConfiguration.java

```java
package com.itheima.redis.config;

import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import redis.clients.jedis.Jedis;

@Configuration
@EnableConfigurationProperties(RedisProperties.class)
@ConditionalOnClass(Jedis.class)
public class RedisAutoConfiguration {

    /**
     * 提供Jedis的bean
     */
    @Bean
    @ConditionalOnMissingBean(name = "jedis")
    public Jedis jedis(RedisProperties redisProperties) {
        System.out.println("RedisAutoConfiguration....");
        return new Jedis(redisProperties.getHost(), redisProperties.getPort());
    }
}

```

##### RedisProperties.java

- 该类可以从application.properties/yml 文件中读取配置并给属性赋值

```java
package com.itheima.redis.config;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "redis")
public class RedisProperties {

    private String host= "localhost";
    private int port = 6379;


    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }

    public int getPort() {
        return port;
    }

    public void setPort(int port) {
        this.port = port;
    }
}
```

##### spring.factories

- 项目resources\META-INF\spring.factories

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.itheima.redis.config.RedisAutoConfiguration
```

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
    <artifactId>redis-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>redis-spring-boot-starter</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <!--引入jedis依赖-->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>

    </dependencies>


</project>
```



#### springboot-enable模块

- 该模块用于测试 redis-spring-boot-starter

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
    <artifactId>springboot-enable</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-enable</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
    
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <!--自定义的redis的starter-->
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>redis-spring-boot-starter</artifactId>
            <version>0.0.1-SNAPSHOT</version>
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

##### application.properties

```properties
redis.host=localhost
redis.port=6379
```

##### SpringbootEnableApplication.java

```java
package com.itheima.springbootenable;

import com.itheima.config.MyImportBeanDefinitionRegistrar;
import com.itheima.domain.Role;
import com.itheima.domain.User;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Import;
import redis.clients.jedis.Jedis;

import java.util.Map;

@SpringBootApplication
public class SpringbootEnableApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringbootEnableApplication.class, args);

        Jedis jedis = context.getBean(Jedis.class);
        System.out.println(jedis);

        jedis.set("name","itcast");

        String name = jedis.get("name");
        System.out.println(name);

    }

}
```





### 7. 事件监听

#### MyApplicationRunner.java

-  如果需要在容器启动的时候追加一些配置数据。比如读取配置文件，数据库连接之类的数据。
- SpringBoot给我们提供了两个接口来帮助我们实现这种需求。这两个接口分别为CommandLineRunner和ApplicationRunner。**他们的执行时机为SpringIOC容器启动完成的时候。** 

```java
package com.itheima.springbootlistener.listener;

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

import java.util.Arrays;

/**
 * 当项目启动后执行run方法。
 */
@Component
public class MyApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("ApplicationRunner...run");
        System.out.println(Arrays.asList(args.getSourceArgs()));
    }
}

```

#### MyCommandLineRunner.java

-  如果需要在容器启动的时候追加一些配置数据。比如读取配置文件，数据库连接之类的数据。
- SpringBoot给我们提供了两个接口来帮助我们实现这种需求。这两个接口分别为CommandLineRunner和ApplicationRunner。**他们的执行时机为SpringIOC容器启动完成的时候。** 

```java
package com.itheima.springbootlistener.listener;

import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

import java.util.Arrays;

/**
 * 命令执行时参数监听
 */
@Component
public class MyCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        System.out.println("CommandLineRunner...run");
        System.out.println(Arrays.asList(args));
    }
}
```

#### MyApplicationContextInitializer.java

- 需要在resources/META-INF/spring.factories 配置

```java
package com.itheima.springbootlistener.listener;

import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.stereotype.Component;

/**
 * ApplicationContextInitializer是Spring留出来允许我们在上下文刷新之前做自定义操作的钩子，
 * 若我们有需求想要深度整合Spring上下文，借助它不乏是一个非常好的实现。
 */
public class MyApplicationContextInitializer implements ApplicationContextInitializer {

    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {

        System.out.println("ApplicationContextInitializer....initialize");
    }

}
```

#### MySpringApplicationRunListener.java

- 需要在resources/META-INF/spring.factories 配置
- SpringApplicationRunListener 接口的作用主要就是在Spring Boot 启动初始化的过程中可以通过SpringApplicationRunListener接口回调来让用户在启动的各个流程中可以加入自己的逻辑。
   Spring Boot启动过程的关键事件（按照触发顺序）包括：
  1. 开始启动
  2. Environment构建完成
  3. ApplicationContext构建完成
  4. ApplicationContext完成加载
  5. ApplicationContext完成刷新并启动
  6. 启动完成
  7. 启动失败

```java
package com.itheima.springbootlistener.listener;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringApplicationRunListener;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.stereotype.Component;

public class MySpringApplicationRunListener implements SpringApplicationRunListener {

    /**
     * 此构造springboot规定固定两个如下类型形参
     * @param application
     * @param args
     */
    public MySpringApplicationRunListener(SpringApplication application, String[] args) {
    }

    @Override
    public void starting() {
        System.out.println("starting...项目启动中");
    }

    @Override
    public void environmentPrepared(ConfigurableEnvironment environment) {
        System.out.println("environmentPrepared...环境对象开始准备");
    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("contextPrepared...上下文对象开始准备");
    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("contextLoaded...上下文对象开始加载");
    }

    @Override
    public void started(ConfigurableApplicationContext context) {
        System.out.println("started...上下文对象加载完成");
    }

    @Override
    public void running(ConfigurableApplicationContext context) {
        System.out.println("running...项目启动完成，开始运行");
    }

    @Override
    public void failed(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("failed...项目启动失败");
    }

}
```

#### spring.factories

- resources/META-INF/spring.factories 

```properties
org.springframework.context.ApplicationContextInitializer=com.itheima.springbootlistener.listener.MyApplicationContextInitializer

org.springframework.boot.SpringApplicationRunListener=com.itheima.springbootlistener.listener.MySpringApplicationRunListener
```

#### SpringbootListenerApplication.java

```java
package com.itheima.springbootlistener;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;

@SpringBootApplication
public class SpringbootListenerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootListenerApplication.class, args);
    }

}
```



### 8. 流程分析

多看视频，知道自己实现springboot功能在哪一步被处理即可。

#### 第一步骤：

1）new SpringApplication() 创建一个应用对象

2）initialize() 初始化的过程会将 META-INF/spring.factories 中所有类加载进内存，并创建工厂对象。

3）其他事项与我们无关，感兴趣自行了解

#### 第二步骤：

4）new StopWatch() 用来计时启动完成总耗时时间

5）加载事件接口SpringApplicationRunListener所有的实现类

6）循环遍历SpringApplicationRunListener实现类, 并调用listeners.starting();

7）创建环境对象 ConfigurableEnvironment

8）循环遍历SpringApplicationRunListener实现类, 并调用this.prepareEnvironment(listeners, applicationArguments)

- 我们可以获取所有的参数，比如：application.properties 或者 命令行中的参数。

9）加载resources/banner.txt 打印到控制台

10）初始化Spring上下文对象 this.createApplicationContext();

12）this.prepareContext(）内部会**顺次调用如下事件回调方法**：

- ApplicationContextInitializer接口的initialize() 并得到spring 上下文对象
- 循环遍历SpringApplicationRunListener实现类, 并调用contextPrepared()
- 循环遍历SpringApplicationRunListener实现类, 并调用contextLoaded()
- 如上三个方法的回调都可以得到: Spring 上线文对象 ConfigurableApplicationContext

11）this.refreshContext(context); 后, 项目中所有需要创建的bean都已完成（最耗时）

12）stopWatch.stop(); 停止计时

13）循环遍历SpringApplicationRunListener实现类, 并调用listeners.started(context);

14）this.callRunners(context, applicationArguments); 回调ApplicationRunner 和 CommandLineRunner 两个接口实现类重写的run方法

<font color=red>注意：run() 为我们提供了参数对象，该对象只装载了我们在命令行设置的参数，如果有需要可以继续追加到此对象中</font>

15）循环遍历SpringApplicationRunListener实现类, 并调用 listeners.running(context); 

至此，所有课上所讲事件监听回调方法都已执行完毕~~~启动结束



#### 总结：

- ApplicationRunner 和 CommandLineRunner 两个接口都可以在Spring初始化之后，追加应用命令行参数，二者任选其一使用即可，一般我们都不需要追加任何东西。
- SpringApplicationRunListener接口是监听springboot 执行流程最完善的机制，它可以帮我们更好的解决问题 <font color=red>（推荐使用）</font>
- ApplicationContextInitializer接口可以被SpringApplicationRunListener 所替代，所以也不推荐使用。
- SpringApplicationRunListener接口中每一个方法都是执行流程中，每一步的关键点，我们可以将该接口中的方法多看看。
- 最后:  清晰了springboot 先后执行流程
  - 才能更好的结合SpringApplicationRunListener监听机制找准时机，完成自己的挂载需求。
  - 这一点跟我们在学Vue声明周期时，思想很像，当时我们也是利用vue生命周期流程中某一个步骤，来挂载我们展示**分页**数据。



### 9. 监控actuator

#### 9.1 SpringBoot 监控概述

SpringBoot自带监控功能Actuator，可以帮助实现对程序内部运行情况监控，比如监控状况、Bean加载情况、配置属性 、日志信息等。

#### 9.2 SpringBoot 监控使用

① 导入依赖坐标

```xml
<dependency> 
 <groupId>org.springframework.boot</groupId> 
 <artifactId>spring-boot-starter-actuator</artifactId> </dependency> 
```

② 访问http://localhost:8080/acruator

|      路径       |                             描述                             |
| :-------------: | :----------------------------------------------------------: |
|     /beans      |        描述应用程序上下文里全部的Bean，以及它们的关系        |
|      /env       |                       获取全部环境属性                       |
|   /env/{name}   |                 根据名称获取特定的环境属性值                 |
|     /health     | 报告应用程序的健康指标，这些值由HealthIndicator的实现类提供  |
|      /info      |     获取应用程序的定制信息，这些信息由info打头的属性提供     |
|    /mappings    | 描述全部的URI路径，以及它们和控制器(包含Actuator端点)的映射关系 |
|    /metrics     |     报告各种应用程序度量信息，比如内存用量和HTTP请求计数     |
| /metrics/{name} |                 报告指定名称的应用程序度量值                 |
|     /trace      |         提供基本的HTTP请求跟踪信息(时间戳、HTTP头等)         |



#### 9.3 Spring Boot Admin

​	⚫ Spring Boot Admin是一个开源社区项目，用于管理和监控SpringBoot应用程序。 

​	⚫ Spring Boot Admin 有两个角色，客户端(Client)和服务端(Server)。 

​	⚫ 应用程序作为Spring Boot Admin Client向为Spring Boot Admin Server注册 

​	⚫ Spring Boot Admin Server 的UI界面将Spring Boot Admin Client的ActuatorEndpoint上的一些监控信息。



<font color=red> 通过使用 Admin-UI 改进 Actuator 的缺点，因为 Admin-UI 基于 Actuator 实现提供了可视化监控管理界面 </font>



##### admin-server： 

​	① 创建 admin-server 模块 

​	② 导入依赖坐标admin-starter-server 

​	③ 在引导类上启用监控功能@EnableAdminServer

###### pom.xml

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
    <artifactId>springboot-admin-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-admin-server</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-boot-admin.version>2.1.5</spring-boot-admin.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>de.codecentric</groupId>
                <artifactId>spring-boot-admin-dependencies</artifactId>
                <version>${spring-boot-admin.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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

###### SpringbootAdminServerApplication.java

```java
package com.itheima.springbootadminserver;

import de.codecentric.boot.admin.server.config.EnableAdminServer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@EnableAdminServer
@SpringBootApplication
public class SpringbootAdminServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootAdminServerApplication.class, args);
    }

}
```

###### application.properties

```properties
server.port=9000
```



##### admin-client： 

​	① 创建 admin-client 模块 

​	② 导入依赖坐标admin-starter-client 

​	③ 配置相关信息：server地址等 

​	④ 启动server和client服务，访问server

###### pom.xml

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
    <artifactId>springboot-admin-client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-admin-client</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-boot-admin.version>2.1.5</spring-boot-admin.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- 传递依赖了 actutor -->
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>de.codecentric</groupId>
                <artifactId>spring-boot-admin-dependencies</artifactId>
                <version>${spring-boot-admin.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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

###### SpringbootAdminClientApplication.java

```java
package com.itheima.springbootadminclient;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootAdminClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootAdminClientApplication.class, args);
    }

}
```

###### UserController.java

```java
package com.itheima.springbootadminclient;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/user")
public class UserController {


    @RequestMapping("/findAll")
    public String findAll(){
        return "success";
    }
}
```

###### application.properties

```properties

# 执行admin.server地址
spring.boot.admin.client.url=http://localhost:9000

# 开启客户端Actuator的权限
management.endpoint.health.show-details=always
management.endpoints.web.exposure.include=*

```



### 10. SpringBoot 项目部署

SpringBoot 项目开发完毕后，支持两种方式部署到服务器： 

​	① jar包(官方推荐) : java -jar  xxx.jar

​	② war包: 需要自己手动传到Tomcat 安装目录webapps/下

#### pom.xml

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
    <artifactId>springboot-deploy</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <!-- 成果物类型 -->
    <packaging>war</packaging>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

    </dependencies>

</project>
```

