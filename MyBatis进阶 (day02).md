# MyBatis进阶-（全天重点）

### 1. MyBatis 接口代理方式实现 Dao 层

#### 1.1 接口代理方式-实现规则

- 传统方式实现 Dao 层，我们既要写接口，还要写实现类。而MyBatis 框架可以帮助我们省略编写Dao 层接 口实现类的步骤。程序员只需要编写接口，由MyBatis 框架根据接口的定义来创建该接口的动态代理对象。

- <font color=red>实现规则:</font> 

  > 1). 映射配置文件中的名称空间必须和Dao 层接口的全类名相同。 
  >
  > 2). 映射配置文件中的增删改查标签的id 属性必须和 Dao 层接口的方法名相同。 
  >
  > 3). 映射配置文件中的增删改查标签的parameterType 属性必须和 Dao 层接口方法的参数相同。 
  >
  > 4). 映射配置文件中的增删改查标签的resultType 属性必须和 Dao 层接口方法的返回值相同。



#### 1.2 接口代理方式-代码实现

1）删除 mapper 层接口的实现类。 

2）修改映射配置文件。 

3）修改 service 层接口的实现类，采用接口代理方式实现功能。



##### **StudentMapper.java**

```java
package com.itheima.mapper;

import com.itheima.bean.Student;

import java.util.List;

/*
    持久层接口
 */
public interface StudentMapper {
    //查询全部
    public abstract List<Student> selectAll();

    //根据id查询
    public abstract Student selectById(Integer id);

    //新增数据
    public abstract Integer insert(Student stu);

    //修改数据
    public abstract Integer update(Student stu);

    //删除数据
    public abstract Integer delete(Integer id);

    //多条件查询
    public abstract List<Student> selectCondition(Student stu);

    //根据多个id查询
    public abstract List<Student> selectByIds(List<Integer> ids);
}

```

##### **StudentMapper.xml**

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
<mapper namespace="com.itheima.mapper.StudentMapper">

    <sql id="select" >SELECT * FROM student</sql>

    <!--
        select：查询功能的标签
        id属性：唯一标识
        resultType属性：指定结果映射对象类型
        parameterType属性：指定参数映射对象类型
    -->
    <select id="selectAll" resultType="student">
        <include refid="select"/>
    </select>

    <select id="selectById" resultType="student" parameterType="int">
        <include refid="select"/> WHERE id = #{id}
    </select>

    <insert id="insert" parameterType="student">
        INSERT INTO student VALUES (#{id},#{name},#{age})
    </insert>

    <update id="update" parameterType="student">
        UPDATE student SET name = #{name},age = #{age} WHERE id = #{id}
    </update>

    <delete id="delete" parameterType="int">
        DELETE FROM student WHERE id = #{id}
    </delete>
</mapper>
```

##### 测试代码

- 如下代码是展示如何调用Mapper 中的方法, 运行参考当天资料中代码

```java
package com.itheima.service.impl;

import com.itheima.bean.Student;
import com.itheima.mapper.StudentMapper;
import com.itheima.service.StudentService;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;
/*
    业务层实现类
 */
public class StudentServiceImpl implements StudentService {

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

            //4.获取StudentMapper接口的实现类对象
            StudentMapper mapper = sqlSession.getMapper(StudentMapper.class); // StudentMapper mapper = new StudentMapperImpl();

            //5.通过实现类对象调用方法，接收结果
            list = mapper.selectAll();

        } catch (Exception e) {

        } finally {
            //6.释放资源
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

        //7.返回结果
        return list;
    }

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

            //4.获取StudentMapper接口的实现类对象
            StudentMapper mapper = sqlSession.getMapper(StudentMapper.class); // StudentMapper mapper = new StudentMapperImpl();

            //5.通过实现类对象调用方法，接收结果
            stu = mapper.selectById(id);

        } catch (Exception e) {

        } finally {
            //6.释放资源
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

        //7.返回结果
        return stu;
    }

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

            //4.获取StudentMapper接口的实现类对象
            StudentMapper mapper = sqlSession.getMapper(StudentMapper.class); // StudentMapper mapper = new StudentMapperImpl();

            //5.通过实现类对象调用方法，接收结果
            result = mapper.insert(stu);

        } catch (Exception e) {

        } finally {
            //6.释放资源
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

        //7.返回结果
        return result;
    }

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

            //4.获取StudentMapper接口的实现类对象
            StudentMapper mapper = sqlSession.getMapper(StudentMapper.class); // StudentMapper mapper = new StudentMapperImpl();

            //5.通过实现类对象调用方法，接收结果
            result = mapper.update(stu);

        } catch (Exception e) {

        } finally {
            //6.释放资源
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

        //7.返回结果
        return result;
    }

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

            //4.获取StudentMapper接口的实现类对象
            StudentMapper mapper = sqlSession.getMapper(StudentMapper.class); // StudentMapper mapper = new StudentMapperImpl();

            //5.通过实现类对象调用方法，接收结果
            result = mapper.delete(id);

        } catch (Exception e) {

        } finally {
            //6.释放资源
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

        //7.返回结果
        return result;
    }
}

```

#### 1.3 接口代理方式-源码分析

**分析动态代理对象如何生成的？** 

> 通过动态代理开发模式，我们只编写一个接口，不写实现类，我们通过getMapper() 方法最终获取到 org.apache.ibatis.binding.MapperProxy代理对象，然后执行功能，而这个代理对象正是 MyBatis使用了 JDK 的动态代理技术，帮助我们生成了代理实现类对象。从而可以进行相关持久化操作。

**分析方法是如何执行的？** 

> 动态代理实现类对象在执行方法的时候最终调用了mapperMethod.execute() 方法，这个方法中通过 switch 语句根据操作类型来判断是新增、修改、删除、查询操作，最后一步回到了MyBatis 最原生的 SqlSession方式来执行增删改查。



#### 1.4 接口代理方式小结

- **原生 **: Dao ----->  DaoImpl

- **接口**: Mapper  ------- > xxMapper.xml

- 接口代理方式可以让我们只编写接口即可，而实现类对象由MyBatis 生成。 

- 实现规则 

  > 1). 映射配置文件中的名称空间必须和Dao 层接口的全类名相同。 
  >
  > 2). 映射配置文件中的增删改查标签的id 属性必须和 Dao 层接口的方法名相同。 
  >
  > 3). 映射配置文件中的增删改查标签的parameterType 属性必须和 Dao 层接口方法的参数相同。 
  >
  > 4). 映射配置文件中的增删改查标签的resultType 属性必须和 Dao 层接口方法的返回值相同。 

- 获取动态代理对象 

  > SqlSession 功能类中的getMapper(Mapper接口.class) 方法。

**了解一下**： 

   1）SqlSession 代表着和数据库的一次会话，用完必须关闭

   2）SqlSession 和 Connection一样都是非线程安全的，每次使用都要去获得新的对象



### 2. MyBatis 映射配置文件 – 动态 SQL

#### 2.1 动态 SQL 介绍

- MyBatis 映射配置文件中，前面我们的SQL 都是比较简单的，有些时候业务逻辑复杂时，我们的SQL 就是 动态变化的，此时在前面学习的SQL 就不能满足要求了。 

- 多条件查询

  > SELECT * FROM student WHERE id = ? AND name = ? AND age = ?
  >
  > SELECT * FROM student WHERE id = ? AND name = ?

- 动态 SQL 

  > 标签: 
  >
  > ​     <if>：条件判断标签。 
  >
  > ​     <foreach>：循环遍历标签。



#### 2.2 "if" 标签

**示例如下：**

```xml
<!-- 

<where>：条件标签。如果有动态条件，则使用该标签代替where 关键字。 
<if>：条件判断标签。
    示例如下:
    <if test=“条件判断”> 查询条件拼接 </if>
--> 

<select id="selectCondition" resultType="student" parameterType="student">
    SELECT * FROM student
    <where>
        <if test="id != null">
            id = #{id}
        </if>
        <if test="name != null">
            AND name = #{name}
        </if>
        <if test="age != null">
            AND age = #{age}
        </if>
    </where>
</select>
```

#### 2.3 “foreach”  标签

- 存在扩展

```xml
<!-- 
<foreach>：循环遍历标签。适用于多个参数或者的关系。
属性 
    collection：参数容器类型，(list-集合，array-数组)。 
    open：开始的 SQL 语句。 
    close：结束的SQL 语句。 
    item：参数变量名。 
    separator：分隔符。
-->
<select id="selectByIds" resultType="student" parameterType="list">
    SELECT * FROM student
    <where>
        <foreach collection="list" open="id IN (" close=")" item="id" separator=",">
            #{id}
        </foreach>
    </where>
</select>

<!-- 还可以这么写  -->
<select id="selectByIds" resultType="student" parameterType="list">
    SELECT * FROM student WHERE id IN 
        <foreach collection="list" open="(" close=")" item="id" separator=",">
            #{id}
        </foreach>
</select>

<!-- 如果是 selectByIds(List<Student>  stus) 这种-->
<select id="selectByIds" resultType="student" parameterType="list">
    SELECT * FROM student
    <where>
        <foreach collection="list" open="id IN (" close=")" item="stu" separator=",">
            #{stu.id}
        </foreach>
    </where>
</select>
```

#### 2.4 SQL 片段抽取

 我们可以将一些重复性的SQL 语句进行抽取，以达到复用的效果。 

- <sql>：抽取SQL 语句标签。

  > <sql id="片段唯一标识">抽取的SQL 语句</sql>

- <include>：引入SQL 片段标签。

  > <include refid="片段唯一标识"/>

**示例代码:**

<font color=red>一般我们用sql标签将字段列表进行抽取, 如下:</font>

> <sql id="select" > id, name, age, address</sql>

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
<mapper namespace="com.itheima.mapper.StudentMapper">

    <sql id="select" >SELECT * FROM student</sql>

    <!--
        select：查询功能的标签
        id属性：唯一标识
        resultType属性：指定结果映射对象类型
        parameterType属性：指定参数映射对象类型
    -->
    <select id="selectAll" resultType="student">
        <include refid="select"/>
    </select>

    <select id="selectById" resultType="student" parameterType="int">
        <include refid="select"/> WHERE id = #{id}
    </select>
</mapper>
```

#### 2.5 动态 SQL 小结

- 动态 SQL 指的就是 SQL 语句可以根据条件或者参数的不同进行动态的变化。
- <where>：条件标签。 
- <if>：条件判断的标签。 
- <foreach>：循环遍历的标签。
- <sql>：抽取SQL 片段的标签。
- <include>：引入SQL 片段的标签。

#### 2.6 (扩展) 其他标签

##### "set" 标签

- set标签和where标签可以将不合理的字符自动给处理掉，比如下方phone为最后一个赋值项，但是注意: "phone = #{phone}," 最后面有一个 逗号(,)  set 标签可以为我们自动去除掉
- 关于 "phone = #{phone},"  问题，也可以使用 “trim标签”将其处理掉

```xml
<!--  修改 用户信息  ${tableName} -->
<update id="updateUserInfo" parameterType="User">
    UPDATE t_user 
    <set>
        <if test="password != null">
            password = #{password},
        </if>
        <if test="nickname != null">
            nickname = #{nickname},
        </if>
        <if test="phone != null">
            phone = #{phone},
        </if>
    </set>
    WHERE user_id = #{user_id} 
</update>
```

##### “where” 标签

- where标签当发现if 标签都不满足条件时，不会再from 关键字后方加入 where 关键字

```xml
<!--

public List<Employee> getEmpsByConditionTrim(Employee employee);  

-->

<select id="getEmpsByConditionTrim" resultType="com.atguigu.mybatis.bean.Employee">
    select * from tbl_employee
    <where>
        <if test="id!=null">
            id=#{id} 
        </if>
        <if test="lastName!=null and lastName!='' ">
            and last_name like #{lastName}
        </if>
        <if test="email!=null and email.trim()!=''">
            and email=#{email} 
        </if>
    </where>
</select>
```

##### “trim”标签

- 参考博客：https://mp.weixin.qq.com/s/hXRdtjoaJz83ANQUYed-AA

- 从下面的代码中，我们可以分析出，trim 标签在最前方 拼接了一个where ，并且如果SQL最后末尾存在 and 它也会帮我自动给处理掉。
- trim 标签不单单可以取代where, 也可以取代set标签

###### trim 取代 where标签

```xml
<!--public List<Employee> getEmpsByConditionTrim(Employee employee);  -->
<select id="getEmpsByConditionTrim" resultType="com.atguigu.mybatis.bean.Employee">
    
    <!-- 后面多出的and或者or where标签不能解决 
           prefix="":前缀：trim标签体中是整个字符串拼串 后的结果。
             prefix给拼串后的整个字符串加一个前缀 
           prefixOverrides="":
             前缀覆盖： 去掉整个字符串前面多余的字符
           suffix="":后缀
             suffix给拼串后的整个字符串加一个后缀 
           suffixOverrides=""
             后缀覆盖：去掉整个字符串结尾多余的字符

        -->
    <!-- 自定义字符串的截取规则 -->
    select * from tbl_employee
    <trim prefix="where" suffixOverrides="and">
        <if test="id!=null">
            id=#{id} and
        </if>
        <if test="lastName!=null and lastName!='' ">
            last_name like #{lastName} and
        </if>
        <if test="email!=null and email.trim()!=''">
            email=#{email} and
        </if>
    </trim>
    
    
  <!--  
    //写法一
    select * from tbl_employee
    <where>
        <if test="id!=null">
            id=#{id} and
        </if>
        <if test="lastName!=null and lastName!='' ">
            last_name like #{lastName} and
        </if>
        <if test="email!=null and email.trim()!=''">
            email=#{email} and
        </if>
    </where>

    //写法二
    select * from tbl_employee
    <where>
        <if test="id!=null">
            id=#{id}
        </if>
        <if test="lastName!=null and lastName!='' ">
             and last_name like #{lastName}
        </if>
        <if test="email!=null and email.trim()!=''">
             and email=#{email}
        </if>
    </where>
    
    -->
    
    
    
</select>
```

###### trim 取代 set标签

```xml
<!--public void updateEmp(Employee employee);  -->
<update id="updateEmp">
    
    update tbl_employee 
    <trim prefix="set" suffixOverrides=",">
        <if test="lastName!=null">
            last_name=#{lastName},
        </if>
        <if test="email!=null">
            email=#{email},
        </if>
        <if test="gender!=null">
            gender=#{gender},
        </if>
    </trim>
    where id=#{id}
    
    <!-- 
        update tbl_employee 
        <set>
            <if test="lastName!=null">
                last_name=#{lastName},
            </if>
            <if test="email!=null">
                email=#{email},
            </if>
            <if test="gender!=null">
                gender=#{gender}
            </if>
        </set>
        where id=#{id}  
    -->		
   
</update>
```



### 3.MyBatis 核心配置文件 – 分页插件

#### 3.1 分页插件介绍

- 分页可以将固定条数结果在页面进行显示。 

- 如果当前在第一页，则没有上一页。如果当前在最后一页，则没有下一页。 

- 需要明确当前是第几页，这一页中显示多少条结果。

  > rows = 10 //每页显示10条
  >
  > limit m, n //limit关键字是MySQL 特有的，其他数据库会有替代的关键字
  >
  > m = (currentPage - 1)  * rows

- 在企业级开发中，分页也是一种常见的技术。而目前使用的MyBatis 是不带分页功能的，如果想实现分页的 功能，需要我们手动编写LIMIT 语句。但是不同的数据库实现分页的SQL 语句也是不同的，所以手写分页 成本较高。这个时候就可以借助分页插件来帮助我们实现分页功能

- PageHelper：

  > 第三方分页助手。将复杂的分页操作进行封装，从而让分页功能变得非常简单。



#### 3.2 分页插件实现步骤

1）导入 jar 包。 pagehelper-5.1.10.jar

2）在核心配置文件中集成分页助手插件。 

3）在测试类中使用分页助手相关API 实现分页功能。

**示例代码:**

**MyBatisConfig.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--configuration 核心根标签-->
<configuration>

    <!--集成分页助手插件-->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
    </plugins>
    
    
</configuration>
```

**Test01.java**

<font color=red>PageHelper.startPage(1,3); 一定要写在SQL执行之前</font>

```java
package com.itheima.paging;

import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import com.itheima.bean.Student;
import com.itheima.mapper.StudentMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.InputStream;
import java.util.List;

public class Test01 {
    
    @Test
    public void selectPaging() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.获取StudentMapper接口的实现类对象
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);

        //通过分页助手来实现分页功能
        // 第一页：显示3条数据
        //PageHelper.startPage(1,3);
        // 第二页：显示3条数据
        //PageHelper.startPage(2,3);
        // 第三页：显示3条数据
        PageHelper.startPage(3,3);

        //5.调用实现类的方法，接收结果
        List<Student> list = mapper.selectAll();

        //6.处理结果
        for (Student student : list) {
            System.out.println(student);
        }

        //获取分页相关参数
        PageInfo<Student> info = new PageInfo<>(list);
        System.out.println("总条数：" + info.getTotal());
        System.out.println("总页数：" + info.getPages());
        System.out.println("当前页：" + info.getPageNum());
        System.out.println("每页显示条数：" + info.getPageSize());
        System.out.println("上一页：" + info.getPrePage());
        System.out.println("下一页：" + info.getNextPage());
        System.out.println("是否是第一页：" + info.isIsFirstPage());
        System.out.println("是否是最后一页：" + info.isIsLastPage());

        //7.释放资源
        sqlSession.close();
        is.close();
    }
}

```

#### 3.3 分页插件相关参数--示例代码见上面代码片段

 PageInfo：封装分页相关参数的功能类。 

**核心方法**

| 返回值  |     方法名      |        说明        |
| :-----: | :-------------: | :----------------: |
|  long   |   getTotal()    |     获取总条数     |
|   int   |   getPages()    |     获取总页数     |
|   int   |  getPageNum()   |     获取当前页     |
|   int   |  getPageSize()  |  获取每页显示条数  |
|   int   |  getPrePage()   |     获取上一页     |
|   int   |  getNextPage()  |     获取下一页     |
| boolean | isIsFirstPage() |  获取是否是第一页  |
| boolean | isIsLastPage()  | 获取是否是最后一页 |

#### 3.4 分页插件小结

-  分页：可以将很多条结果进行分页显示。

- 分页插件 jar 包：pagehelper-5.1.10.jar        jsqlparser-3.1.jar 

- <plugins>：集成插件标签。 

- 分页助手相关 API 

  > PageHelper：分页助手功能类。 
  >
  > ​       startPage()：设置分页参数 <font color=red>必须写在执行SQL方法调用之前</font>
  >
  > PageInfo：分页相关参数功能类。 
  >
  > ​       getTotal()：获取总条数 
  >
  > ​       getPages()：获取总页数 
  >
  > ​       getPageNum()：获取当前页 
  >
  > ​       getPageSize()：获取每页显示条数 
  >
  > ​       getPrePage()：获取上一页 
  >
  > ​       getNextPage()：获取下一页 
  >
  > ​       isIsFirstPage()：获取是否是第一页 
  >
  > ​       isIsLastPage()：获取是否是最后一页



### 4.MyBatis 多表操作

#### 注意：

<font color=red>接口代理的方式，用到ResultMap 标签时, JavaBean至少要有一个无参数的构造方法；, 否则会出现异常：Cause: java.lang.IllegalArgumentException: argument type mismatch</font>

#### 4.1 多表模型介绍

- 我们之前学习的都是基于单表操作的，而实际开发中，随着业务难度的加深，肯定需要多表操作的。 

- 一对一：在任意一方建立**外键**，关联对方的主键。

  >  用户基本信息表  和  用户详细信息表

- 一对多：在多的一方建立外键，关联一的一方的主键。

  > 店铺表 和 商品表
  >
  > 用户表 和 订单表

- 多对多：借助中间表，中间表至少两个字段，分别关联两张表的主键。

  > 学生表 和 课程表
  >
  > 用户表 和  兴趣爱好表

- <font color=red>外键字段的建立是体现表与表关系的所在</font>

#### 4.2 多表操作一对一

 一对一模型：人和身份证，一个人只有一个身份证。 

##### **环境准备**

```mysql
CREATE DATABASE db2;

USE db2;

CREATE TABLE person(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20),
	age INT
);
INSERT INTO person VALUES (NULL,'张三',23);
INSERT INTO person VALUES (NULL,'李四',24);
INSERT INTO person VALUES (NULL,'王五',25);

CREATE TABLE card(
	id INT PRIMARY KEY AUTO_INCREMENT,
	number VARCHAR(30),
	pid INT,
	CONSTRAINT cp_fk FOREIGN KEY (pid) REFERENCES person(id)
);
INSERT INTO card VALUES (NULL,'12345',1);
INSERT INTO card VALUES (NULL,'23456',2);
INSERT INTO card VALUES (NULL,'34567',3);
```

##### **OneToOneMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--
            <resultMap>：配置字段和对象属性的映射关系标签。 
               属性：
                   id 属性：唯一标识 
                   type 属性：实体对象类型 

            <id>：配置主键映射关系标签。 
            <result>：配置非主键映射关系标签。 
               属性：
                   column 属性：表中字段名称 
                   property 属性： 实体对象变量名称
            <association>：配置被包含对象的映射关系标签。                            属性：
                   property 属性：被包含对象的变量名 
                   javaType 属性：被包含对象的数据类型
-->

<mapper namespace="com.itheima.table01.OneToOneMapper">
    <!--配置字段和实体对象属性的映射关系-->
    <resultMap id="oneToOne" type="card">
        <id column="cid" property="id" />
        <result column="number" property="number" />
        <!--
            association：配置被包含对象的映射关系
            property：被包含对象的变量名
            javaType：被包含对象的数据类型
        -->
        <association property="p" javaType="person">
            <id column="pid" property="id" />
            <result column="name" property="name" />
            <result column="age" property="age" />
        </association>
    </resultMap>

    <select id="selectAll" resultMap="oneToOne">
        SELECT c.id cid,number,pid,NAME,age FROM card c,person p WHERE c.pid=p.id
    </select>
</mapper>
```

##### **OneToOneMapper.java**

```java
public interface OneToOneMapper {
    //查询全部
    public abstract List<Card> selectAll();
}

```

##### **测试类-Test01.java**

```java
package com.itheima.table01;

import com.itheima.bean.Card;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.InputStream;
import java.util.List;

public class Test01 {
    @Test
    public void selectAll() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.获取OneToOneMapper接口的实现类对象
        OneToOneMapper mapper = sqlSession.getMapper(OneToOneMapper.class);

        //5.调用实现类的方法，接收结果
        List<Card> list = mapper.selectAll();

        //6.处理结果
        for (Card c : list) {
            System.out.println(c);
        }

        //7.释放资源
        sqlSession.close();
        is.close();
    }
}

```



#### 4.3 多表操作一对多

 一对多模型：班级和学生，一个班级可以有多个学生。 

##### **环境准备**

```mysql
CREATE TABLE classes(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20)
);
INSERT INTO classes VALUES (NULL,'黑马一班');
INSERT INTO classes VALUES (NULL,'黑马二班');


CREATE TABLE student(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(30),
	age INT,
	cid INT,
	CONSTRAINT cs_fk FOREIGN KEY (cid) REFERENCES classes(id)
);
INSERT INTO student VALUES (NULL,'张三',23,1);
INSERT INTO student VALUES (NULL,'李四',24,1);
INSERT INTO student VALUES (NULL,'王五',25,2);
INSERT INTO student VALUES (NULL,'赵六',26,2);
```

##### **OneToManyMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
            <resultMap>：配置字段和对象属性的映射关系标签。 
               属性：
                   id 属性：唯一标识 
                   type 属性：实体对象类型 

            <id>：配置主键映射关系标签。 
            <result>：配置非主键映射关系标签。 
               属性：
                   column 属性：表中字段名称 
                   property 属性： 实体对象变量名称
            <collection>：配置被包含集合对象的映射关系标签。 
               属性：
                   property 属性：被包含集合对象的变量名 
                   ofType 属性：集合中保存的对象数据类型
-->
<mapper namespace="com.itheima.table02.OneToManyMapper">
    <resultMap id="oneToMany" type="classes">
        <id column="cid" property="id"/>
        <result column="cname" property="name"/>

        <!--
            collection：配置被包含的集合对象映射关系
            property：被包含对象的变量名
            ofType：被包含对象的实际数据类型
        -->
        <collection property="students" ofType="student">
            <id column="sid" property="id"/>
            <result column="sname" property="name"/>
            <result column="sage" property="age"/>
        </collection>
    </resultMap>
    <select id="selectAll" resultMap="oneToMany">
        SELECT c.id cid,c.name cname,s.id sid,s.name sname,s.age sage FROM classes c,student s WHERE c.id=s.cid
    </select>
</mapper>
```

##### OneToManyMapper.java

```java
package com.itheima.table02;

import com.itheima.bean.Classes;

import java.util.List;

public interface OneToManyMapper {
    //查询全部
    public abstract List<Classes> selectAll();
}

```

##### 测试类-Test01.java

```java
package com.itheima.table02;

import com.itheima.bean.Classes;
import com.itheima.bean.Student;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.InputStream;
import java.util.List;

public class Test01 {
    @Test
    public void selectAll() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.获取OneToManyMapper接口的实现类对象
        OneToManyMapper mapper = sqlSession.getMapper(OneToManyMapper.class);

        //5.调用实现类的方法，接收结果
        List<Classes> classes = mapper.selectAll();

        //6.处理结果
        for (Classes cls : classes) {
            System.out.println(cls.getId() + "," + cls.getName());
            List<Student> students = cls.getStudents();
            for (Student student : students) {
                System.out.println("\t" + student);
            }
        }

        //7.释放资源
        sqlSession.close();
        is.close();
    }
}

```

#### 4.4 多表操作多对多

 多对多模型：学生和课程，一个学生可以选择多门课程、一个课程也可以被多个学生所选择

##### **环境准备**

```mysql
CREATE TABLE course(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20)
);
INSERT INTO course VALUES (NULL,'语文');
INSERT INTO course VALUES (NULL,'数学');


CREATE TABLE stu_cr(
	id INT PRIMARY KEY AUTO_INCREMENT,
	sid INT,
	cid INT,
	CONSTRAINT sc_fk1 FOREIGN KEY (sid) REFERENCES student(id),
	CONSTRAINT sc_fk2 FOREIGN KEY (cid) REFERENCES course(id)
);
INSERT INTO stu_cr VALUES (NULL,1,1);
INSERT INTO stu_cr VALUES (NULL,1,2);
INSERT INTO stu_cr VALUES (NULL,2,1);
INSERT INTO stu_cr VALUES (NULL,2,2);
```

##### ManyToManyMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
            <resultMap>：配置字段和对象属性的映射关系标签。 
               属性：
                   id 属性：唯一标识 
                   type 属性：实体对象类型 

            <id>：配置主键映射关系标签。 
            <result>：配置非主键映射关系标签。 
               属性：
                   column 属性：表中字段名称 
                   property 属性： 实体对象变量名称
            <collection>：配置被包含集合对象的映射关系标签。 
               属性：
                   property 属性：被包含集合对象的变量名 
                   ofType 属性：集合中保存的对象数据类型
-->
<mapper namespace="com.itheima.table03.ManyToManyMapper">
    <resultMap id="manyToMany" type="student">
        <id column="sid" property="id"/>
        <result column="sname" property="name"/>
        <result column="sage" property="age"/>

        <collection property="courses" ofType="course">
            <id column="cid" property="id"/>
            <result column="cname" property="name"/>
        </collection>
    </resultMap>
    <select id="selectAll" resultMap="manyToMany">
        SELECT sc.sid,s.name sname,s.age sage,sc.cid,c.name cname FROM student s,course c,stu_cr sc WHERE sc.sid=s.id AND sc.cid=c.id
    </select>
</mapper>
```

##### ManyToManyMapper.java

```java
import com.itheima.bean.Student;

import java.util.List;

public interface ManyToManyMapper {
    //查询全部
    public abstract List<Student> selectAll();
}

```

##### 测试类-Test01.java

```java
package com.itheima.table03;

import com.itheima.bean.Course;
import com.itheima.bean.Student;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.InputStream;
import java.util.List;

public class Test01 {
    @Test
    public void selectAll() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.获取ManyToManyMapper接口的实现类对象
        ManyToManyMapper mapper = sqlSession.getMapper(ManyToManyMapper.class);

        //5.调用实现类的方法，接收结果
        List<Student> students = mapper.selectAll();

        //6.处理结果
        for (Student student : students) {
            System.out.println(student.getId() + "," + student.getName() + "," + student.getAge());
            List<Course> courses = student.getCourses();
            for (Course cours : courses) {
                System.out.println("\t" + cours);
            }
        }

        //7.释放资源
        sqlSession.close();
        is.close();
    }
}

```

#### 4.5 多表操作小结

```xml
<!--一对一 所有标签如下

            <resultMap>：配置字段和对象属性的映射关系标签。 
               属性：
                   id 属性：唯一标识 
                   type 属性：实体对象类型 

            <id>：配置主键映射关系标签。 
            <result>：配置非主键映射关系标签。 
               属性：
                   column 属性：表中字段名称 
                   property 属性： 实体对象变量名称
         (*)<association>：配置被包含对象的映射关系标签。                            属性：
                   property 属性：被包含对象的变量名 
                   javaType 属性：被包含对象的数据类型
-->


<!--多对多 & 一对多 所有标签如下

            <resultMap>：配置字段和对象属性的映射关系标签。 
               属性：
                   id 属性：唯一标识 
                   type 属性：实体对象类型 

            <id>：配置主键映射关系标签。 
            <result>：配置非主键映射关系标签。 
               属性：
                   column 属性：表中字段名称 
                   property 属性： 实体对象变量名称
         (*)<collection>：配置被包含集合对象的映射关系标签。 
               属性：
                   property 属性：被包含集合对象的变量名 
                   ofType 属性：集合中保存的对象数据类型
-->
```



### 5. 扩展

#### 5.1 动脑来想一想？

- 以上多表查询是否还有简单实现方式？

  > 1) 其实在企业中我们并不会采用上述方式来处理多表查询的结果集数据
  >
  > 2) 多表查询也好，单表查询也好，都是将结果集封装到我们的javabean对象中
  >
  > 3) 我们可以针对查询结果集来创建对应的javabean类，来接收结果集数据，这种javabean我们在领域驱动模型中称之为：VO
  >
  > 4) 有了这个java类，我们就不需要考虑：表与表关系，繁琐的标签配置

##### 5.1.1 多表操作一对一

###### PersonCardVo.java

```java
package com.itheima.bean.vo;

public class PersonCardVo {

    private Integer pid;     //人表主键id
    private String name;    //人的姓名
    private Integer age;    //人的年龄

    private Integer cid;     //身份证号表主键

    private String number;  //身份证号

    public PersonCardVo(){}

    public Integer getPid() {
        return pid;
    }

    public void setPid(Integer pid) {
        this.pid = pid;
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

    public Integer getCid() {
        return cid;
    }

    public void setCid(Integer cid) {
        this.cid = cid;
    }

    public String getNumber() {
        return number;
    }

    public void setNumber(String number) {
        this.number = number;
    }

    @Override
    public String toString() {
        return "PersonCardVo{" +
                "pid=" + pid +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", cid=" + cid +
                ", number='" + number + '\'' +
                '}';
    }
}

```

###### OneToOneMapper.java

```java
package com.itheima.one_to_one;

import com.itheima.bean.Card;
import com.itheima.bean.vo.PersonCardVo;

import java.util.List;

public interface OneToOneMapper {
    //查询全部
    public abstract List<PersonCardVo> selectAllZls();
}

```

###### OneToOneMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.one_to_one.OneToOneMapper">

    <!--方式二-->
    <select id="selectAllZls" resultType="com.itheima.bean.vo.PersonCardVo">
        SELECT c.id AS cid, c.number, c.pid, p.`name`, p.age FROM card c,person p WHERE c.pid=p.id
    </select>

</mapper>
```

###### TestOneToOne.java

```java
package com.itheima.test;

import com.itheima.bean.Card;
import com.itheima.bean.vo.PersonCardVo;
import com.itheima.one_to_one.OneToOneMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.InputStream;
import java.util.List;

public class TestOneToOne {

    /**
     * 方式二
     * @throws Exception
     */
    @Test
    public void selectAllZls() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.获取OneToOneMapper接口的实现类对象
        OneToOneMapper mapper = sqlSession.getMapper(OneToOneMapper.class);

        //5.调用实现类的方法，接收结果
        List<PersonCardVo> list = mapper.selectAllZls();

        //6.处理结果
        for (PersonCardVo personCardVo : list) {
            System.out.println(personCardVo);
        }

        //7.释放资源
        sqlSession.close();
        is.close();
    }

}

```



##### 5.1.2 多表操作一对多

###### PersonCardVo.java

```java
package com.itheima.bean.vo;

public class ClassesStudentVo {

    private Integer cid;     //班级的主键id
    private String cname;    //班级名称

    private Integer sid;     //学生的主键id
    private String sname;    //学生姓名
    private Integer sage;    //学生年龄

    public ClassesStudentVo(){}

    public Integer getCid() {
        return cid;
    }

    public void setCid(Integer cid) {
        this.cid = cid;
    }

    public String getCname() {
        return cname;
    }

    public void setCname(String cname) {
        this.cname = cname;
    }

    public Integer getSid() {
        return sid;
    }

    public void setSid(Integer sid) {
        this.sid = sid;
    }

    public String getSname() {
        return sname;
    }

    public void setSname(String sname) {
        this.sname = sname;
    }

    public Integer getSage() {
        return sage;
    }

    public void setSage(Integer sage) {
        this.sage = sage;
    }

    @Override
    public String toString() {
        return "ClassesStudentVo{" +
                "cid=" + cid +
                ", cname='" + cname + '\'' +
                ", sid=" + sid +
                ", sname='" + sname + '\'' +
                ", sage=" + sage +
                '}';
    }
}

```

###### OneToManyMapper.java

```java
package com.itheima.one_to_many;

import com.itheima.bean.Classes;
import com.itheima.bean.vo.ClassesStudentVo;

import java.util.List;

public interface OneToManyMapper {

    //查询全部
    public abstract List<ClassesStudentVo> selectAllZls();

}

```

###### OneToManyMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.one_to_many.OneToManyMapper">

    <!-- 方式二 -->
    <select id="selectAllZls"  resultType="com.itheima.bean.vo.ClassesStudentVo">
         SELECT
             c.id AS cid,c.name AS cname,s.id AS sid,s.name AS sname,s.age AS sage
         FROM
             classes AS c,student AS s
         WHERE
            c.id=s.cid
    </select>


</mapper>
```

###### TestOneToMany.java

```java
package com.itheima.test;

import com.itheima.bean.Classes;
import com.itheima.bean.Student;
import com.itheima.bean.vo.ClassesStudentVo;
import com.itheima.one_to_many.OneToManyMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.InputStream;
import java.util.List;

public class TestOneToMany {

    /**
     * 方式二
     * @throws Exception
     */
    @Test
    public void selectAllZls() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.获取OneToManyMapper接口的实现类对象
        OneToManyMapper mapper = sqlSession.getMapper(OneToManyMapper.class);

        //5.调用实现类的方法，接收结果
        List<ClassesStudentVo> classes = mapper.selectAllZls();

        //6.处理结果
        for (ClassesStudentVo classesStudentVo : classes) {
            System.out.println(classesStudentVo);
        }

        //7.释放资源
        sqlSession.close();
        is.close();
    }


}

```



##### 5.1.3 多表操作多对多

- 策略和一对多一样思想

###### StudentCourseVo.java

```java
package com.itheima.bean.vo;

public class StudentCourseVo {

    private Integer sid;     //学生的主键id
    private String sname;    //学生姓名
    private Integer sage;    //学生年龄

    private Integer cid;     //课程的主键id
    private String cname;    //课程名称

    public StudentCourseVo(){}

    public Integer getSid() {
        return sid;
    }

    public void setSid(Integer sid) {
        this.sid = sid;
    }

    public String getSname() {
        return sname;
    }

    public void setSname(String sname) {
        this.sname = sname;
    }

    public Integer getSage() {
        return sage;
    }

    public void setSage(Integer sage) {
        this.sage = sage;
    }

    public Integer getCid() {
        return cid;
    }

    public void setCid(Integer cid) {
        this.cid = cid;
    }

    public String getCname() {
        return cname;
    }

    public void setCname(String cname) {
        this.cname = cname;
    }

    @Override
    public String toString() {
        return "StudentCourseVo{" +
                "sid=" + sid +
                ", sname='" + sname + '\'' +
                ", sage=" + sage +
                ", cid=" + cid +
                ", cname='" + cname + '\'' +
                '}';
    }
}

```

###### ManyToManyMapper.java

```java
package com.itheima.many_to_many;

import com.itheima.bean.Student;
import com.itheima.bean.vo.StudentCourseVo;

import java.util.List;

public interface ManyToManyMapper {
    //查询全部
    public abstract List<StudentCourseVo> selectAllZls();
}

```

###### ManyToManyMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.many_to_many.ManyToManyMapper">

    <!--方式二-->
    <select id="selectAllZls" resultType="com.itheima.bean.vo.StudentCourseVo">
        SELECT
           sc.sid, s.name sname, s.age sage, c.`id` cid, c.name cname
        FROM
           student s, course c, stu_cr sc
        WHERE
           sc.sid=s.id AND sc.cid=c.id
    </select>


</mapper>
```

###### TestManyToMany.java

```java
package com.itheima.test;

import com.itheima.bean.Course;
import com.itheima.bean.Student;
import com.itheima.bean.vo.StudentCourseVo;
import com.itheima.many_to_many.ManyToManyMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.InputStream;
import java.util.List;

public class TestManyToMany {

    /**
     * 方式二
     * @throws Exception
     */
    @Test
    public void selectAllZls() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.获取ManyToManyMapper接口的实现类对象
        ManyToManyMapper mapper = sqlSession.getMapper(ManyToManyMapper.class);

        //5.调用实现类的方法，接收结果
        List<StudentCourseVo> students = mapper.selectAllZls();

        //6.处理结果
        for (StudentCourseVo studentCourseVo : students) {
            System.out.println(studentCourseVo);
        }

        //7.释放资源
        sqlSession.close();
        is.close();
    }

}

```





#### 5.2 mapper 接口方法参数传值

##### 单参数传值

StudentMapper.java

```java
//根据id查询
public abstract Student selectById(Integer id);
```

StudentMapper.xml

```xml
<select id="selectById" resultType="student" parameterType="int">    
    SELECT * FROM student WHERE id = #{aaaa}
</select>
```

<font color=red>如上可以看出，当只有一个参数时，我们#{}中写任意字符串都可以顺利获取到值。</font>



##### 多参数传值

前言： 多个参数mybatis 会做特殊处理, 多个参数会被封装成一个map

> key: param1 ..... paramN 或者arg1 .....argN 也可以
>
> value: 就是传入的值
>
> #{}就是从map中获取指定key的值
>
> 当存在多个参数时，<delete id="deleteByParams" > 标签中 parameterType属性**省略不写**

###### 基本操作

示例代码如下：** 

StudentMapper.xml

```xml
<delete id="deleteByParams" >
    DELETE FROM student WHERE id = #{param1} and age = #{param2}
</delete>

```

测试代码

```java
@Test
public Integer deleteByParams() {
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

        //4.获取StudentMapper接口的实现类对象
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class); // StudentMapper mapper = new StudentMapperImpl();

        //5.通过实现类对象调用方法，接收结果
        result = mapper.deleteByParams(4, 23);

    } catch (Exception e) {

    } finally {
        //6.释放资源
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

    //7.返回结果
    return result;
}
```

基于注解也是一样的

```java
//删除操作
@Delete("DELETE FROM student WHERE id=#{param1} and age = #{param2}")
public abstract Integer deleteByParams(Integer id, Integer age);
```

###### 思考一下

以上实现代码中，我们可以看到，param1.....paramN这种约定俗成的key名称不太好记忆，那么我们可否针对多个参数，为每个参数自己起别名呢？

- @Param("id")  该注解可以给每个参数自定义别名

StudentMapper.java

```java
public abstract Integer deleteByParams(@Param("stuid")  Integer id, @Param("stuage")  Integer age);
```

StudentMapper.xml

```xml
<delete id="deleteByParams" >
    DELETE FROM student WHERE id = #{stuid} and age = #{stuage}
</delete>

```



注解实现方式 

```java 
//删除操作
@Delete("DELETE FROM student WHERE id=#{stuid} and age = #{stuage}")
public abstract Integer deleteByParams(@Param("stuid")  Integer id, @Param("stuage")  Integer age);
```



###### 继续思考

- 定义多个参数如果嫌弃麻烦，也可以定义一个参数，可以是实体也可以是Map集合对象。 如下演示Map对象方式:

StudentMapper.java

```java
/*
   map.put("stuid", 4);
   map.put("stuage", 23);
*/
public abstract Integer deleteByParams(Map map);
```

StudentMapper.xml

```xml
<delete id="deleteByParams" >
    DELETE FROM student WHERE id = #{stuid} and age = #{stuage}
</delete>

```

###### 总结

- public Student     getStudent( @Param("sid") Integer  id,  String name );

  > 取值: id ==>   #{sid/param1}    name ==> #{param2}

- public Student    getStudent( Integer  id,  @Param("student") Student  stu );

  > 取值: id ==>   #{param1}    name ==> #{param2/student.name}



##### 5.3 企业示例代码演示

###### 预览: 演示一

```xml
 <!--  添加 省级管理员 & 全国级管理员副手  管理员  -->
<insert id="addUser" parameterType="com.platform.entity.User">
    INSERT INTO 
    t_user(user_id , username ,password)
    VALUES
    (#{user_id},#{username},#{password})
</insert>
	
	 
<!--  修改 用户信息  ${tableName} -->
<update id="updateUserInfo" parameterType="User">
    UPDATE t_user
    <set>
        <if test="password != null and password != '' ">
            password = #{password},
        </if>
        <if test="nickname != null and nickname != '' ">
            nickname = #{nickname},
        </if>
        <if test="phone != null and phone != '' ">
            phone = #{phone},
        </if>
        <if test="email != null and email != '' ">
            email = #{email},
        </if>
        <if test="scope != null and scope != '' ">
            scope = #{scope},
        </if>
        <if test="employers_id != null and employers_id != '' ">
            employers_id = #{employers_id},
        </if>
    </set>
    WHERE user_id = #{user_id} 
</update>


<!-- 全国级管理员删除他 添加的  管理员-->
<delete id="delectUsersByUser_id" parameterType="java.util.List">
    DELETE  FROM t_user  WHERE  user_id  in  
    <foreach collection="list" item = "model" open="(" separator="," close=")">
        #{model.user_id}  
    </foreach>
</delete>
	
<!-- 获取副级管理员的个数 -->
<select id="findCountryAdminNumberByScope_idAndLevel" parameterType="Map" resultType="int">
    select count(*) from t_user 
    WHERE scope = #{scope}  and user_level = #{user_level}
</select>
	
<!-- 获取用户信息，条件 ： 账号 -->
<select id="findUsreInfoByUsername" parameterType="String" resultType="com.platform.entity.User">
    select * from t_user  t , t_employers te
    WHERE t.username = #{username} and t.employers_id = te.employers_id
</select>

<!--  获取所有省级管理员(正级 和副级) -->
<select id="selectProviceAdmin" parameterType="Map" resultType="com.platform.entity.User">
    select * from t_user  tu , t_employers te , t_province  tp 
    WHERE   tu.scope  in ( select province_id from t_province )
    AND user_type = 2
    AND tu.employers_id = te.employers_id
    AND tu."scope" = tp.province_id
</select>
	
    
<!-- 获取 检测数据    除了ATP —— -->
<select id="getHospitalAndTest"  parameterType="java.util.Map"  resultType="com.platform.entity.Hostpital_Device">
    SELECT 
        *  
    FROM ${t_hospital} th, ${test_tableName} tt

    <!-- 也可以用<where>标签进行包裹 -->
    WHERE  tt.hospital_id = th.hospital_id   
    <if test="hospital_name != null">
        AND th.hospital_name LIKE CONCAT(CONCAT('%', #{hospital_name}), '%')
    </if> 
    <if test="hospital_id != null">
        AND th.hospital_id = #{hospital_id}
    </if>
    <if test="test_result != null">
        AND  tt.test_result = #{test_result} 
    </if>  
    <if test="start_time != null">
        AND tt.test_time  >=  #{start_time}
    </if> 
    ORDER BY  tt.test_time DESC 
</select>
```



###### 预览: 演示二

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.mybatis.dao.EmployeeMapperDynamicSQL">
    <!-- 
        • if:判断
        • choose (when, otherwise):分支选择；带了break的swtich-case
         如果带了id就用id查，如果带了lastName就用lastName查;只会进入其中一个
        • trim 字符串截取(where(封装查询条件), set(封装修改条件))
        • foreach 遍历集合
  -->
    
    <!--public List<Employee> getEmpsByConditionTrim(Employee employee);  -->
    <select id="getEmpsByConditionTrim" resultType="com.atguigu.mybatis.bean.Employee">
        select * from tbl_employee
        <!-- 后面多出的and或者or where标签不能解决 
           prefix="":前缀：trim标签体中是整个字符串拼串 后的结果。
             prefix给拼串后的整个字符串加一个前缀 
           prefixOverrides="":
             前缀覆盖： 去掉整个字符串前面多余的字符
           suffix="":后缀
             suffix给拼串后的整个字符串加一个后缀 
           suffixOverrides=""
             后缀覆盖：去掉整个字符串后面多余的字符

        -->
        <!-- 自定义字符串的截取规则 -->
        <trim prefix="where" suffixOverrides="and">
            <if test="id!=null">
                id=#{id} and
            </if>
            <if test="lastName!=null and lastName!='' ">
                last_name like #{lastName} and
            </if>
            <if test="email!=null and email.trim()!=''">
                email=#{email} and
            </if>
        </trim>
    </select>

    <!-- public List<Employee> getEmpsByConditionChoose(Employee employee); -->
    <select id="getEmpsByConditionChoose" resultType="com.atguigu.mybatis.bean.Employee">
        select * from tbl_employee 
        <where>
            <!-- 如果带了id就用id查，如果带了lastName就用lastName查;只会进入其中一个 -->
            <choose>
                <when test="id!=null">
                    id=#{id}
                </when>
                <when test="lastName!=null">
                    last_name like #{lastName}
                </when>
                <when test="email!=null">
                    email = #{email}
                </when>
                <otherwise>
                    gender = 0
                </otherwise>
            </choose>
        </where>
    </select>

    <!--public void updateEmp(Employee employee);  -->
    <update id="updateEmp">
        <!-- Set标签的使用 -->
        update tbl_employee 
        <set>
            <if test="lastName!=null">
                last_name=#{lastName},
            </if>
            <if test="email!=null">
                email=#{email},
            </if>
            <if test="gender!=null">
                gender=#{gender}
            </if>
        </set>
        where id=#{id} 
        <!-- 		
          Trim：更新拼串  suffixOverrides去掉最后一个 , 号
          update tbl_employee 
          <trim prefix="set" suffixOverrides=",">
           <if test="lastName!=null">
            last_name=#{lastName},
           </if>
           <if test="email!=null">
            email=#{email},
           </if>
           <if test="gender!=null">
            gender=#{gender}
           </if>
          </trim>
          where id=#{id}  -->
    </update>

<!--public List<Employee>    getEmpsByConditionForeach(@Param("ids") List<Integer> ids);  
-->
    <select id="getEmpsByConditionForeach" resultType="com.atguigu.mybatis.bean.Employee">
        select * from tbl_employee
     <!--
        collection：指定要遍历的集合：
         list类型的参数会特殊处理封装在map中，map的key就叫list
        item：将当前遍历出的元素赋值给指定的变量
        separator:每个元素之间的分隔符
        open：遍历出所有结果拼接一个开始的字符
        close:遍历出所有结果拼接一个结束的字符
        index:索引。遍历list的时候是index就是索引，item就是当前值
                遍历map的时候index表示的就是map的key，item就是map的值

        #{变量名}就能取出变量的值也就是当前遍历出的元素
     -->
        <foreach collection="ids" item="item_id" separator=","
                 open="where id in(" close=")">
            #{item_id}
        </foreach>
    </select>
    

    <!-- 批量保存 -->
    <!--public void addEmps(@Param("emps")List<Employee> emps);  -->
    <!--MySQL下批量保存：可以foreach遍历   mysql支持values(),(),()语法-->
    <insert id="addEmps">
        insert into tbl_employee(
        <include refid="insertColumn"></include>
        ) 
        values
        <foreach collection="emps" item="emp" separator=",">
            (#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
        </foreach>
    </insert>


    <!-- 
    抽取可重用的sql片段。方便后面引用 
    1、sql抽取：经常将要查询的列名，或者插入用的列名抽取出来方便引用
    2、include来引用已经抽取的sql：
    3、include还可以自定义一些property，sql标签内部就能使用自定义的属性
      include-property：取值的正确方式${prop},
      #{不能使用这种方式}
   -->
    <sql id="insertColumn">
        last_name,email,gender,d_id
    </sql>

</mapper>
```

