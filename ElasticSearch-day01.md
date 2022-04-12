## ElasticSearch-day01

### 今日目标

- **创建 修改 和 删除** 数据库 表 字段（指定类型）可以在SQLyog中完成操作
- 对表中数据进行 **增删改查** 操作的 SQL 关键字 + 结构，加强练习，（企业就玩这些）

- 所有约束要知道其作用，记不住SQL 可以，但是要在sqlyog中完成约束的条件。（多加练习）





### 初识 ElasticSearch 

<a href="https://mp.weixin.qq.com/s/cmP5ggiFTPzY_dT6MCBRZw">从诗词大会到图解 ElasticSearch 原理解析</a>

- ElasticSearch是一个搜索服务器，百度，京东，淘宝，搜狗，拉勾网等，首页搜索沟通都可以使用ES来实现。

- 从检索数据的效率来看，同等数量的数据，比如：1亿，ES查询效率是远高于mysql。
- 关系型数据库侧重的是数据与数据之间的关系查询，ES侧重如何高效的检索出我们想要的数据，侧重点的不同必然决定了二者底层应用算法和设计不同，所以查询效率是有差距

- 往往我们会在一些需要快速检索数据业务采用ES。整套系统的搭建还是要由mysql来支撑的，二者并不存在谁取代谁。



#### ES核心倒排索引

- 正排索引：根据 “古诗名”  找诗句中的分词。
- 倒排索引：根据诗句中的分词，找“古诗名”。
- 将各个文档中的内容，先分词，然后记录分词和数据的唯一标识（id）的对 应关系，形成的产物。

| Key（term） |          Value           |
| :---------: | :----------------------: |
|     床      |        《静夜思》        |
|     前      |        《静夜思》        |
|    床前     |        《静夜思》        |
|    明月     | 《静夜思》，《水调歌头》 |
|     月      | 《静夜思》, 《水调歌头》 |
|     光      |        《静夜思》        |
|    月光     |        《静夜思》        |
|    几时     |       《水调歌头》       |
|     有      |       《水调歌头》       |



#### ElasticSearch存储和搜索原理

- 倒排索引是ES的核心，倒排索引建立少不了 **“分词”**。
- 存储数据时，要根据 **“分词”** 建立倒排索引库，然后将数据文档存储在库中。
- 当用户在百度输入检索内容时，也要根据**“分词”** 将用户输入的内容拆分，根据拆分出来的每一个分词，通过倒排索引，将数据文档检索出来，并展示。



![img](https://img-blog.csdnimg.cn/2020111118501446.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20201111185146455.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)



### 2. ElasticSearch概念

- ElasticSearch是一个基于Lucene(一套jar)的搜索服务器，同类产品还有solr。

- 是一个分布式、高扩展、高实时的搜索与数据分析引擎 

- 基于RESTful web接口 
- Elasticsearch是用Java语言开发的，并作为Apache许可条款下的开放源码发布，是一种 流行的企业级搜索引擎 

- 官网：https://www.elastic.co/



#### 2.1 应用场景    

- 搜索：海量数据的查询 

- 日志数据分析 

- 实时数据分析



#### 2.2 ES对比MySql   

- MySQL有事务性,而ElasticSearch没有事务性,所以你删了的数据是无法恢复的。
- ElasticSearch没有物理**外键**这个特性, 不存在数据关系这回事。
- 分工不同
  - MySQL负责存储数据和维护数据之间的关系
  - ElasticSearch负责缓存Mysql的某块搜索业务对应的数据，并提供**高效检索**功能。



![img](https://img-blog.csdnimg.cn/20201111185052109.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)



### 3. ElasticSearch 核心概念

- **索引**（index） ElasticSearch存储数据的地方，可以理解成关系型数据库中的数据库概念。 

- **映射**（mapping） mapping定义了每个字段的类型、字段所使用的分词器等。相当于关系型数据库中的表结构。 

- **文档**（document） Elasticsearch中的最小数据单元，常以json格式显示。一个document相当于关系型数据库中的一行数据。
- **倒排索引** 一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，对应一个包含它的文档id列表。
- 类型（type） 一种type就像一类表。如用户表、角色表等。
  - ES 5.x中一个index可以有多种type。 
  - _ES 6.x中一个index只能有一种type。_
  - _ES 7.x以后，将逐步移除type这个概念，现在的操作已经不再使用，默认_doc

![img](https://img-blog.csdnimg.cn/20201111185212283.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)



### 4. 脚本操作 ElasticSearch



#### 4.1 RESTful风格

- REST（Representational State Transfer），表述性状态转移，是一组架构约束条件和原则。满足这些约束条件和 原则的应用程序或设计就是RESTful。就是一种定义接口的规范。 
- 基于HTTP。
- 可以使用XML格式定义或JSON格式定义。
- 每一个URI代表1种资源。 
- 客户端使用GET、POST、PUT、DELETE4个请求方式，表示对同一个资源不同动作，例如：有一个/user资源
  - get：/user/1    用来获取资源 
  - post：/user     用来新建资源（也可以用于更新资源） 
  - put：/user       用来更新资源 
  - delete：/user/1  用来删除资源



#### 4.2_操作索引

```mysql
/*
	Postman工具
*/

-- 1.添加索引
-- PUT http://ip:端口/索引名称 
localhost:9200/goods_index


-- 2.查看索引
-- GET http://ip:端口/索引名称 # 查询单个索引信息 
-- GET http://ip:端口/索引名称1,索引名称2... # 查询多个索引信息 
-- GET http://ip:端口/_all # 查询所有索引信息 
localhost:9200/goods_index
localhost:9200/_all


-- 3.删除指定索引
-- DELETE http://ip:端口/索引名称 
localhost:9200/goods_index


-- 4.关闭指定索引
-- POST http://ip:端口/索引名称/_close 
localhost:9200/goods_index/_close


-- 5.打开指定索引
-- POST http://ip:端口/索引名称/_open
localhost:9200/goods_index/_open


```

#### 4.3 操作映射

##### 数据类型   

• 简单数据类型 

• 复杂数据类型

###### 简单数据类型 

- 字符串 
  - • text：会分词，不支持聚合 
  - • keyword：不会分词，将全部内容作为词条，支持聚合

- 数值 

  - ①long     -- 带符号的64位整数，最小值-263，最大值263-1

    ②integer    -- 带符号的32位整数，最小值-231，最大值231-1

    ③short     -- 带符号的16位整数，最小值-32768，最大值32767

    ④byte     -- 带符号的8位整数，最小值-128，最小值127

    ⑤double    -- 双精度64位IEEE 754 浮点数

    ⑥float    -- 单精度32位IEEE 754 浮点数

    ⑦half_float  -- 半精度16位IEEE 754 浮点数

    ⑧scaled_float -- 带有缩放因子的缩放类型浮点数

- 布尔

  - • boolean 

-  二进制 
  - • binary 

- 范围类型
  -  integer_range， long_range， float_range，double_range，date_range 
- 日期
  - • date



######  复杂数据类型

- • 数组：[ ] 
- • 对象：{ }



##### 操作映射

```mysql
/*
	kibana页面
*/

# 1.创建索引
PUT person_index
# 2.查询索引
GET person_index
# 3.删除索引
DELETE person_index

# 4.查询索引的映射
GET person_index/_mapping 

# 5.添加映射, json 和 指令不能有空格, 否则执行不成功
PUT person_index/_mapping 
{
  "properties": {
    "name": {"type": "keyword"},
    "age": {"type": "integer"}
  }
}

# 6.创建索引并添加映射
PUT person_index 
{
  "mappings": {
    "properties": {
        "name": {"type": "keyword"},
        "age": {"type": "integer"}
 	 }
  }
}


# 7.追加索引映射字段
PUT person_index/_mapping
{
  "properties": {
    "address": {"type": "keyword"}
  }
}


```

#### 

#### 4.4 操作文档

```mysql
/*
	kibana页面
*/

# 8.添加文档, 指定id=1
# 不指定id, 必须为POST 请求方式
# 多次执行为update修改
#PUT person_index/_doc/1 
POST person_index/_doc
{
  "properties": {
    "name": "张三",
    "age": 23,
    "address": "北京回龙观"
  }
}


# 9.查看文档, 指定id=1
GET person_index/_doc/1 
# 9.1查询所有文档
GET person_index/_search


# 10.删除文档, 指定id=1
DELETE person_index/_doc/1 


```



### 5. IK分词器-(重点)

#### 5.1 分词器

- • 分词器（Analyzer）：将一段文本，按照一定逻辑，分析成多个词语的一种工具 如：华为手机 --- >  华为、手、手机 
- • ElasticSearch 内置分词器 
  - • Standard Analyzer -默认分词器，按词切分，小写处理 
  - • Simple Analyzer - 按照非字母切分(符号被过滤), 小写处理 
  - • Stop Analyzer - 小写处理，停用词过滤(the,a,is) 
  - • Whitespace Analyzer - 按照空格切分，不转小写 
  - • Keyword Analyzer - 不分词，直接将输入当作输出 
  - • Patter Analyzer - 正则表达式，默认\W+(非字符分割) 
  - • Language - 提供了30多种常见语言的分词器
- • ElasticSearch 内置分词器对中文很不友好，处理方式为：<font color=red>一个字一个词</font>



#### 5.2_IK分词器

- • IKAnalyzer是一个开源的，基于java语言开发的轻量级的中文分词工具包 
- • 是一个基于Maven构建的项目 • 具有60万字/秒的高速处理能力 
- • 支持用户词典扩展定义 
- • 下载地址：https://github.com/medcl/elasticsearch-analysis-ik/releases



#### 5.3 IK分词器使用

- IK分词器有两种分词模式：ik_max_word  和   ik_smart  模式。

##### ik_max_word

- 会将文本做最细粒度的拆分，比如会将   “乒乓球明年总冠军”    拆分为    “乒乓球、乒乓、球、明年、总冠军、冠军。

```mysql
/*
	kibana页面
*/

# ik分词器, 粗粒度分词
# ik_max_word
GET /_analyze
{
  "analyzer": "ik_max_word",
  "text": "乒乓球明年总冠军"
}
```

执行后，效果如下：

```json
{
  "tokens" : [
    {
      "token" : "乒乓球",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "乒乓",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "球",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "CN_CHAR",
      "position" : 2
    },
    {
      "token" : "明年",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "总冠军",
      "start_offset" : 5,
      "end_offset" : 8,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "冠军",
      "start_offset" : 6,
      "end_offset" : 8,
      "type" : "CN_WORD",
      "position" : 5
    }
  ]
}

```

##### ik_smart

会做最粗粒度的拆分，比如会将    “乒乓球明年总冠军    ”拆分为乒乓球、明年、总冠军。

```mysql
/*
	kibana页面
*/

# ik分词器, 粗粒度分词
# ik_smart
GET /_analyze
{
  "analyzer": "ik_smart",
  "text": "乒乓球明年总冠军"
}
```

执行效果如下：

```json
{
  "tokens" : [
    {
      "token" : "乒乓球",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "明年",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "总冠军",
      "start_offset" : 5,
      "end_offset" : 8,
      "type" : "CN_WORD",
      "position" : 2
    }
  ]
}

```

由此可见  使用ik_smart可以将文本"text": "乒乓球明年总冠军"分成了【乒乓球】【明年】【总冠军】

这样看的话，这样的分词效果达到了我们的要求。



#### 5.4 IK分词查询文档

- • 词条查询：term 
  - •  不对搜索内容进行分词查询,  只要搜索内容 和 分词列的分词 或者 非分词列内容完全匹配, 就能检索出来
- • 全文查询：match 
  - • 对搜索内容进行分词查询，搜索内容分词后的所有词汇，都能够跟分词列的分词100%匹对上，就能够检索出来
  - 说白了, 所有分词查询出来数据求交集。

```mysql
/*
	kibana页面
*/

# 删除索引
DELETE people_index

# 添加映射
# name无需分词 keyword
# address 需要ik分词 type:text
PUT people_index
{
  "mappings": {
    "properties": {
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        },
        "address": {
          "type": "text", 
          "analyzer": "ik_smart"
        }
 	 }
  }
}


# 查询索引
GET people_index

# 添加4个文档
POST people_index/_doc
{
  "name": "张三",
  "age": 23,
  "address": "北京回龙观"
}

POST people_index/_doc
{
  "name": "李四",
  "age": 23,
  "address": "黑龙江回龙观"
}

POST people_index/_doc
{
  "name": "王五",
  "age": 26,
  "address": "吉林回龙观"
}

POST people_index/_doc
{
  "name": "赵六",
  "age": 28,
  "address": "南京雨花台区"
}

##查询文档
GET people_index/_search
#词条查询：term
#不对搜索内容分词查询, 只要搜索内容 和 分词列的分词 或者 非分词列内容完全匹配, 才能检索出来
GET people_index/_search
{
   "query": {
     "term": {
       "address": {
         "value": "北京"
       }
     }
   }
}

#全文查询：match
#先将 “北京有个回龙观” 进行分词, 根据分词查询所有, 并集返回结果
GET people_index/_search
{
  "query": {
     "match":{
      	"address": "北京有个回龙观"
     }
  }
}

```



### 6. ElasticSearch JavaAPI

#### 6.1 操作索引&文档

• 添加索引 • 查询索引 • 删除索引 • 判断索引是否存在

• 添加文档 • 修改文档 • 根据id查询文档 • 删除文档



##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.itheima</groupId>
    <artifactId>elasticsearch-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>elasticsearch-demo</name>
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
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!--引入es的坐标-->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.9.3</version>
<!--            <version>7.4.0</version>-->
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-client</artifactId>
            <version>7.9.3</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>7.9.3</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.4</version>
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
elasticsearch:
  host: 127.0.0.1 #192.168.149.135
  port: 9200

```

##### ElasticSearchConfig.java

```java
package com.itheima.elasticsearchdemo.config;

import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConfigurationProperties(prefix = "elasticsearch")
public class ElasticSearchConfig {
    private String host;
    private int port;

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

    @Bean
    public RestHighLevelClient client(){
        return new RestHighLevelClient(RestClient.builder(
                new HttpHost(
                        host,
                        port,
                        "http"
                )
        ));
    }
}

```

##### Person.java

```java
package com.itheima.domain;

public class Person {
    private String id;
    private String name;
    private int age;
    private String address;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

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

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", address='" + address + '\'' +
                '}';
    }
}

```

##### ElasticsearchDemoApplication.java

```java
package com.itheima.elasticsearchdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ElasticsearchDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(ElasticsearchDemoApplication.class, args);
    }

}
```

##### ElasticsearchDemoApplicationTests.java

```java
package com.itheima.elasticsearchdemo;

import com.alibaba.fastjson.JSON;
import com.itheima.domain.Person;
import org.apache.http.HttpHost;
import org.elasticsearch.action.admin.indices.delete.DeleteIndexRequest;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.delete.DeleteResponse;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.action.support.master.AcknowledgedResponse;
import org.elasticsearch.client.IndicesClient;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.CreateIndexRequest;
import org.elasticsearch.client.indices.CreateIndexResponse;
import org.elasticsearch.client.indices.GetIndexRequest;
import org.elasticsearch.client.indices.GetIndexResponse;
import org.elasticsearch.cluster.metadata.MappingMetadata;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

@SpringBootTest(classes = ElasticsearchDemoApplication.class)
class ElasticsearchDemoApplicationTests {

    @Autowired
    private RestHighLevelClient client;

    @Test
    void contextLoads() {
       /* //1.创建ES客户端对象
        RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(
           new HttpHost(
                   "192.168.149.135",
                   9200,
                   "http"
           )
        ));*/

        System.out.println(client);


    }


    /**
     * 添加索引
     */
    @Test
    public void addIndex() throws IOException {
        //1.使用client获取操作索引的对象
        IndicesClient indicesClient = client.indices();
        //2.具体操作，获取返回值
        CreateIndexRequest createRequest = new CreateIndexRequest("itheima");
        CreateIndexResponse response = indicesClient.create(createRequest, RequestOptions.DEFAULT);

        //3.根据返回值判断结果
        System.out.println(response.isAcknowledged());



    }


    /**
     * 添加索引
     */
    @Test
    public void addIndexAndMapping() throws IOException {
        //1.使用client获取操作索引的对象
        IndicesClient indicesClient = client.indices();
        //2.具体操作，获取返回值
        CreateIndexRequest createRequest = new CreateIndexRequest("itcast");
        //2.1 设置mappings
        String mapping = "{\n" +
                "      \"properties\" : {\n" +
                "        \"address\" : {\n" +
                "          \"type\" : \"text\",\n" +
                "          \"analyzer\" : \"ik_max_word\"\n" +
                "        },\n" +
                "        \"age\" : {\n" +
                "          \"type\" : \"long\"\n" +
                "        },\n" +
                "        \"name\" : {\n" +
                "          \"type\" : \"keyword\"\n" +
                "        }\n" +
                "      }\n" +
                "    }";
        createRequest.mapping(mapping,XContentType.JSON);


        CreateIndexResponse response = indicesClient.create(createRequest, RequestOptions.DEFAULT);

        //3.根据返回值判断结果
        System.out.println(response.isAcknowledged());



    }


    /**
     * 查询索引
     */
    @Test
    public void queryIndex() throws IOException {
        IndicesClient indices = client.indices();

        GetIndexRequest getReqeust = new GetIndexRequest("itcast");
        GetIndexResponse response = indices.get(getReqeust, RequestOptions.DEFAULT);


        //获取结果
        Map<String, MappingMetadata> mappings = response.getMappings();
        for (String key : mappings.keySet()) {
            System.out.println(key+":" + mappings.get(key).getSourceAsMap());

        }

    }


    /**
     * 删除索引
     */
    @Test
    public void deleteIndex() throws IOException {
        IndicesClient indices = client.indices();

        DeleteIndexRequest deleteRequest = new DeleteIndexRequest("itheima");
        AcknowledgedResponse response = indices.delete(deleteRequest, RequestOptions.DEFAULT);

        System.out.println(response.isAcknowledged());

    }

    /**
     * 判断索引是否存在
     */
    @Test
    public void existIndex() throws IOException {
        IndicesClient indices = client.indices();

        GetIndexRequest getRequest = new GetIndexRequest("itcast");
        boolean exists = indices.exists(getRequest, RequestOptions.DEFAULT);

        System.out.println(exists);

    }


    /**
     * 添加文档,使用map作为数据
     */
    @Test
    public void addDoc() throws IOException {
        //数据对象，map
        Map data = new HashMap();
        data.put("address","北京昌平");
        data.put("name","大胖");
        data.put("age",20);


        //1.获取操作文档的对象
        IndexRequest request = new IndexRequest("itcast").id("1").source(data);
        //添加数据，获取结果
        IndexResponse response = client.index(request, RequestOptions.DEFAULT);

        //打印响应结果
        System.out.println(response.getId());


    }


    /**
     * 添加文档,使用对象作为数据
     */
    @Test
    public void addDoc2() throws IOException {
        //数据对象，javaObject
        Person p = new Person();
        p.setId("2");
        p.setName("小胖2222");
        p.setAge(30);
        p.setAddress("陕西西安");

        //将对象转为json
        String data = JSON.toJSONString(p);

        //1.获取操作文档的对象
        IndexRequest request = new IndexRequest("itcast").id(p.getId()).source(data,XContentType.JSON);
        //添加数据，获取结果
        IndexResponse response = client.index(request, RequestOptions.DEFAULT);

        //打印响应结果
        System.out.println(response.getId());


    }


    /**
     * 修改文档：添加文档时，如果id存在则修改，id不存在则添加
     */
    @Test
    public void updateDoc() throws IOException {

    }



    /**
     * 根据id查询文档
     */
    @Test
    public void findDocById() throws IOException {

        GetRequest getReqeust = new GetRequest("itcast","1");
        //getReqeust.id("1");
        GetResponse response = client.get(getReqeust, RequestOptions.DEFAULT);
        //获取数据对应的json
        System.out.println(response.getSourceAsString());


    }



    /**
     * 根据id删除文档
     */
    @Test
    public void delDoc() throws IOException {


        DeleteRequest deleteRequest = new DeleteRequest("itcast","1");
        DeleteResponse response = client.delete(deleteRequest, RequestOptions.DEFAULT);
        System.out.println(response.getId());


    }

}

```

