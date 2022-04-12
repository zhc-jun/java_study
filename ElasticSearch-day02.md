## ElasticSearch-day02

### 今日目标

- 能够将数据从MySQL导入到ElasticSearch中
- 多加练习：JavaAPI对ES数据各种查询





### 7. ElasticSearch 高级操作



#### 7.1 基础代码



##### application.yml

```yaml
elasticsearch:
  host: 127.0.0.1  #192.168.149.135
  port: 9200


# datasource
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/elasticsearch?serverTimezone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver


# mybatis
mybatis:
  mapper-locations: classpath:mapper/*Mapper.xml # mapper映射文件路径
  type-aliases-package: com.itheima.elasticsearchdemo2.domain


```

##### ElasticSearchConfig.java

```java
package com.itheima.elasticsearchdemo2.config;

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

##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.8.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.itcast</groupId>
    <artifactId>esearch-demo2</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!--引入es的坐标-->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.9.3</version>
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

        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <!--<scope>runtime</scope>-->
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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

##### Goods.java

```java
package com.itheima.elasticsearchdemo2.domain;

import com.alibaba.fastjson.annotation.JSONField;

import java.util.Date;
import java.util.Map;

public class Goods {

    private int id;
    private String title;
    private double price;
    private int stock;
    private int saleNum;
    private Date createTime;
    private String categoryName;
    private String brandName;
    private Map spec;

    @JSONField(serialize = false)//在转换JSON时，忽略该字段
    private String specStr;//接收数据库的信息 "{}"


    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    public int getStock() {
        return stock;
    }

    public void setStock(int stock) {
        this.stock = stock;
    }

    public int getSaleNum() {
        return saleNum;
    }

    public void setSaleNum(int saleNum) {
        this.saleNum = saleNum;
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

    public String getCategoryName() {
        return categoryName;
    }

    public void setCategoryName(String categoryName) {
        this.categoryName = categoryName;
    }

    public String getBrandName() {
        return brandName;
    }

    public void setBrandName(String brandName) {
        this.brandName = brandName;
    }

    public Map getSpec() {
        return spec;
    }

    public void setSpec(Map spec) {
        this.spec = spec;
    }

    public String getSpecStr() {
        return specStr;
    }

    public void setSpecStr(String specStr) {
        this.specStr = specStr;
    }


    @Override
    public String toString() {
        return "Goods{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", price=" + price +
                ", stock=" + stock +
                ", saleNum=" + saleNum +
                ", createTime=" + createTime +
                ", categoryName='" + categoryName + '\'' +
                ", brandName='" + brandName + '\'' +
                ", spec=" + spec +
                ", specStr='" + specStr + '\'' +
                '}';
    }
}

```









#### 7.2 批量操作

- Bulk 批量操作是将文档的增删改查一些列操作，通过一次请求全都做完。减少网络传输次数。

##### 脚本

```mysql
/*
	kibana页面
*/

# 批量操作
# 1.删除5号记录
# 2.添加8号记录
# 3.修改2号记录 名称为 “二号”

POST _bulk
{"delete": {"_index": "person", "_id": "5"}}
{"create": {"_index": "person", "_id": "8"}}
{"name": "葫芦娃", "age", 80, "address": "北京"}}
{"update": {"_index": "person", "_id": "8"}}
{"doc": {"name": "二号"}}


```

##### JavaAPI

ElasticsearchDemo2ApplicationTests.java

```java
package com.itheima.elasticsearchdemo2;

import com.alibaba.fastjson.JSON;
import com.itheima.elasticsearchdemo2.domain.Goods;
import com.itheima.elasticsearchdemo2.mapper.GoodsMapper;
import org.elasticsearch.action.DocWriteRequest;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.text.Text;
import org.elasticsearch.common.xcontent.XContentType;
import org.elasticsearch.index.query.*;
import org.elasticsearch.rest.RestStatus;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.elasticsearch.search.aggregations.Aggregation;
import org.elasticsearch.search.aggregations.AggregationBuilder;
import org.elasticsearch.search.aggregations.AggregationBuilders;
import org.elasticsearch.search.aggregations.Aggregations;
import org.elasticsearch.search.aggregations.bucket.terms.Terms;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightField;
import org.elasticsearch.search.sort.SortOrder;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@SpringBootTest(classes = ElasticsearchDemo2Application.class)
class ElasticsearchDemo2ApplicationTests {

    @Autowired
    private RestHighLevelClient client;

    /**
     * 1. 批量操作 bulk
     */
    @Test
    public void testBulk() throws IOException {

        //创建bulkrequest对象，整合所有操作
        BulkRequest bulkRequest = new BulkRequest();

        /*
        # 1. 删除1号记录
        # 2. 添加6号记录
        # 3. 修改3号记录 名称为 “三号”
         */
        //添加对应操作
        //1. 删除1号记录
        DeleteRequest deleteRequest = new DeleteRequest("person","1");
        bulkRequest.add(deleteRequest);

        //2. 添加6号记录
        Map map = new HashMap();
        map.put("name","六号");
        IndexRequest indexRequest = new IndexRequest("person").id("6").source(map);
        bulkRequest.add(indexRequest);


        Map map2 = new HashMap();
        map2.put("name","三号");
        //3. 修改3号记录 名称为 “三号”
        UpdateRequest updateReqeust = new UpdateRequest("person","3").doc(map2);
        bulkRequest.add(updateReqeust);
        

        //执行批量操作
        BulkResponse response = client.bulk(bulkRequest, RequestOptions.DEFAULT);
        RestStatus status = response.status();
        System.out.println(status);

    }


}
```



#### 7.2 导入数据

- 需求：将数据库中Goods表的数据导入到ElasticSearch中

- ① 创建goods索引 

  ② 查询Goods表数据 

  ③ 批量添加到ElasticSearch中



##### 创建索引

- title:商品标题
- price:商品价格
- createTime:创建时间
- categoryName:分类名称。如：家电，手机
- brandName:品牌名称。如：华为，小米
- spec: 商品规格。如： spec:{"屏幕尺寸","5寸"，"内存大小","128G"}
- saleNum:销量
- stock:库存量

```json
PUT goods
{
	"mappings": {
		"properties": {
			"title": {
				"type": "text",
				"analyzer": "ik_smart"
			},
			"price": { 
				"type": "double"
			},
			"createTime": {
				"type": "date"
			},
			"categoryName": {	
				"type": "keyword"
			},
			"brandName": {	
				"type": "keyword"
			},
	
			"spec": {		
				"type": "object"
			},
			"saleNum": {	
				"type": "integer"
			},
			
			"stock": {	
				"type": "integer"
			}
		}
	}
}
```

##### 添加文档

- 测试

```json
POST goods/_doc/1
{
  "title":"小米手机",
  "price":1000,
  "createTime":"2019-12-01",
  "categoryName":"手机",
  "brandName":"小米",
  "saleNum":3000,
  "stock":10000,
  "spec":{
    "网络制式":"移动4G",
    "屏幕尺寸":"4.5"
  }
}
```

##### ElasticsearchDemo2ApplicationTests.java

```java
package com.itheima.elasticsearchdemo2;

import com.alibaba.fastjson.JSON;
import com.itheima.elasticsearchdemo2.domain.Goods;
import com.itheima.elasticsearchdemo2.mapper.GoodsMapper;
import org.elasticsearch.action.DocWriteRequest;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.text.Text;
import org.elasticsearch.common.xcontent.XContentType;
import org.elasticsearch.index.query.*;
import org.elasticsearch.rest.RestStatus;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.elasticsearch.search.aggregations.Aggregation;
import org.elasticsearch.search.aggregations.AggregationBuilder;
import org.elasticsearch.search.aggregations.AggregationBuilders;
import org.elasticsearch.search.aggregations.Aggregations;
import org.elasticsearch.search.aggregations.bucket.terms.Terms;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightField;
import org.elasticsearch.search.sort.SortOrder;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@SpringBootTest(classes = ElasticsearchDemo2Application.class)
class ElasticsearchDemo2ApplicationTests {

    @Autowired
    private RestHighLevelClient client;
    
    @Autowired
    private GoodsMapper goodsMapper;

    /**
     * 批量导入
     */
    @Test
    public void importData() throws IOException {
        //1.查询所有数据，mysql
        List<Goods> goodsList = goodsMapper.findAll();

        //System.out.println(goodsList.size());
        //2.bulk导入
        BulkRequest bulkRequest = new BulkRequest();

        //2.1 循环goodsList，创建IndexRequest添加数据
        for (Goods goods : goodsList) {
            //2.2 设置spec规格信息 Map的数据   specStr:{}
            //goods.setSpec(JSON.parseObject(goods.getSpecStr(),Map.class));

            String specStr = goods.getSpecStr();
            //将json格式字符串转为Map集合
            Map map = JSON.parseObject(specStr, Map.class);
            //设置spec map
            goods.setSpec(map);
            //将goods对象转换为json字符串
            String data = JSON.toJSONString(goods);//map --> {}
            IndexRequest indexRequest = new IndexRequest("goods");
            indexRequest.id(goods.getId()+"").source(data, XContentType.JSON);
            bulkRequest.add(indexRequest);
        }

        BulkResponse response = client.bulk(bulkRequest, RequestOptions.DEFAULT);
        System.out.println(response.status());


    }

}
```













#### 7.3 查询操作

##### 脚本

```mysql
/*
	kibana页面
*/


# 默认情况下，es一次展示10条数据,通过from和size来控制分页
# 查询结果详解
GET goods/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 100
}

GET goods

# term查询
GET goods/_search
{
  "query": {
    "term": {
      "title": {
        "value": "华为"
      }
    }
  }
}


# match查询
GET goods/_search
{
  "query": {
    "match": {
      "title": "华为手机"
    }
  },
  "size": 500
}



GET goods/_search
{
  "query": {
    "match": {
      "title": {
        "query":  "华为手机",
        "operator": "and"
      }
    }
  }
}
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "施华洛世奇"
}


# wildcard 查询。查询条件分词，模糊查询
GET goods/_search
{
  "query": {
    "wildcard": {
      "title": {
        "value": "华*"
      }
    }
  }
}

# 正则查询
GET goods/_search
{
  "query": {
    "regexp": {
      "title": "\\w+(.)*"
    }
  }
}

# 前缀查询
GET goods/_search
{
  "query": {
    "prefix": {
      "brandName": {
        "value": "三"
      }
    }
  }
}


# 范围查询

GET goods/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 2000,
        "lte": 3000
      }
    }
  },
  "sort": [
    {
      "price": {
        "order": "desc"
      }
    }
  ]
}

# queryString

GET goods/_search
{
  "query": {
    "query_string": {
      "fields": ["title","categoryName","brandName"], 
      "query": "华为 AND 手机"
    }
  }
}


GET goods/_search
{
  "query": {
    "simple_query_string": {
      "fields": ["title","categoryName","brandName"], 
      "query": "华为 AND 手机"
    }
  }
}



# boolquery

GET goods/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "brandName": {
              "value": "华为"
            }
          }
        }
      ],
      "filter":[ 
        {
        "term": {
          "title": "手机"
        }
       },
       {
         "range":{
          "price": {
            "gte": 2000,
            "lte": 3000
         }
         }
       }
      
      ]
    }
  }
}

GET goods/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "brandName": {
              "value": "华为"
            }
          }
        }
      ]
    }
  }
}




# 聚合查询

# 指标聚合 聚合函数

GET goods/_search
{
  "query": {
    "match": {
      "title": "手机"
    }
  },
  "aggs": {
    "max_price": {
      "max": {
        "field": "price"
      }
    }
  }
}

# 桶聚合  分组

GET goods/_search
{
  "query": {
    "match": {
      "title": "手机"
    }
  },
  "aggs": {
    "goods_brands": {
      "terms": {
        "field": "brandName",
        "size": 100
      }
    }
  }
}



GET goods/_search
{
  "query": {
    "match": {
      "title": "电视"
    }
  },
  "highlight": {
    "fields": {
      "title": {
        "pre_tags": "<font color='red'>",
        "post_tags": "</font>"
      }
    }
  }
}


# -------重建索引-----------

# 新建student_index_v1。索引名称必须全部小写

PUT student_index_v1
{
  "mappings": {
    "properties": {
      "birthday":{
        "type": "date"
      }
    }
  }
}

GET student_index_v1

PUT student_index_v1/_doc/1
{
  "birthday":"1999-11-11"
}


GET student_index_v1/_search


PUT student_index_v1/_doc/1
{
  "birthday":"1999年11月11日"
}

# 业务变更了，需要改变birthday字段的类型为text

# 1. 创建新的索引 student_index_v2
# 2. 将student_index_v1 数据拷贝到 student_index_v2

# 创建新的索引 student_index_v2
PUT student_index_v2
{
  "mappings": {
    "properties": {
      "birthday":{
        "type": "text"
      }
    }
  }
}
# 将student_index_v1 数据拷贝到 student_index_v2
# _reindex 拷贝数据
POST _reindex
{
  "source": {
    "index": "student_index_v1"
  },
  "dest": {
    "index": "student_index_v2"
  }
}

GET student_index_v2/_search



PUT student_index_v2/_doc/2
{
  "birthday":"1999年11月11日"
}



# 思考： 现在java代码中操作es，还是使用的实student_index_v1老的索引名称。
# 1. 改代码（不推荐）
# 2. 索引别名（推荐）

# 步骤：
# 0. 先删除student_index_v1
# 1. 给student_index_v2起个别名 student_index_v1



# 先删除student_index_v1
DELETE student_index_v1
# 给student_index_v2起个别名 student_index_v1
POST student_index_v2/_alias/student_index_v1


GET student_index_v1/_search
GET student_index_v2/_search


```



##### JavaAPI

```java
package com.itheima.elasticsearch2;

import com.alibaba.fastjson.JSON;
import com.itheima.elasticsearch2.domain.Goods;
import com.itheima.elasticsearch2.mapper.GoodsMapper;
import org.elasticsearch.action.DocWriteRequest;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.text.Text;
import org.elasticsearch.common.xcontent.XContentType;
import org.elasticsearch.index.query.*;
import org.elasticsearch.rest.RestStatus;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.elasticsearch.search.aggregations.Aggregation;
import org.elasticsearch.search.aggregations.AggregationBuilder;
import org.elasticsearch.search.aggregations.AggregationBuilders;
import org.elasticsearch.search.aggregations.Aggregations;
import org.elasticsearch.search.aggregations.bucket.terms.Terms;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightField;
import org.elasticsearch.search.sort.SortOrder;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@RunWith(SpringRunner.class)
@SpringBootTest
public class ElasticsearchDemo2ApplicationTests {

    @Autowired
    private RestHighLevelClient client;

    @Autowired
    private GoodsMapper goodsMapper;

    /**
     * 1. 批量操作 bulk
     */
   /* @Test
    public void testBulk() throws IOException {

        System.out.println("测试.....");
        System.out.println(client);
        System.out.println(goodsMapper);

    }*/

    /**
     * 1. 批量操作 bulk
     */
    @Test
    public void testBulk() throws IOException {
        //创建bulkrequest对象，整合所有操作
        BulkRequest bulkRequest = new BulkRequest();

        /*
        # 1. 删除1号记录
        # 2. 添加6号记录
        # 3. 修改3号记录 名称为 “三号”
         */
        //添加对应操作
        //1. 删除1号记录
        DeleteRequest deleteRequest = new DeleteRequest("person","1");
        bulkRequest.add(deleteRequest);

        //2. 添加6号记录
        Map map = new HashMap();
        map.put("name","六号");
        IndexRequest indexRequest = new IndexRequest("person").id("6").source(map);
        bulkRequest.add(indexRequest);


        Map map2 = new HashMap();
        map2.put("name","三号");
        //3. 修改3号记录 名称为 “三号”
        UpdateRequest updateReqeust = new UpdateRequest("person","3").doc(map2);
        bulkRequest.add(updateReqeust);




        //执行批量操作
        BulkResponse response = client.bulk(bulkRequest, RequestOptions.DEFAULT);
        RestStatus status = response.status();
        System.out.println(status);

    }


    /**
     * 批量导入
     */
    @Test
    public void importData() throws IOException {
        //1.查询所有数据，mysql
        List<Goods> goodsList = goodsMapper.findAll();

        //System.out.println(goodsList.size());
        //2.bulk导入
        BulkRequest bulkRequest = new BulkRequest();

        //2.1 循环goodsList，创建IndexRequest添加数据
        for (Goods goods : goodsList) {
            //2.2 设置spec规格信息 Map的数据   specStr:{}
            //goods.setSpec(JSON.parseObject(goods.getSpecStr(),Map.class));

            String specStr = goods.getSpecStr();
            //将json格式字符串转为Map集合
            Map map = JSON.parseObject(specStr, Map.class);
            //设置spec map
            goods.setSpec(map);
            //将goods对象转换为json字符串
            String data = JSON.toJSONString(goods);//map --> {}
            IndexRequest indexRequest = new IndexRequest("goods");
            indexRequest.id(goods.getId()+"").source(data, XContentType.JSON);
            bulkRequest.add(indexRequest);
        }


        BulkResponse response = client.bulk(bulkRequest, RequestOptions.DEFAULT);
        System.out.println(response.status());


    }

    /**
     * 查询所有
     *  1. matchAll
     *  2. 将查询结果封装为Goods对象，装载到List中
     *  3. 分页。默认显示10条
     */
    @Test
    public void testMatchAll() throws IOException {
        //2. 构建查询请求对象，指定查询的索引名称
        SearchRequest searchRequest = new SearchRequest("goods");
        //4. 创建查询条件构建器SearchSourceBuilder
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

        //6. 查询条件
        QueryBuilder query = QueryBuilders.matchAllQuery();//查询所有文档
        //5. 指定查询条件
        sourceBuilder.query(query);

        //3. 添加查询条件构建器 SearchSourceBuilder
        searchRequest.source(sourceBuilder);

        // 8 . 添加分页信息
        sourceBuilder.from(0);
        sourceBuilder.size(100);

        //1. 查询,获取查询结果
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        //7. 获取命中对象 SearchHits
        SearchHits searchHits = searchResponse.getHits();
        //7.1 获取总记录数
        long value = searchHits.getTotalHits().value;
        System.out.println("总记录数："+value);


        List<Goods> goodsList = new ArrayList<>();
        //7.2 获取Hits数据  数组
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            //获取json字符串格式的数据
            String sourceAsString = hit.getSourceAsString();
            //转为java对象
            Goods goods = JSON.parseObject(sourceAsString, Goods.class);

            goodsList.add(goods);

        }

        for (Goods goods : goodsList) {
            System.out.println(goods);
        }
    }


    /**
     * termQuery:词条查询
     */
    @Test
    public void testTermQuery() throws IOException {

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder sourceBulider = new SearchSourceBuilder();

        QueryBuilder query = QueryBuilders.termQuery("title","华为");//term词条查询
        sourceBulider.query(query);

        searchRequest.source(sourceBulider);

        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        SearchHits searchHits = searchResponse.getHits();
        //获取记录数
        long value = searchHits.getTotalHits().value;
        System.out.println("总记录数："+value);

        List<Goods> goodsList = new ArrayList<>();
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();

            //转为java
            Goods goods = JSON.parseObject(sourceAsString, Goods.class);

            goodsList.add(goods);
        }

        for (Goods goods : goodsList) {
            System.out.println(goods);
        }
    }


    /**
     * matchQuery:词条分词查询
     */
    @Test
    public void testMatchQuery() throws IOException {

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder sourceBulider = new SearchSourceBuilder();

        MatchQueryBuilder query = QueryBuilders.matchQuery("title", "华为手机");
        query.operator(Operator.AND);//求并集
        sourceBulider.query(query);

        searchRequest.source(sourceBulider);

        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        SearchHits searchHits = searchResponse.getHits();
        //获取记录数
        long value = searchHits.getTotalHits().value;
        System.out.println("总记录数："+value);

        List<Goods> goodsList = new ArrayList<>();
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();

            //转为java
            Goods goods = JSON.parseObject(sourceAsString, Goods.class);

            goodsList.add(goods);
        }

        for (Goods goods : goodsList) {
            System.out.println(goods);
        }
    }


    /**
     * 模糊查询:WildcardQuery
     */
    @Test
    public void testWildcardQuery() throws IOException {

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder sourceBulider = new SearchSourceBuilder();

        WildcardQueryBuilder query = QueryBuilders.wildcardQuery("title", "华*");

        sourceBulider.query(query);

        searchRequest.source(sourceBulider);

        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        SearchHits searchHits = searchResponse.getHits();
        //获取记录数
        long value = searchHits.getTotalHits().value;
        System.out.println("总记录数："+value);

        List<Goods> goodsList = new ArrayList<>();
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();

            //转为java
            Goods goods = JSON.parseObject(sourceAsString, Goods.class);

            goodsList.add(goods);
        }

        for (Goods goods : goodsList) {
            System.out.println(goods);
        }
    }


    /**
     * 模糊查询:regexpQuery
     */
    @Test
    public void testRegexpQuery() throws IOException {

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder sourceBulider = new SearchSourceBuilder();

        RegexpQueryBuilder query = QueryBuilders.regexpQuery("title", "\\w+(.)*");

        sourceBulider.query(query);

        searchRequest.source(sourceBulider);


        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);


        SearchHits searchHits = searchResponse.getHits();
        //获取记录数
        long value = searchHits.getTotalHits().value;
        System.out.println("总记录数："+value);

        List<Goods> goodsList = new ArrayList<>();
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();

            //转为java
            Goods goods = JSON.parseObject(sourceAsString, Goods.class);

            goodsList.add(goods);
        }

        for (Goods goods : goodsList) {
            System.out.println(goods);
        }
    }


    /**
     * 模糊查询:perfixQuery
     */
    @Test
    public void testPrefixQuery() throws IOException {

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder sourceBulider = new SearchSourceBuilder();

        PrefixQueryBuilder query = QueryBuilders.prefixQuery("brandName", "三");

        sourceBulider.query(query);

        searchRequest.source(sourceBulider);

        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        SearchHits searchHits = searchResponse.getHits();
        //获取记录数
        long value = searchHits.getTotalHits().value;
        System.out.println("总记录数："+value);

        List<Goods> goodsList = new ArrayList<>();
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();

            //转为java
            Goods goods = JSON.parseObject(sourceAsString, Goods.class);

            goodsList.add(goods);
        }

        for (Goods goods : goodsList) {
            System.out.println(goods);
        }
    }


    /**
     * 1. 范围查询：rangeQuery
     * 2. 排序
     */
    @Test
    public void testRangeQuery() throws IOException {

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder sourceBulider = new SearchSourceBuilder();


        //范围查询
        RangeQueryBuilder query = QueryBuilders.rangeQuery("price");

        //指定下限
        query.gte(2000);
        //指定上限
        query.lte(3000);

        sourceBulider.query(query);

        //排序
        sourceBulider.sort("price", SortOrder.DESC);

        searchRequest.source(sourceBulider);

        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);


        SearchHits searchHits = searchResponse.getHits();
        //获取记录数
        long value = searchHits.getTotalHits().value;
        System.out.println("总记录数："+value);

        List<Goods> goodsList = new ArrayList<>();
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();

            //转为java
            Goods goods = JSON.parseObject(sourceAsString, Goods.class);

            goodsList.add(goods);
        }

        for (Goods goods : goodsList) {
            System.out.println(goods);
        }
    }


    /**
     * queryString
     */
    @Test
    public void testQueryStringQuery() throws IOException {

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder sourceBulider = new SearchSourceBuilder();

        //queryString
        QueryStringQueryBuilder query = QueryBuilders.queryStringQuery("华为手机").field("title").field("categoryName").field("brandName").defaultOperator(Operator.AND);

        sourceBulider.query(query);

        searchRequest.source(sourceBulider);

        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);


        SearchHits searchHits = searchResponse.getHits();
        //获取记录数
        long value = searchHits.getTotalHits().value;
        System.out.println("总记录数："+value);

        List<Goods> goodsList = new ArrayList<>();
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();

            //转为java
            Goods goods = JSON.parseObject(sourceAsString, Goods.class);

            goodsList.add(goods);
        }

        for (Goods goods : goodsList) {
            System.out.println(goods);
        }
    }


    /**
     * 布尔查询：boolQuery
     * 1. 查询品牌名称为:华为
     * 2. 查询标题包含：手机
     * 3. 查询价格在：2000-3000
     */
    @Test
    public void testBoolQuery() throws IOException {

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder sourceBulider = new SearchSourceBuilder();

        //1.构建boolQuery
        BoolQueryBuilder query = QueryBuilders.boolQuery();

        //2.构建各个查询条件
        //2.1 查询品牌名称为:华为
        QueryBuilder termQuery = QueryBuilders.termQuery("brandName","华为");
        query.must(termQuery);

        //2.2. 查询标题包含：手机
        QueryBuilder matchQuery = QueryBuilders.matchQuery("title","手机");
        query.filter(matchQuery);

        //2.3 查询价格在：2000-3000
        QueryBuilder rangeQuery = QueryBuilders.rangeQuery("price");
        ((RangeQueryBuilder) rangeQuery).gte(2000);
        ((RangeQueryBuilder) rangeQuery).lte(3000);
        query.filter(rangeQuery);

        //3.使用boolQuery连接

        sourceBulider.query(query);

        searchRequest.source(sourceBulider);

        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        SearchHits searchHits = searchResponse.getHits();
        //获取记录数
        long value = searchHits.getTotalHits().value;
        System.out.println("总记录数："+value);

        List<Goods> goodsList = new ArrayList<>();
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();

            //转为java
            Goods goods = JSON.parseObject(sourceAsString, Goods.class);

            goodsList.add(goods);
        }

        for (Goods goods : goodsList) {
            System.out.println(goods);
        }
    }


    /**
     * 聚合查询：桶聚合，分组查询
     * 1. 查询title包含手机的数据
     * 2. 查询品牌列表
     */
    @Test
    public void testAggQuery() throws IOException {

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder sourceBulider = new SearchSourceBuilder();

        // 1. 查询title包含手机的数据
        MatchQueryBuilder query = QueryBuilders.matchQuery("title", "手机");

        sourceBulider.query(query);

        // 2. 查询品牌列表
        /*
        参数：
            1. 自定义的名称，将来用于获取数据
            2. 分组的字段
         */
        AggregationBuilder agg = AggregationBuilders.terms("goods_brands").field("brandName").size(100);
        sourceBulider.aggregation(agg);

        searchRequest.source(sourceBulider);

        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        SearchHits searchHits = searchResponse.getHits();
        //获取记录数
        long value = searchHits.getTotalHits().value;
        System.out.println("总记录数："+value);

        List<Goods> goodsList = new ArrayList<>();
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();

            //转为java
            Goods goods = JSON.parseObject(sourceAsString, Goods.class);
            goodsList.add(goods);
        }

        for (Goods goods : goodsList) {
            System.out.println(goods);
        }

        // 获取聚合结果
        Aggregations aggregations = searchResponse.getAggregations();

        Map<String, Aggregation> aggregationMap = aggregations.asMap();

        //System.out.println(aggregationMap);
        Terms goods_brands = (Terms) aggregationMap.get("goods_brands");

        List<? extends Terms.Bucket> buckets = goods_brands.getBuckets();

        List brands = new ArrayList();
        for (Terms.Bucket bucket : buckets) {
            Object key = bucket.getKey();
            brands.add(key);
        }

        for (Object brand : brands) {
            System.out.println(brand);
        }

    }


    /**
     *
     * 高亮查询：
     *  1. 设置高亮
     *      * 高亮字段
     *      * 前缀
     *      * 后缀
     *  2. 将高亮了的字段数据，替换原有数据
     */
    @Test
    public void testHighLightQuery() throws IOException {


        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder sourceBulider = new SearchSourceBuilder();

        // 1. 查询title包含手机的数据
        MatchQueryBuilder query = QueryBuilders.matchQuery("title", "手机");

        sourceBulider.query(query);

        //设置高亮
        HighlightBuilder highlighter = new HighlightBuilder();
        //设置三要素
        highlighter.field("title");
        highlighter.preTags("<font color='red'>");
        highlighter.postTags("</font>");

        sourceBulider.highlighter(highlighter);

        // 2. 查询品牌列表
        /*
        参数：
            1. 自定义的名称，将来用于获取数据
            2. 分组的字段
         */
        AggregationBuilder agg = AggregationBuilders.terms("goods_brands").field("brandName").size(100);
        sourceBulider.aggregation(agg);

        searchRequest.source(sourceBulider);

        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        SearchHits searchHits = searchResponse.getHits();
        //获取记录数
        long value = searchHits.getTotalHits().value;
        System.out.println("总记录数："+value);

        List<Goods> goodsList = new ArrayList<>();
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();

            //转为java
            Goods goods = JSON.parseObject(sourceAsString, Goods.class);

            // 获取高亮结果，替换goods中的title
            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            HighlightField HighlightField = highlightFields.get("title");
            Text[] fragments = HighlightField.fragments();
            //替换
            goods.setTitle(fragments[0].toString());

            goodsList.add(goods);
        }

        for (Goods goods : goodsList) {
            System.out.println(goods);
        }

        // 获取聚合结果
        Aggregations aggregations = searchResponse.getAggregations();

        Map<String, Aggregation> aggregationMap = aggregations.asMap();

        //System.out.println(aggregationMap);
        Terms goods_brands = (Terms) aggregationMap.get("goods_brands");

        List<? extends Terms.Bucket> buckets = goods_brands.getBuckets();

        List brands = new ArrayList();
        for (Terms.Bucket bucket : buckets) {
            Object key = bucket.getKey();
            brands.add(key);
        }

        for (Object brand : brands) {
            System.out.println(brand);
        }

    }


}
```





### 8. ElasticSearch 集群

#### 8.1 搭建集群

Elasticsearch如果做集群的话Master节点至少三台服务器或者三个Master实例加入相同集群，三个Master节点最多只能故障一台Master节点，如果故障两个Master节点，Elasticsearch将无法组成集群.会报错，Kibana也无法启动，因为Kibana无法获取集群中的节点信息。

由于，我们使用只有一台虚拟机，所以我们在虚拟机中安装三个ES实例，搭建伪集群，而ES启动比较耗内存，所以先设置虚拟机的内存3G和CPU个数4个



![img](https://img-blog.csdnimg.cn/20201113221828675.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)



##### 8.1.1 整体步骤

步骤如下：

- 拷贝opt目录下的elasticsearch-7.4.0安装包3个，分别命名：

  elasticsearch-7.4.0-itcast1

  elasticsearch-7.4.0-itcast2

  elasticsearch-7.4.0-itcast3 

- 然后修改elasticsearch.yml文件件。 

- 然后启动启动itcast1、itcast2、itcast3三个节点。 

- 打开浏览器输⼊：http://192.168.149.135:9200/_cat/health?v ,如果返回的node.total是3，代表集 群搭建成功

**在此，需要我们特别注意的是，像本文这样单服务器多节点（ 3 个节点）的情况，仅供测试使用**，集群环境如下：

|              |           |                 |                      |
| ------------ | --------- | --------------- | -------------------- |
| cluster name | node name | IP Addr         | http端口  / 通信端口 |
| itcast-es    | itcast1   | 192.168.149.135 | 9201   /  9700       |
| itcast-es    | itcast2   | 192.168.149.135 | 9202   /   9800      |
| itcast-es    | itcast3   | 192.168.149.135 | 9203   /  9900       |

##### 8.1.2 拷贝副本

拷贝opt目录下的elasticsearch-7.4.0安装包3个，打开虚拟机到opt目录



执行 拷贝三份

```shell
cd /opt
cp -r  elasticsearch-7.4.0   elasticsearch-7.4.0-itcast1
cp -r  elasticsearch-7.4.0   elasticsearch-7.4.0-itcast2
cp -r  elasticsearch-7.4.0   elasticsearch-7.4.0-itcast3
```



##### 8.1. 3 elasticsearch.yml配置文件

**1)、创建日志目录**

```shell
cd /opt
mkdir logs
mkdir  data
# 授权给itheima用户
chown -R itheima:itheima ./logs
chown -R itheima:itheima ./data

chown -R itheima:itheima ./elasticsearch-7.4.0-itcast1
chown -R itheima:itheima ./elasticsearch-7.4.0-itcast2
chown -R itheima:itheima ./elasticsearch-7.4.0-itcast3
```

打开elasticsearch.yml配置，分别配置下面三个节点的配置文件

```shell
vim /opt/elasticsearch-7.4.0-itcast1/config/elasticsearch.yml 
vim /opt/elasticsearch-7.4.0-itcast2/config/elasticsearch.yml 
vim /opt/elasticsearch-7.4.0-itcast3/config/elasticsearch.yml 
```

**2)、下面是elasticsearch-7.4.0-itcast1配置文件**

```yml
cluster.name: itcast-es
node.name: itcast-1 
node.master: true
node.data: true
node.max_local_storage_nodes: 3 
network.host: 0.0.0.0
http.port: 9201
transport.tcp.port: 9700
discovery.seed_hosts: ["localhost:9700","localhost:9800","localhost:9900"]
cluster.initial_master_nodes: ["itcast-1", "itcast-2","itcast-3"]
path.data: /opt/data
path.logs: /opt/logs
```

如下为配置说明：

```yml
#集群名称
cluster.name: itcast-es
#节点名称
node.name: itcast-1 
#是不是有资格主节点
node.master: true
#是否存储数据
node.data: true
#最大集群节点数
node.max_local_storage_nodes: 3 
#ip地址
network.host: 0.0.0.0
#端口
http.port: 9201
#内部节点之间沟通端口
transport.tcp.port: 9700
#es7.x 之后新增的配置，节点发现
discovery.seed_hosts: ["localhost:9700","localhost:9800","localhost:9900"]
#es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举master
cluster.initial_master_nodes: ["itcast-1", "itcast-2","itcast-3"] 
#数据和存储路径
path.data: /opt/data
path.logs: /opt/logs
```



**3)、下面是elasticsearch-7.4.0-itcast2配置文件**

```yml
cluster.name: itcast-es
node.name: itcast-2 
node.master: true
node.data: true
node.max_local_storage_nodes: 3 
network.host: 0.0.0.0
http.port: 9202
transport.tcp.port: 9800
discovery.seed_hosts: ["localhost:9700","localhost:9800","localhost:9900"]
cluster.initial_master_nodes: ["itcast-1", "itcast-2","itcast-3"]
path.data: /opt/data
path.logs: /opt/logs

```



```yml
#集群名称
cluster.name: itcast-es
#节点名称
node.name: itcast-2 
#是不是有资格主节点
node.master: true
#是否存储数据
node.data: true
#最大集群节点数
node.max_local_storage_nodes: 3 
#ip地址
network.host: 0.0.0.0
#端口
http.port: 9202
#内部节点之间沟通端口
transport.tcp.port: 9800
#es7.x 之后新增的配置，节点发现
discovery.seed_hosts: ["localhost:9700","localhost:9800","localhost:9900"]
#es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举master
cluster.initial_master_nodes: ["itcast-1", "itcast-2","itcast-3"] 
#数据和存储路径
path.data: /opt/data
path.logs: /opt/logs
```

**4)、下面是elasticsearch-7.4.0-itcast3 配置文件**

```yml

cluster.name: itcast-es
node.name: itcast-3 
node.master: true
node.data: true
node.max_local_storage_nodes: 3 
network.host: 0.0.0.0
http.port: 9203
transport.tcp.port: 9900
discovery.seed_hosts: ["localhost:9700","localhost:9800","localhost:9900"]
cluster.initial_master_nodes: ["itcast-1", "itcast-2","itcast-3"] 
path.data: /opt/data
path.logs: /opt/logs
```



```yml
#集群名称
cluster.name: itcast-es
#节点名称
node.name: itcast-3 
#是不是有资格主节点
node.master: true
#是否存储数据
node.data: true
#最大集群节点数
node.max_local_storage_nodes: 3 
#ip地址
network.host: 0.0.0.0
#端口
http.port: 9203
#内部节点之间沟通端口
transport.tcp.port: 9900
#es7.x 之后新增的配置，节点发现
discovery.seed_hosts: ["localhost:9700","localhost:9800","localhost:9900"]
#es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举master
cluster.initial_master_nodes: ["itcast-1", "itcast-2","itcast-3"] 
#数据和存储路径
path.data: /opt/data
path.logs: /opt/logs
```

##### 8.1.4  执行授权

```shell
在root用户下执行
chown -R itheima:itheima /opt/elasticsearch-7.4.0-itcast1
chown -R itheima:itheima /opt/elasticsearch-7.4.0-itcast2
chown -R itheima:itheima /opt/elasticsearch-7.4.0-itcast3
如果有的日志文件授权失败，可使用(也是在root下执行)
cd /opt/elasticsearch-7.4.0-itcast1/logs
chown -R itheima:itheima ./* 
cd /opt/elasticsearch-7.4.0-itcast2/logs
chown -R itheima:itheima ./* 
cd /opt/elasticsearch-7.4.0-itcast3/logs
chown -R itheima:itheima ./* 
```

##### 8.1.5 启动三个节点

启动之前，设置ES的JVM占用内存参数，防止内存不足错误

![img](https://img-blog.csdnimg.cn/20201113221906843.png)



```shell
vim /opt/elasticsearch-7.4.0-itcast1/bin/elasticsearch
```

![img](https://img-blog.csdnimg.cn/20201113221945949.png)



可以发现，ES启动时加载/config/jvm.options文件

```shell
vim /opt/elasticsearch-7.4.0-itcast1/config/jvm.options
```

![img](https://img-blog.csdnimg.cn/20201113222017622.png)



 默认情况下，ES启动JVM最小内存1G，最大内存1G

```shell
-xms:最小内存
-xmx:最大内存
```

修改为256m

![img](https://img-blog.csdnimg.cn/2020111322203810.png)



启动成功访问节点一：

![img](https://img-blog.csdnimg.cn/2020111322210141.png)



可以从日志中看到：master not discovered yet。还没有发现主节点

访问集群状态信息 http://192.168.149.135:9201/_cat/health?v 不成功

![img](https://img-blog.csdnimg.cn/20201113222127777.png)





启动成功访问节点二:

![img](https://img-blog.csdnimg.cn/20201113222155312.png)





可以从日志中看到：master not discovered yet。还没有发现主节点master node changed.已经选举出主节点itcast-2

访问集群状态信息 http://192.168.149.135:9201/_cat/health?v  成功

![img](https://img-blog.csdnimg.cn/20201113222217570.png)





```tex
健康状况结果解释：

cluster 集群名称
status 集群状态 
	green代表健康；
	yellow代表分配了所有主分片，但至少缺少一个副本，此时集群数据仍旧完整；
	red 代表部分主分片不可用，可能已经丢失数据。
node.total代表在线的节点总数量
node.data代表在线的数据节点的数量
shards 存活的分片数量
pri 存活的主分片数量 正常情况下 shards的数量是pri的两倍。
relo迁移中的分片数量，正常情况为 0
init 初始化中的分片数量 正常情况为 0
unassign未分配的分片 正常情况为 0
pending_tasks准备中的任务，任务指迁移分片等 正常情况为 0
max_task_wait_time任务最长等待时间
active_shards_percent正常分片百分比 正常情况为 100%
```



启动成功访问节点三

访问集群状态信息 http://192.168.149.135:9201/_cat/health?v  成功

![img](https://img-blog.csdnimg.cn/20201113222254512.png)





可以看到节点已经变为3个，至此，ES集群已经搭建成功~



#### 8.2  使用Kibana配置和管理集群

##### 8.2.1 集群配置

因为之前我们在单机演示的时候也使用到了Kibana，我们先复制出来一个Kibana，然后修改它的集群配置

```shell
cd /opt/
cp -r kibana-7.4.0-linux-x86_64   kibana-7.4.0-linux-x86_64-cluster
# 由于 kibana 中文件众多，此处会等待大约1分钟的时间
```

修改Kibana的集群配置

```shell
vim  kibana-7.4.0-linux-x86_64-cluster/config/kibana.yml
加入下面的配置
elasticsearch.hosts: ["http://localhost:9201","http://localhost:9202","http://localhost:9203"]
```

启动Kibana

```shell
sh kibana --allow-root
```

![img](https://img-blog.csdnimg.cn/2020111322232779.png)



##### 8.2.2 管理集群

1、打开Kibana，点开 Stack Monitoring 集群监控

![img](https://img-blog.csdnimg.cn/20201113222353523.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)





![img](https://img-blog.csdnimg.cn/20201113222432611.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)

2、点击【Nodes】查看节点详细信息

![img](https://img-blog.csdnimg.cn/20201113222450107.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)





![img](https://img-blog.csdnimg.cn/20201113222510136.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)



在上图可以看到，第一个红框处显示【Green】，绿色，表示集群处理健康状态

第二个红框是我们集群的三个节点，注意，itcast-3旁边是星星，表示是主节点



#### 8.3 JavaAPI操作集群

- java API想操作集群只需要更改配置，建立连接，增删改查均不变。
- springboot项目

##### application.yml

```yaml
#standonlne 单机192.168.149.135
elasticsearch:
  host: 127.0.0.1
  port: 9200

#cluster 集群配置
#elasticsearch:
#  host1: 127.0.0.1
#  port1: 9201
#  host2: 127.0.0.1
#  port2: 9202
#  host3: 127.0.0.1
#  port3: 9203


# datasource
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/elasticsearch?serverTimezone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver


# mybatis
mybatis:
  mapper-locations: classpath:mapper/*Mapper.xml # mapper映射文件路径
  type-aliases-package: com.itheima.elasticsearch2.domain

  # config-location:  # 指定mybatis的核心配置文件

```

##### ElasticSearchClusterConfig.java

```java
package com.itheima.elasticsearch2.config;

import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConfigurationProperties(prefix = "elasticsearch")
public class ElasticSearchClusterConfig {

    private String host1;
    private int port1;
    private String host2;
    private int port2;
    private String host3;
    private int port3;

    public String getHost1() {
        return host1;
    }

    public void setHost1(String host1) {
        this.host1 = host1;
    }

    public int getPort1() {
        return port1;
    }

    public void setPort1(int port1) {
        this.port1 = port1;
    }

    public String getHost2() {
        return host2;
    }

    public void setHost2(String host2) {
        this.host2 = host2;
    }

    public int getPort2() {
        return port2;
    }

    public void setPort2(int port2) {
        this.port2 = port2;
    }

    public String getHost3() {
        return host3;
    }

    public void setHost3(String host3) {
        this.host3 = host3;
    }

    public int getPort3() {
        return port3;
    }

    public void setPort3(int port3) {
        this.port3 = port3;
    }

    @Bean
    public RestHighLevelClient client(){
        return new RestHighLevelClient(RestClient.builder(
                new HttpHost(
                        host1,
                        port1,
                        "http"
                ),
                new HttpHost(
                        host2,
                        port2,
                        "http"
                ),
                new HttpHost(
                        host3,
                        port3,
                        "http"
                )
        ));
    }
}

```



#### 8.4 ES集群介绍 

- 集群和分布式
  - • 集群：多个人做一样的事。
  -  • 分布式：多个人做不一样的事。

- 集群解决的问题：
  - • 让系统高可用 
  - • 分担请求压力 分布式解决的问题： 
  - • 分担存储和计算的压力，提速 
  - • 解耦

- ElasticSearch 集群分布式架构相关概念
  - • 集群（cluster）：一组拥有共同的cluster name 的 节点。 
  - • 节点（node)  ：集群中的一个Elasticearch 实例 
  - • 索引（index) ：es存储数据的地方。相当于关系数据库中的database概念 
  - • 分片（shard）：索引可以被拆分为不同的部分进行存储，称为分片。在集群环境下，一个索引的不同分片可以拆分到不同的节点中，相当于mysql分库。 
  - • 主分片（Primary shard）：相对于副本分片的定义。 
  - • 副本分片（Replica shard）每个主分片可以有一个或者多个副本，数据和主分片一样。

- 集群原理-分片配置

  - • 在创建索引时，如果不指定分片配置，则默认主分片1，副本分片1。 

  - • 在创建索引时，可以通过settings设置分片

  - • 分片与自平衡：当节点挂掉后，挂掉的节点分片会自平衡到其他节点中 

  - • 在Elasticsearch 中，每个查询在每个分片的单个线程中执行，但是，可以并行处理多个分片。 

  - • 分片数量一旦确定好，不能修改。 

  - • 索引分片推荐配置方案： 1. 每个分片推荐大小10-30GB 2. 分片数量推荐 = 节点数量 * 1~3倍

  - 思考：比如有1000GB数据，应该有多少个分片？多少个节点 ?

    答:    40个分片 20个节点

  - ```shell
    ###kibana 脚本
    #创建索引时设置分片
    PUT test_index
    {
       "settings": {
          "number_of_shards": 3,
          "number_of_replicas": 1
       },
       "mappings": {
          "properties": {
              "name": {"type": "text"}
          }
       }
    }
    
    
    ```

    

- 集群原理-路由原理
  - • 文档存入对应的分片，ES计算分片编号的过程，称为路由。 
  - • Elasticsearch是怎么知道一个文档应该存放到哪个分片中呢？ 
  - • 查询时，根据文档id查询文档，Elasticsearch 又该去哪个分片中查询数据呢？ 
  - • 路由算法 ：shard_index = hash(id) % number_of_primary_shards

![img](https://img-blog.csdnimg.cn/20201114081103803.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)



#### 8.5 脑裂

- • 一个正常es集群中只有一个主节点（Master），主节点负责管理整个集群。如创建或删除索引，跟踪哪些节点是群 集的一部分，并决定哪些分片分配给相关的节点。 
- • 集群的所有节点都会选择同一个节点作为主节点。 
- • 脑裂问题的出现就是因为从节点在选择主节点上出现分歧导致一个集群出现多个主节点从而使集群分裂，使得集群 处于异常状态。



##### 脑裂原因

1. 网络原因：网络延迟 • 一般es集群会在内网部署，也可能在外网部署，比如阿里云。 • 内网一般不会出现此问题，外网的网络出现问题的可能性大些。 

2. 节点负载 • 主节点的角色既为master又为data。数据访问量较大时，可能会导致Master节点停止响应（假死状态）。

  ```properties
  ##elasticsearch.yml 配置
  
  #是否有资格选举为主节点
  node.master=true
  
  #是否存储数据
  node.data=true
  ```

3. JVM内存回收 • 当Master节点设置的JVM内存较小时，引发JVM的大规模内存回收，造成ES进程失去响应。



##### 避免脑裂

1. 网络原因：discovery.zen.ping.timeout超时时间配置大一点。默认是3S 

2. 节点负载：角色分离策略

   • 候选主节点配置为 • node.master: true • node.data: false 

   • 数据节点配置为 • node.master: false • node.data: true

3. JVM内存回收：修改 config/jvm.options 文件的 -Xms 和 -Xmx 为服务器的内存一半。



#### 8.6 集群扩容

- 修改节点名称itcast-1...4
- node.max_local_storage_nodes = 4
- discovery.seed_hosts: ["localhost:9700","localhost:9800","localhost:9900","localhost:9600"]
- cluster.initial_master_nodes: ["itcast-1", "itcast-2","itcast-3","itcast-4"]
- 9204, 9600
- 注意：其他三个节点ES，discovery.seed_hosts 和 cluster.initial_master_nodes 和 node.max_local_storage_nodes 也要做出调整。
- <font color=red>配置修改好后，四台ES重启即可。</font>

```properties
#集群名称
cluster.name: itcast-es
#节点名称
node.name: itcast-4 
#是不是有资格主节点
node.master: true
#是否存储数据
node.data: true
#最大集群节点数
node.max_local_storage_nodes: 4 
#ip地址
network.host: 0.0.0.0
#端口
http.port: 9204
#内部节点之间沟通端口
transport.tcp.port: 9600
#es7.x 之后新增的配置，节点发现
discovery.seed_hosts: ["localhost:9700","localhost:9800","localhost:9900","localhost:9600"]
#es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举master
cluster.initial_master_nodes: ["itcast-1", "itcast-2","itcast-3","itcast-4"] 
#数据和存储路径
path.data: /opt/data
path.logs: /opt/logs
```

