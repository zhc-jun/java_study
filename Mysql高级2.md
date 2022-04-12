# Mysql高级1

### 1.  存储引擎

##### 01. MySQL支持的存储引擎 

- MySQL5.7支持的引擎包括：InnoDB、MyISAM、MEMORY、Archive、Federate、CSV、BLACKHOLE等 

- 其中较为常用的有三种：InnoDB、MyISAM、MEMORY

- **<font color=red>MyISAM存储引擎 :</font>** 

  > 查询速度快，不支持事务和外键, 支持表锁和全文索引, 表结构保存在.frm文件中，表数据保存在.MYD文件中，索引保存在.MYI文件中

  **特点:** 

  - MyISAM 特点： MyISAM不支持事务、不支持外键。支持全文检索和表级锁定，读写相互阻塞，读取速度快，节约资源。 

  **使用场景:** 

  - 以查询操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不是很高！

-  **<font color=red>InnoDB存储引擎(MySQL5.5版本后默认) :</font>**

  > 查询性能相比MyISAM 低，支持事务，占用磁盘空间大，支持并发控制。表结构保存在.frm文件中，如果是共享表空间，数据和索引保存 在 innodb_data_home_dir 和 innodb_data_file_path定义的表空间中，可以是多个文件。如果是多表空间 存储，每个表的数据和索引单独保存在.ibd中。 

  **特点:** 

  - MySQL的默认存储引擎，InnoDB支持事务、支持外键、行级锁定。 支持所有辅助索引(5.5.5后不支持全文检索)，高缓存。 

  **使用场景:** 

  - 对事务的完整性有比较高的要求，在并发条件下要求数据的一致性，读写频繁的操作！

- **<font color=red>MEMORY存储引擎 ：</font>** 

  > 内存存储，所以操作速度特别快，不安全。适合小量快速访问的数据。表结构保存在.frm中。

  **特点:** 

  - 将所有数据保存在内存中，在需要快速定位记录和其他类似数据环境下，可以提供更快的访问。 对表的大小有限制，太大的表无法缓存在内存中， 其次是要确保表的数据可以恢复，数据库异常终止后表中的数据是可以恢复的。

  **使用场景:** 

  - 通常用于更新不太频繁的小表，用来快速得到访问的结果！



##### 02. InnoDB、MyISAM、MEMORY 对比

|     特性     |   MyISAM   |    InnoDB     | MEMORY（了解） |
| :----------: | :--------: | :-----------: | :------------: |
| **事物安全** | **不支持** |   **支持**    |   **不支持**   |
|  **锁机制**  |  **表锁**  | **表锁/行锁** |    **表锁**    |
|  B+Tree索引  |    支持    |     支持      |      支持      |
|   Hash索引   |   不支持   |    不支持     |      支持      |
|   **外键**   | **不支持** |   **支持**    |   **不支持**   |



**<font color=red>总结：针对不同的需求场景，来选择最适合的存储引擎即可！如果不确定、则使用数据库默认的存储引擎！</font>**



##### 03. 查看 创建 修改存储引擎

```mysql
/*
	查询数据库支持的存储引擎
	标准语法
		SHOW ENGINES;
*/
-- 查询数据库支持的存储引擎
SHOW ENGINES;

/*
	查询某个数据库中所有数据表的存储引擎
	标准语法
		SHOW TABLE STATUS FROM 数据库名称;
*/
-- 查询db9数据库所有表的存储引擎
SHOW TABLE STATUS FROM db9;

/*
	查询某个数据库中某个表的存储引擎
	标准语法
		SHOW TABLE STATUS FROM 数据库名称 WHERE NAME = '数据表名称';
*/
-- 查看db9数据库中account表的存储引擎
SHOW TABLE STATUS FROM db9 WHERE NAME = 'account';


/*
	创建数据表指定存储引擎
	标准语法
		CREATE TABLE 表名(
			列名,数据类型,
			...
		)ENGINE = 引擎名称;
*/
-- 创建db11数据库
CREATE DATABASE db11;

-- 使用db11数据库
USE db11;

-- 创建engine_test表，指定存储引擎为MyISAM
CREATE TABLE engine_test(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(10)
)ENGINE = MYISAM;

-- 查询engine_test表的存储引擎
SHOW TABLE STATUS FROM db11 WHERE NAME ='engine_test';


/*
	修改数据表的存储引擎
	标准语法
		ALTER TABLE 表名 ENGINE = 引擎名称;
*/
-- 修改engine_test表的引擎为InnoDB
ALTER TABLE engine_test ENGINE = INNODB;

-- 查询engine_test表的存储引擎
SHOW TABLE STATUS FROM db11 WHERE NAME ='engine_test';

```



##### 04. 场景练习

- 通过以上SQL练习, 我们可以知道MySQL的存储引擎是基于表的设置的, 可以给不同的表设置不同的存储引擎
- 我们可以基于业务, 对不同表设置不同的存储引擎, 进行优化

   **业务场景:**   电商项目中,  订单表   和  商品表

- 订单表: 电商系统中，订单表特点必须要保证订单数据安全, 必须支持事物, 订单表写多读少(买买买！大量insert),  所以适合采用Innodb存储引擎
- 商品表: 由商家后台将数据插入到商品表, 商品更新不存在频繁更新, 网名再下单时会先查看商品的信息, 所以商品的查询操作十分频繁，尤其双十一、618、秒杀活动等,  具备读多写少,  所以适合采用Myisam存储引擎。



### 2. 索引  --（重要）

##### 01. 索引的介绍、分类

- 索引实际上是一种数据结构, 能够提高数据检索效率。
- 没有索引, MySQL默认全表扫描, 逐个数据进行匹对, 效率低下。
- 索引会对数据进行排序(底层)
- <font color=red>MySQL    Innoddb和 myisam 默认索引使用的数据结构是是B + Tree，B + Tree是BTree （平衡树-数据结构）的变种 </font>

   **按照功能分类** 

- 普通索引- Normal ：最基本的索引，它对数据没有任何限制
- 唯一索引-UNIQUE：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值组合必须唯一。 
- 主键索引-PRIMARY：一种特殊的唯一索引，不允许有空值。一般在建表时同时创建主键索引。
- 组合索引：顾名思义，就是将单列索引进行组合。
- 外键索引：只有InnoDB引擎支持外键索引，用来保证数据的一致性、完整性和实现级联操作。
- 全文索引-FULLTEXT：快速匹配全部文档的方式。InnoDB引擎5.6版本后才支持全文索引。MEMORY引擎不支持。

   **按数据结构分类:** 

- B+Tree索引：MySQL使用最频繁的一个索引数据结构，是InnoDB和MyISAM存储引擎默认的索引类型。 
- Hash索引：MySQL中Memory存储引擎默认支持的索引类型。



##### 02._索引_数据准备

```mysql
-- 创建db12数据库
CREATE DATABASE db12;

-- 使用db12数据库
USE db12;

-- 创建student表
CREATE TABLE student(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(10),
	age INT,
	score INT
);
-- 添加数据
INSERT INTO student VALUES (NULL,'张三',23,98),(NULL,'李四',24,95),
(NULL,'王五',25,96),(NULL,'赵六',26,94),(NULL,'周七',27,99);
```

##### 03. _索引_创建和查询索引

```mysql
/*
	创建索引
	标准语法
		CREATE [UNIQUE|FULLTEXT] INDEX 索引名称
		[USING 索引类型]  -- 默认是B+TREE
		ON 表名(列名...);
*/
-- 为student表中的name列创建一个普通索引
CREATE INDEX idx_name ON student(NAME);

-- 为student表中的age列创建一个唯一索引
CREATE UNIQUE INDEX idx_age ON student(age);


/*
	查询索引
	标准语法
		SHOW INDEX FROM 表名;
*/
-- 查询student表中的索引
SHOW INDEX FROM student;
```



##### 04._索引_添加索引和删除索引

```mysql
/*
	ALTER添加索引
	-- 普通索引
	ALTER TABLE 表名 ADD INDEX 索引名称(列名);

	-- 组合索引
	ALTER TABLE 表名 ADD INDEX 索引名称(列名1,列名2,...);

	-- 主键索引
	ALTER TABLE 表名 ADD PRIMARY KEY(主键列名); 

	-- 外键索引(添加外键约束，就是外键索引)
	ALTER TABLE 表名 ADD CONSTRAINT 外键名 FOREIGN KEY (本表外键列名) REFERENCES 主表名(主键列名);

	-- 唯一索引
	ALTER TABLE 表名 ADD UNIQUE 索引名称(列名);

	-- 全文索引
	ALTER TABLE 表名 ADD FULLTEXT 索引名称(列名);
*/
-- 为student表中name列添加全文索引
ALTER TABLE student ADD FULLTEXT idx_fulltext_name(NAME);

-- 查询student表的索引
SHOW INDEX FROM student;



/*
	删除索引
	标准语法
		DROP INDEX 索引名称 ON 表名;
*/
-- 删除idx_name索引
DROP INDEX idx_name ON student;
```



##### 05. 索引的效率测试

```mysql
-- 创建product商品表
CREATE TABLE product(
	id INT PRIMARY KEY AUTO_INCREMENT,  -- 商品id
	NAME VARCHAR(10),		    -- 商品名称
	price INT                           -- 商品价格
);

-- 定义存储函数，生成长度为10的随机字符串并返回
DELIMITER $

CREATE FUNCTION rand_string() 
RETURNS VARCHAR(255)
BEGIN
	DECLARE big_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIGKLMNOPQRSTUVWXYZ';
	DECLARE small_str VARCHAR(255) DEFAULT '';
	DECLARE i INT DEFAULT 1;
	
	WHILE i <= 10 DO
		SET small_str =CONCAT(small_str,SUBSTRING(big_str,FLOOR(1+RAND()*52),1));
		SET i=i+1;
	END WHILE;
	
	RETURN small_str;
END$

DELIMITER ;



-- 定义存储过程，添加100万条数据到product表中
DELIMITER $

CREATE PROCEDURE pro_test()
BEGIN
	DECLARE num INT DEFAULT 1;
	
	WHILE num <= 1000000 DO
		INSERT INTO product VALUES (NULL,rand_string(),num);
		SET num = num + 1;
	END WHILE;
END$

DELIMITER ;

-- 调用存储过程
CALL pro_test();


-- 查询总记录条数
SELECT COUNT(*) FROM product;



-- 查询product表的索引
SHOW INDEX FROM product;

-- 查询name为OkIKDLVwtG的数据   (0.049)
SELECT * FROM product WHERE NAME='OkIKDLVwtG';

-- 通过id列查询OkIKDLVwtG的数据  (1毫秒)
SELECT * FROM product WHERE id=999998;

-- 为name列添加索引
ALTER TABLE product ADD INDEX idx_name(NAME);

-- 查询name为OkIKDLVwtG的数据   (0.001)
SELECT * FROM product WHERE NAME='OkIKDLVwtG';


/*
	范围查询
*/
-- 查询价格为800~1000之间的所有数据 (0.052)
SELECT * FROM product WHERE price BETWEEN 800 AND 1000;

/*
	排序查询
*/
-- 查询价格为800~1000之间的所有数据,降序排列  (0.083)
SELECT * FROM product WHERE price BETWEEN 800 AND 1000 ORDER BY price DESC;

-- 为price列添加索引
ALTER TABLE product ADD INDEX idx_price(price);

-- 查询价格为800~1000之间的所有数据 (0.011)
SELECT * FROM product WHERE price BETWEEN 800 AND 1000;

-- 查询价格为800~1000之间的所有数据,降序排列  (0.001)
SELECT * FROM product WHERE price BETWEEN 800 AND 1000 ORDER BY price DESC;
```



##### 06. (扩展) 索引的性能分析-Explain

- 使用explain（就是在sql语句前，加explain而已，特别简单~~，但报告结果就不简单了，根本看不懂，下面开始介绍~） 

```mysql
-- 查询价格为800~1000之间的所有数据,降序排列  (0.001)
EXPLAIN SELECT * FROM product WHERE price BETWEEN 800 AND 1000 ORDER BY price DESC;
```

**报告:** 

+----+-------------+---------+-------+---------------+-----------+---------+------+------+-------------+
| id | select_type | table   | type  | possible_keys | key       | key_len | ref  | rows | Extra       |
+----+-------------+---------+-------+---------------+-----------+---------+------+------+-------------+
|  1 | SIMPLE      | product | range | idx_price     | idx_price | 5       | NULL |  201 | Using where |
+----+-------------+---------+-------+---------------+-----------+---------+------+------+-------------+

**id列：** 影响表的读取顺序

> - id列相同：可以认为是一组，从上至下顺序执行
> - id列不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先执行。

**select_type列：** 查询类型，主要用于区别：普通查询，联合查询，子查询等复杂查询。

> - SIMPLE(简单SELECT,没有实用UNION或子查询等其他)
> - PRIMARY(查询中若包含任何复杂的子部分,最外层的select被标记为PRIMARY)
> - DERIVED( 位于FROM后的的子查询，也称为派生表的SELECT)
> - UNION(UNION中出现的第二个SELECT语句)
> - UNION RESULT(UNION的结果)，从 union 临时表检索结果的 select
> - SUBQUERY 它是子查询中的出现的第一个SELECT，它是不在from后的子查询

 **table列** : 表示对那个表进行查询

   **表示对哪个表进行的查询**，有时候可能不是真实存在的表名字，例如：当 from 子句中有子查询时，table列是 格式，表示当前查询依赖 id=N 的查询，于是先执行 id=N 的查询。当有 union 时，UNION RESULT 的 table 列的值为 <union1,2>，1和2表示参与 union 的 select 行id。

**type列:** 

**表示查询的方式或访问类型**，常用的访问类型有：system > const > eq_ref > ref > fulltext > index_merge > unique_subquery > index_subquery > range > index > all （依次表示查询类型的性能排序，从优到差）

> ALL： 对全表数据进行查询遍历
>
> index: 它通过表中索引进行查询，通常比ALL快很多
>
> range: 使用一个索引来选择行,范围扫描通常出现在 in(), between ,> ,<, >= 等操作中。
>
>  **eq_ref:** 简单来说，就是多表连接中使用primary key或者 unique key作为关联条件,多表之间的关联之间可见。 
>
>  **const、system:** mysql能对查询的某部分进行优化并将其转化成一个常量，用于 primary key 或 unique key 的所有列与常量比较时，所以表最多有一个匹配行，读取1次，速度比较快。  

**possible_keys列:**

**显示可能应用在这张表中的索引，一个或者多个**，如果查询的表中建立有索引可能会被显示到这里，但也不一定被查询使用。（不一定有索引就一定显示索引出来）explain 时可能出现 possible_keys 有列，而 key 显示 NULL 的情况，这种情况是因为表中数据不多，mysql认为索引对此查询帮助不大，选择了全表查询。

**key列:** 一般用来检测select 是否用到索引
**实际使用的索引，若没有使用索引，则键是NULL**。若想要强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。

**key_len列:**
表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度（key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的）

不损失精确性的情况下，长度越短越好

**ref列:**
这一列显示了在key列记录的索引中，表查找值所用到的列或常量，常见的有：const（常量），func，NULL，字段名（例：film.id）【表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值】

**rows列:**
表示是MySQL估计要读取并检测的行数，并不是结果集里的行数。

**Extra列:** 

> Using where: 列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候，表示mysql服务器将在存储引擎检索行后再进行过滤
>
> Using temporary： 表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询
>
> Using filesort： MySQL中无法利用索引完成的排序操作称为“文件排序”
>
> Using join buffer： 改值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进能。
>
> Impossible where： 这个值强调了where语句会导致没有符合条件的行。
>
> Select tables optimized away： 这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行



##### 07. 索引的原理

   **磁盘存储 :** 

-  系统从磁盘读取数据到内存时是以磁盘块（block）为基本单位的 
-  位于同一个磁盘块中的数据会被一次性读取出来，而不是需要什么取什么。
-  InnoDB存储引擎中有页（Page）的概念，页是其磁盘管理的最小单位。InnoDB存储引擎中默认每个页的大小为16KB。
- InnoDB引擎将若干个地址连接磁盘块，以此来达到页的大小16KB，在查询数据时如果一个页中的每条数据都能有助于定位数 据记录的位置，这将会减少磁盘I/O次数，提高查询效率。



   **BTree：** 

- MySQL根据表的索引列创建平衡树（BTree）时，先对索引列中的数据进行**排序**，取中间值作为树的根节点，根节点的子节点规则：**左边放小值，右边放大值**。 
- 每一个节点保存的数据有:  列值    子节点指针   指向数据行的指针
- 叶子节点没有子节点

- 在B树中, 还存在两种类型的索引: **聚集索引 ** 和 **非聚集索引**



**举例介绍 聚集索引  和 非聚集索引**

前言:    我们用书举例, 把书看成是一张数据表, 那么每一张纸上的文字就是我们表中的数据，当我们想要查看找数据时，有两种方式可以进行检索, 一种是书前几页目录，另外一种是每页纸右下角的页码

**聚集索引:** 

​     聚集索引就好比我们书的页码，它的特点是:  索引值与真实数据同在一个页（磁盘存储: Page）中；

**非聚集索引:** 

​     非聚集索引就是指我们的书的目录,  目录列表中每一项就是我们的索引值，每一个索引背后都指向了页码，也就是指向了我们的聚集索引的id值，好处就是我们不需要针对非聚集索引值额外的在存储一份行数据在该磁盘页上，可以大大节省了我们的磁盘开销；

**关于聚集和非聚集索引-请注意:** 

- Innodb每张表中只有一个聚集索引, 其他索引都为 非聚集索引

- 默认会拿主键id作为聚集索引

- 如果没有主键，会取非空的唯一索引作为聚集索引

- 如果上面都没有，innodb会自己创建一个唯一id来作为聚集索引

  

##### 08. 索引设计原则

   **最左匹配原则** 

- 建立联合索引时会遵循最左匹配原则，即最左优先，在检索数据时从联合索引的最左边开始匹配 

- 例如：为user表中的name、address、phone列添加联合索引 ALTER TABLE user ADD INDEX index_three(name,address,phone); 

- 此时，联合索引index_three实际建立了(name)、(name,address)、(name,address,phone)三个索引 

- 所以，下面的三个SQL语句都可以命中索引

- SELECT * FROM user WHERE address = '北京' AND phone = '12345' AND name = '张三'; 

- SELECT * FROM user WHERE name = '张三' AND address = '北京'; 

- SELECT * FROM user WHERE name = '张三';

- 这三条SQL语句在检索时分别会使用以下索引进行数据匹配 (name,address,phone)     (name,address)      (name) 

- 索引字段出现的顺序可以是任意的，MySQL优化器会帮我们自动的调整where条件中的顺序 

- 如下不会全都命中索引 （**了解-看完10.索引失效后再回来看**）:

  1) SELECT * FROM user WHERE address = '北京' ;  最左面name为出现
  
  2) SELECT * FROM user WHERE name= '张' AND  address like '%北京';  只用到name索引
  
  3) SELECT * FROM user WHERE address = '北京' OR phone = '12345' OR name = '张三';  未使用任何索引.
  
  4) SELECT * FROM user WHERE address = '北京' OR phone like  '%12345' OR name = '张三';  只用到name索引,  phone 因为like  % 开头，导致 phone 和 address 都失效。
  
  5) SELECT * FROM user WHERE  name = '张三'  AND  phone > '12345' AND address = '北京' ;  只用到name, phone 索引,  address 不能在范围查询之后，否则失效。
  
   

###### **适合建立索引的列:** 

- 主键 唯一 外键约束会自动创建索引
- 频繁作为where查询条件的字段应该建立索引
- 单列和组合索引的选择问题, who? （高并发下倾向创建组合索引）

- 排序order by  col,  分组 group by col 的通过索引去查询，将大大提高速度， ps： 排序和分组底层都是要进行排序，因为索引排好序了，所以查询很快.

###### **不适合建立索引列:** 

- 表数据太少，例如 每张表控制在600-800万条数据；
- 经常增删改的表

- where 条件中用不到的字段，不创建索引。
- 频繁更新的字段，不适合建立索引，因为每次insert update delete 更新的不单单是数据，还会更新索引，影响性能
- 如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。例如：一个表中有2000条记录，索引列有1980个不同的值，索引选择性= 1980/2000 = 0.99; 一个列选择性越接近于1， 这个索引的效率越高。
- 数据值发布比较均匀的不适合建立索引，例如男女，真假值等，他们的选择性不高，没有建立索引的必要。

##### 09. (扩展) 索引的优缺点

- 优点: 

  > a) 提高检索速度,   除了B树的查找特点以外，索引只对匹对上的磁盘页进行IO操作，而且每个磁盘页中的大小默认都是16KB，所以相比没有索引时对所有数据进行IO操作，整体效率要更高；

- 缺点:  

  > a) 索引会额外占用一部分磁盘空间,
  >
  > b) 创建索引和维护索引要耗费时间，这种时间随着数据量的增加而增加
  >
  > c) 以表中的数据进行增、删、改的时候，索引也要动态的维护，这就降低了整体的维护速度 



##### 10. (扩展) Mysql索引会失效的几种情况分析

参考博客: https://www.cnblogs.com/applelife/p/10511455.html

-  如果条件中有or，即使其中有条件带索引也不会使用(这也是为什么尽量少用or的原因)  注意：要想使用or，又想让索引生效，只能将or条件中的每个列都加上索引 
-  like查询是以%开头 
-  组合索引中，如果出现中间列为范围查询，则后面的列不走索引，可以考虑将范围查询列从组合索引中剔除
-  如果mysql估计使用全表扫描要比使用索引快,则不使用索引 
-  字符串不加单引号索引失效

其他情况

```
1) 没有查询条件，或者查询条件没有建立索引 
2) 在查询条件上没有使用索引列 
3) 查询的数量是大表的大部分，应该是30％以上。 
4) 索引本身失效
5) 查询条件使用函数在索引列上，或者对索引列进行运算，运算包括(+，-，*，/，! 等) 错误的例子：select * from test where id-1=9; 正确的例子：select * from test where id=10; 
6) 对小表查询 
7) 提示不使用索引
8) 统计数据不真实 
9) CBO计算走索引花费过大的情况。其实也包含了上面的情况，这里指的是表占有的block要比索引小。 
10)隐式转换导致索引失效.这一点应当引起重视.也是开发中经常会犯的错误. 由于表的字段tu_mdn定义为varchar2(20),但在查询时把该字段作为number类型以where条件传给Oracle,这样会导致索引失效. 错误的例子：select * from test where tu_mdn=13333333333; 正确的例子：select * from test where tu_mdn='13333333333'; 
12) 1,<> 2,单独的>,<,(有时会用到，有时不会) 
13,like "%_" 百分号在前. 
4,表没分析. 
15,单独引用复合索引里非第一位置的索引列. 
16,字符型字段为数字时在where条件里不添加引号. 
17,对索引列进行运算.需要建立函数索引. 
18,not in ,not exist. 
19,当变量采用的是times变量，而表的字段采用的是date变量时.或相反情况。 
20,B-tree索引 is null不会走,is not null会走,位图索引 is null,is not null 都会走 
21,联合索引 is not null 只要在建立的索引列（不分先后）都会走, in null时 必须要和建立索引第一列一起使用,当建立索引第一位置条件是is null 时,其他建立索引的列可以是is null（但必须在所有列 都满足is null的时候）,或者=一个值； 当建立索引的第一位置是=一个值时,其他索引列可以是任何情况（包括is null =一个值）,以上两种情况索引都会走。其他情况不会走。
```





### 3. 锁机制 --（重要）

**加锁可以保证事务的一致性，那么什么是数据的一致性？**

-  一致性:  是指数据按照我们预期的效果去发生变化, 比如 商品杯子的库存100个，库存的最小值应为0个，如果不考虑锁，库存可能会出现负数，这显然是不合理的，我们可以通过锁的机制来保证数据按照我们预期去变化，这就是所谓的数据一致性；

##### 01. 锁按照操作分类-共享&排他锁

- 共享锁【S锁】

  > 一，又称读锁，若事务T对数据对象A加 上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。
  >
  > 二，换一种方式解释， 你家有一个大门，大门的钥匙有好几把，你有一把，你女朋友有一把，你们都可能通过这把钥匙进入你们家，把想要的物品用手机拍照带走，就是不能动原样物件，拿走就意味着原有的数据进行了修改，这个就是所谓的共享锁。 

- 排他锁【X锁】

  > 一，又称写锁。若事务T对数据对象A加 上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。这保证了其他事务在T释放A上的锁之前不能再读取和修改A
  >
  > 二，换一种方式解释，你们家大门，只有一把钥匙，藏在门口的鞋里，你回家拿钥匙进屋，你女朋友此时回家，将等待你出来归还（释放）钥匙后，它才可以继续进屋。

#####  02. 按操作数据粒度分类 -表锁&行锁

- 表锁

  > 事物A操作表中数据时，会对整个表进行加锁，会限制其他事物操作表中数据。
  >
  > **特点:**   开销小，加锁快。不会出现死锁。锁定力度大，发生锁冲突概率高，并发度最低
  >
  > **加锁的方式：**自动加锁。
  >
  > 一,  Innodb在执行 update delete insert时不会加表锁，在查询时，如果我们显示的通过关键字进行加锁（读写锁），**需注意:**   行级锁都是基于索引的，如果一条SQL用不到索引, 行级锁是不会生效的，此时mysql会使用表级锁把整张表锁住
  >
  > 
  
- 行锁

  > 事物A操作表中某一行数据时，会对改行数据进行加锁，如果事物B想要操作表中其他行数据，不会受到事物A锁影响，事物B要是对事物A锁定行数据操作，就会受到事物A锁影响；
  >
  > **特点:** 开销大，加锁慢。会出现死锁。锁定粒度小，发生锁冲突概率低，并发度高
  >
  >  **加锁的方式：** 自动加锁。对于UPDATE、DELETE和INSERT语句，InnoDB会自动给涉及数据行加 **“排他锁”** 来锁定当前行，当然我们也可以使用FOR UPDATE 在SELECT 时给莫一行就上**排它锁**，来锁定当前行；对于普通SELECT语句，InnoDB不会加任何锁。
  >
  > **<font color=red>注意事项:</font>** InnoDB默认就是给数据加行级锁，如果一条SQL用不到索引是不会使用行级锁的，行级锁的加锁就是基于索引的，会使用表级锁把整张表锁住。

##### 03. 使用方式分类-乐观&悲观锁

**乐观锁**

- **乐观锁不是数据库自带的，需要我们自己去实现** 
-  乐观锁是指操作数据库时(更新操作)，想法很乐观，认为这次的操作不会导致冲突，在操作数据时，并不进行任何其他的特殊处理（也就是不加锁），而在进行更新后，再去判断是否有冲突了。 

**悲观锁**

-  **悲观锁是由数据库自己实现了的，要用的时候，我们直接调用数据库的相关<font color=red>关键字</font>就可以了。** 
- 与乐观锁相对应的就是悲观锁了。悲观锁就是在操作数据时，认为此操作会出现数据冲突，所以在进行每次操作时都要通过获取锁才能进行对相同数据的操作，这点跟java中的synchronized很相似，所以悲观锁需要耗费较多的时间。另外与乐观锁相对应的，悲观锁是由数据库自己实现了的，要用的时候，我们直接调用数据库的相关语句就可以了。 
- 说到这里，由悲观锁涉及到的另外两个我们聊过的锁，它们就是共享锁与排它锁。共享锁和排它锁是悲观锁的不同的实现，它俩都属于悲观锁的范畴。
- <font color=red>SQL关键字实现共享锁: </font>  SELECT 语句 LOCK IN SHARE MODE
- <font color=red>SQL关键字实现排他锁: </font>  SELECT 语句 FOR UPDATE
- INSERT UPDATE DELETE 数据库自动加**排它锁**，默认行级锁，没走索引，加表锁。

**InnoDB和MyISAM的最大不同点有两个：**

一，InnoDB支持事务(transaction)；默认采用行级锁, 行级锁都是基于索引的，如果一条SQL用不到索引是不会使用行级锁的，会使用表级锁把整张表锁住。

二，MyISAM不支持事物, 默认采用表锁。

##### 04. Innodb 共享锁 演示

###### _锁_InnoDB锁的数据准备

```mysql
-- 创建db13数据库
CREATE DATABASE db13;

-- 使用db13数据库
USE db13;

-- 创建student表
CREATE TABLE student(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(10),
	age INT,
	score INT
);
-- 添加数据
INSERT INTO student VALUES (NULL,'张三',23,99),(NULL,'李四',24,95),
(NULL,'王五',25,98),(NULL,'赵六',26,97);
```

###### _锁_InnoDB共享锁_窗口1

```mysql
-- 窗口1
/*
	共享锁：数据可以被多个事务查询，但是不能修改
	创建锁的格式
		SELECT语句 LOCK IN SHARE MODE;
*/

-- 手动开启事务
START TRANSACTION;

-- 查询id为1的数据记录。并且加入共享锁
SELECT * FROM student WHERE id=1 LOCK IN SHARE MODE;

-- 查询分数为99的数据记录。并且加入共享锁
SELECT * FROM student WHERE score=99 LOCK IN SHARE MODE;

-- 提交事务
COMMIT;
```

###### _锁_InnoDB共享锁_窗口2

```mysql
-- 开启事务
START TRANSACTION;

-- 查询id为1的数据记录。(普通查询没问题)
SELECT * FROM student WHERE id=1;

-- 查询id为1的数据记录。并且加入共享锁(共享锁和共享锁是兼容的)
SELECT * FROM student WHERE id=1 LOCK IN SHARE MODE;

-- 修改id为1的数据。修改姓名为张三三(修改失败，会出现锁的情况。只有窗口1提交事务后，才能修改成功)
UPDATE student SET NAME='张三三' WHERE id=1;

-- 修改id为2的数据。修改姓名为李四四(修改成功。InnoDB引擎默认是行锁)
UPDATE student SET NAME='李四四' WHERE id=2;

-- 修改id为3的数据。修改姓名为王五五(注意：InnoDB引擎如果不采用带索引的列，则会提升为表锁)
UPDATE student SET NAME='王五五' WHERE id=3;

-- 提交事务
COMMIT;
```

##### 05. Inodb 排它锁演示

- 加锁的数据，不能被其他事务<font color=red>**加锁**</font>查询或修改 ；
- <font color=red>注意: 在加锁事物提交之前, 其他事物普通查询是可以查到修改前的数据</font>

###### 锁_InnoDB排他锁_窗口1

```mysql
-- 窗口1
/*
	排他锁：加锁的数据，不能被其他事务加锁查询或修改
	排他锁创建格式
		SELECT语句 FOR UPDATE;
*/
-- 开启事务
START TRANSACTION;

-- 查询id为1的数据记录。并且加入排他锁
SELECT * FROM student WHERE id=1 FOR UPDATE;

-- 提交事务
COMMIT;
```

###### _锁_InnoDB排他锁_窗口2

```mysql
-- 开启事务
START TRANSACTION;

-- 查询id为1的数据记录。(普通查询没问题)
SELECT * FROM student WHERE id=1;

-- 查询id为1的数据记录。并且加入共享锁(排他锁不能和共享锁共存)
SELECT * FROM student WHERE id=1 LOCK IN SHARE MODE;

-- 查询id为1的数据记录。并且加入排他锁(排他锁和排他锁不能共存)
SELECT * FROM student WHERE id=1 FOR UPDATE;

-- 修改id为1的数据。将姓名修改为张三三(不能修改。会出现锁的情况。只有窗口1提交事务后，才能修改成功)
UPDATE student SET NAME='张三三' WHERE id=1;

-- 提交事务
COMMIT;
```

###### 排它锁演示-是否会阻塞普通查询SQL

- 加锁事务提交前，其他事物可以通过普通SELECT读取修改前数据

**窗口1**

```mysql
-- 窗口1
/*
	排它锁：
	 1. 开启事物
	 2. 通过update语句, 让mysql自动对id=1行数据加锁；但不提交，让其锁住
	 3. 操作窗口2,去测试普通查询是否会被阻塞；
*/

-- 开启事务
START TRANSACTION;

-- 修改id为1的数据。修改姓名为张三三(修改失败，会出现锁的情况。只有窗口1提交事务后，才能修改成功)
UPDATE student SET NAME='张三' WHERE id=1;

-- 提交事务
COMMIT;
```

**窗口2**

```mysql
-- 窗口2
/*
	排它锁：开启事物后, 对 id=1数据进行普通查询
*/

-- 手动开启事务
START TRANSACTION;

-- 查询id为1的数据记录。(普通查询没问题)
SELECT * FROM student WHERE id=1 ;
-- 提交事务
COMMIT;
```



##### 06. MYISAM 读锁

- 所有连接只能读取数据，不能修改 

###### 锁_MYISAM锁的数据准备

```mysql
-- 创建product表
CREATE TABLE product(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20),
	price INT
)ENGINE = MYISAM;  -- 指定存储引擎为MyISAM

-- 添加数据
INSERT INTO product VALUES (NULL,'华为手机',4999),(NULL,'小米手机',2999),
(NULL,'苹果',8999),(NULL,'中兴',1999);
```

###### _锁_MYISAM读锁_窗口1

```mysql
/*
	读锁：所有连接只能读取数据，不能修改
	加锁
		LOCK TABLE 表名 READ;

	解锁(将当前会话所有的表进行解锁)
		UNLOCK TABLES;
*/
-- 为product表添加读锁
LOCK TABLE product READ;

-- 查询product表(查询成功)
SELECT * FROM product;

-- 将id为1的价格修改为5999(不能修改)
UPDATE product SET price=5999 WHERE id=1;

-- 解锁
UNLOCK TABLES;
```

###### _锁_MYISAM读锁_窗口2

```mysql
-- 查询product表(查询成功)
SELECT * FROM product;

-- 将id为1的价格修改为5999(不能修改。只有窗口1解锁后才能修改成功)
UPDATE product SET price=5999 WHERE id=1;
```

##### 07. _MYISAM写锁_

- 其他连接不能查询和修改数据 

###### 锁_MYISAM写锁_窗口1

```mysql
/*
	写锁：其他连接不能查询和修改数据
	加锁
		LOCK TABLE 表名 WRITE;

	解锁(将当前会话所有的表进行解锁)
		UNLOCK TABLES;
*/
-- 为product表添加写锁
LOCK TABLE product WRITE;

-- 查询product表(查询成功)
SELECT * FROM product;

-- 修改id为2的价格为3999(修改成功)
UPDATE product SET price=3999 WHERE id=2;

-- 解锁
UNLOCK TABLES;
```

###### _锁_MYISAM写锁_窗口2

```mysql
-- 查询product表（查询失败。只有窗口1解锁后，才能查询成功）
SELECT * FROM product;

-- 修改id为2的价格为1999(修改失败。只有窗口1解锁后，才能修改成功)
UPDATE product SET price=1999 WHERE id=2;
```

##### _08. 锁_乐观锁

```mysql
-- 创建city表
CREATE TABLE city(
	id INT PRIMARY KEY AUTO_INCREMENT,  -- 城市id
	NAME VARCHAR(20),                   -- 城市名称
	VERSION INT                         -- 版本号
);

-- 添加数据
INSERT INTO city VALUES (NULL,'北京',1),(NULL,'上海',1),(NULL,'广州',1),(NULL,'深圳',1);


-- 将北京修改为北京市
-- 1.将北京的版本号读取出来
SELECT VERSION FROM city WHERE NAME='北京';   -- 1
-- 2.修改北京为北京市，版本号+1.并对比版本号是否相同
UPDATE city SET NAME='北京市',VERSION=VERSION+1 WHERE NAME='北京' AND VERSION=1;
```

##### 09. 案例扩展

###### **乐观锁-购买商品:**

```mysql
-- 开始事务
start transaction;

-- 查询出商品信息, 普通的查询SQL不会加任何锁
select goodname, kuncun from t_goods where id=1

-- 修改商品库存
int 响应行数 = update t_goods set kuncun=kuncun-1 where id=1 and kuncun > 0;

-- 根据商品信息生成订单
if(响应行数 == 1){
  insert into t_orders (goodname, goods_id) values (goodname,1);
}else{
   -- 提示用户商品已卖完(库存为0)
}

-- 提交事务
commit;

-- 以上查询语句中，我们使用普通SQL查询商品信息，这对该商品数据行是没有加任何锁的, 当事物A想要修改库存时, 其他事物可能优先A提交, 导致库存为0,所以事物A在执行update 语句时， 因为kuncun 不满足>0, 所以响应行数不为1, 此时因为没有商品可卖了, 我们也不应该去创建订单, 因为 库存都没有了，还创建订单，这叫超卖，是不合理的，我们通过乐观锁（update where 使用字段做过滤条件）加 逻辑判断来保证数据按照我们预期的样子放生变化，进而保证了数据的一致性；
```

###### **悲观锁-购买商品:** 

- 悲观锁多用于高并发下，保证数据一致性的业务，比如：金融-支付系统用户的账户表等等。

```mysql

-- 开始事务, 并手动提交
start transaction;

-- 查找商品信息，商品id=1, 使用排它锁（写锁）来锁定该行数据, 不能使用共享锁，因为共享锁不允许锁定同时去修改改行数据, 只能读
select goodname, kuncun from t_goods where id=1 for update;
 
-- 判断存库是否大于0
if(kuncun > 0){
    -- 根据商品信息生成订单
    insert into t_orders (goodname, goods_id) values (goodname,1);
    -- 修改商品库存
    update t_goods set kucun=kuncun-1 where id=1;
}

-- 提交事务
commit;

-- 以上查询语句中，使用了select...for update方式，通过开启排他锁的方式实现了悲观锁。则相应的记录被锁定，其他事务必须等本次事务提交之后才能够执行, 所以我们利用悲观锁来保证商品表中“库存”正确做出修改的同时，也添加了一笔对应的订单, 这就是我们想要的数据正确变更, 也就是所谓的保证了数据一致性
-- 我们使用select ... for update会把数据给锁定，不过我们需要注意一些锁的级别，MySQL InnoDB默认行级锁。行级锁都是基于索引的，如果一条SQL用不到索引是不会使用行级锁的，会使用表级锁把整张表锁住。
```



### 4. Mysql--集群

### 1. 集群的概念

- 由多台计算机，每台部署一个MySQL数据库软件共同对外提供数据库的服务，这多台计算机整体，我们称之为集群
- 集群的优点是，可以存储更大的数据量，互联网系统的特点就是流量和数据都特别大，在一些大型的互联网公司，数据库集群是十分有必要的，一般当数据要超过GB到达TB甚至到PB时，就要考虑搭建数据集群
- 举例：假设一台mysql服务器可以承载的流量峰值为1000个并发，能够存储的数据量为1TB，如果我们搭建一个MySQL数据集群，集群中拥有3台计算机分别安装一个MySQL，那么集群理论上能够承载的流量为 1000 * 3 = 3000个并发，能够存储的数据总量为1TB * 3 = 3TB的数据；

### 2. MyCat介绍

- MyCat是一款管理数据库集群软件，是阿里曾经开源的知名产品——Cobar。 
-  简单的说，MyCat就是一个新颖的数据库中间件产品支持MySQL集群，提供高可用性数据分片集群。 
-  我们可以像使用mysql一样使用mycat。对于开发人员来说根本感觉不到mycat的存在。
-  MyCat不单单支持MySQL，像常用的关系型数据库Oracle、SqlServer都支持。

##### 01. Mycat 在系统架构中的角色

**Web系统(java 语言编写)   ---- > 访问Mycat Server  ---- > MySql 集群**

- 在没有MySQL集群之前，我们是java项目直接访问MySQL数据库, 这也是最简单的软件实现方式

- 从上面的关系，不难看出，当系统引入Mycat之后，我们对数据库的操作（正删改查）将由mycat通过网络将请求发给MySQL数据库

- 这么做的好处就是，Mycat会分析出MySQL集群中每台数据库的压力，通过算法，将本次请求发给压力较小的那台MySQL数据库，进而实现我们所说的 “负载均衡”；

- 如果，MySQL集群中某台MySQL 服务挂掉，Mycat 也可以感知到，并将我们的请求交给没有挂掉的其他MySQL，这就是我们所说的 “高可用”，一般软件系统能够达到四个9（99.99%），就可以称之为高可用,  解释： 全年在99.99%时间内，MySQL服务都是可用的，就可以称为高可用, 四个9也是高可用的最低标准， 365 天 * 0.0001 * 24小时 * 60分 = 53分钟; 从公式可以看出，如果全年停机时间大于53分钟，该系统服务达不到高可用标准；

  |           **描述**           | **通俗叫法** | **可用性级别** | **年度停机时间** |
  | :--------------------------: | :----------: | :------------: | :--------------: |
  |          基本可用性          |     2个9     |      99%       |     87.6小时     |
  |          较高可用性          |     3个9     |     99.9%      |     8.8小时      |
  | 具有故障自动恢复能力的可用性 |     4个9     |     99.99%     |      53分钟      |
  |          极高可用性          |     5个9     |    99.999%     |      5分钟       |

  

##### 02. Mycat 中间件的安装

```shell
# 1. 解压缩 itheima2.zip
# 文件位置:  07-Mysql（双元）\day04_mysql高级\资料\基础环境虚拟机\itheima2.zip

# 2. 用vmware打开解压缩后文件夹中的 CentOS7.vmx

# 3. 启动虚拟机后，关机再重启输入: root  itheima 进行登录

# 4. ifconfig 查看ip地址, 我的是 192.168.23.131

# 5. 使用SCRT 通过ip 进行远程登录

# 6. yum -y install lrzsz 
#运行 rz 命令； 在弹出的窗口选择需要上传的文件，文件会被上传至对应的目录下
#运行 sz file.name 在弹出的窗口选择保存文件的位置，文件会被下载至对应的目录下
# 7. 将\07-Mysql（双元）\day04_mysql高级\资料\Mycat-server-1.6.7.1-release-20190627191042-linux.tar.gz 上传到 ~目录下
[root@192 ~]$ rz    

# 8. 解压缩Mycat
[root@192 ~]$ tar -zxvf Mycat-server-1.6.7.1-release-20190627191042-linux.tar.gz

# 9. 授予最大读写执行权限
[root@192 ~]$ chmod 777 mycat -R

# 10. 编辑profile文件
[root@192 bin]$ vi /etc/profile

  #添加到文件的最下方
  export MYCAT_HOME=/root/mycat
  
# 11. 让文件重新加载一下
[root@192 ~]$ source /etc/profile

# 12. 启动mycat并检验, 如果可以通过netstat 检索出一条记录，则启动成功
[root@192 ~]$ cd mycat/bin/
[root@192 bin]$ ./mycat start
Starting Mycat-server...
[root@192 bin]$ netstat -ant|grep 8066
tcp6       0      0 :::8066                 :::*                    LISTEN     

# 14. 关闭防火墙
[root@192 bin]$ systemctl stop  firewalld

# 15. 执行开机禁用防火墙自启命令
[root@192 bin]$ systemctl disable firewalld.service

# 16. sqlyog 连接mycat 
ip: 192.168.23.131
端口: 8066
账号: root 
密码: 123456


```

##### 03. 克隆虚拟机-构建MySQL从服务器

- 关机当前虚拟机

- 右键vmware当前虚拟机----重命名为:  MySql-Master  //代表集群主节点

- 右键MySql-Master,  管理---克隆--完全克隆；给克隆虚拟机命名为: MySQL-Slave 代码从节点

- 左键选中 MySQL-Slave，点击上方 “虚拟机” -- 设置 -- 网络适配器2 -- 右下角高级 -- 最下方生成点击一下 -- 确定

- 启动 MySQL-Slave虚拟机，root itheima登录后，查看ip, 我的是 192.168.23.132,   SCRT 登录

- [root@192 ~]# vi /var/lib/mysql/auto.cnf 

- 随便修改一个数字， server-uuid=0b42cb61-d048-11e9-9440-000c297d15e6

- 关闭两台服务器的防火墙： 

  > systemctl stop  firewalld
  >
  > systemctl disable firewalld.service

- 启动两台服务器的mycat

  > [root@192 ~]# cd mycat/bin/
  > [root@192 bin]# ./mycat  start

- 查看两台服务器监听端口

  > [root@192 bin]# netstat -ant|grep 8066
  > tcp6       0      0 :::8066                 :::*                    LISTEN     
  > [root@192 bin]# netstat -ant|grep 3306
  > tcp6       0      0 :::3306                 :::*                    LISTEN

- sqlyog登录 MySql-Master 服务器的mycat 和 mysql

  > root  itheima  3306  192.168.23.131

- sqlyog登录 MySql-Slave 服务器的mysql

  > root  itheima  3306  192.168.23.132
  >
  > 

##### 04. 主从复制

###### 01主从复制配置

```shell
#1. 在MySql-Master服务器上，编辑mysql配置文件
# 编辑mysql配置文件
vi /etc/my.cnf

#在[mysqld]下面加上：
log-bin=mysql-bin # 开启复制操作
server-id=1  # master is 1, 其他从节点2 3 4 5 6...
innodb_flush_log_at_trx_commit=1  #事物提交方式
sync_binlog=1  #同步binlog日志

#2. 登录mysql，创建用户并授权
  # 登录mysql
  mysql -u root -p
  
  # 去除密码权限
  SET GLOBAL validate_password_policy=0;
  SET GLOBAL validate_password_length=1;
  
  # 创建用户
  CREATE USER 'hm'@'%' IDENTIFIED BY 'itheima';
  
  # 授权
  GRANT ALL ON *.* TO 'hm'@'%';

#3. 重启mysql服务，登录mysql服务
  # 重启mysql
  service mysqld restart
  
  # 登录mysql
  mysql -u root -p

#4. 查看主服务器的配置
  # 查看主服务器配置
  show master status;

  
#- MySql-Slave 从服务器的配置
#1. 在第二个服务器上，编辑mysql配置文件
  # 编辑mysql配置文件
  vi /etc/my.cnf
  
  # 在[mysqld]下面加上：
  server-id=2

#2. 登录mysql
  # 登录mysql
  mysql -u root -p
  
  # 执行
  mysql> use mysql;
  mysql> drop table slave_master_info;
  mysql> drop table slave_relay_log_info;
  mysql> drop table slave_worker_info;
  mysql> drop table innodb_index_stats;
  mysql> drop table innodb_table_stats;
  mysql> source /usr/share/mysql/mysql_system_tables.sql;

#3. 重启mysql，重新登录，配置从节点
  # 重启mysql
  service mysqld restart
  
  # 重新登录mysql
  mysql -u root -p
  
  # 执行
  change master to master_host='192.168.23.131',master_port=3306,master_user='hm',master_password='itheima',master_log_file='mysql-bin.000005',master_log_pos=154;

#4. 重启mysql，重新登录，开启从节点
  # 重启mysql
  service mysqld restart
  
  # 重新登录mysql
  mysql -u root -p
  
  # 开启从节点,
  # 注意: 如果出错：ERROR 1872 (HY000): Unknown error 1872 则 mysql> reset slave; 后 在 start slave;
  start slave;
  
  # 查询结果--不需要加;号
  show slave status\G
  #Slave_IO_Running和Slave_SQL_Running都为yes才表示同步成功。
  
  #注意: MYSQL同步故障：" SLAVE_SQL_RUNNING:NO" 解决办法
  1)进入master

    复制代码
    mysql> show master status;
    ±---------------------±---------±-------------±-----------------+
    | File | Position | Binlog_Do_DB | Binlog_Ignore_DB |
    ±---------------------±---------±-------------±-----------------+
    | localhost-bin.000005 | 154 | | |
    ±---------------------±---------±-------------±-----------------+
    1 row in set (0.00 sec)
    
  2)然后到slave服务器上执行手动同步：
  
    mysql> change master to master_host='192.168.23.131',master_port=3306,master_user='hm',master_password='itheima',master_log_file='mysql-bin.000005',master_log_pos=154;
    1 row in set (0.00 sec)
    
    mysql> start slave ;
    1 row in set (0.00 sec)

    mysql> show slave status\G
  

#5. 测试---在主服务器上创建一个db1数据库，查看从服务器上是否自动同步
```



###### 02主从复制_主服务器操作

```mysql
-- 主服务器创建db1数据库,从服务器会自动同步
CREATE DATABASE db1;

-- 从服务器创建db2数据库,主服务器不会自动同步
CREATE DATABASE db2;

```

##### 05. 读写分离

- Mysql-Master 服务器 -- 修改mycat配置

  >  1) 修改server.xml：vi /root/mycat/conf/server.xml
  >
  > ```xml
  > <user name="root">
  > 		<property name="password">123456</property>
  > 		<property name="schemas">HEIMADB</property>
  > 		
  > 		<!-- 表级 DML 权限设置 -->
  > 		<!-- 		
  > 		<privileges check="false">
  > 			<schema name="TESTDB" dml="0110" >
  > 				<table name="tb01" dml="0000"></table>
  > 				<table name="tb02" dml="1111"></table>
  > 			</schema>
  > 		</privileges>		
  > 		 -->
  > 	</user>
  > ```
  >
  >  2) 修改schema.xml： vi /root/mycat/conf/schema.xml     注意： 从服务器ip 改成你自己的，我的是 192.168.23.132
  >
  > ```xml
  > <?xml version="1.0"?>
  > <!DOCTYPE mycat:schema SYSTEM "schema.dtd">
  > <mycat:schema xmlns:mycat="http://io.mycat/">
  > 
  > 	<schema name="HEIMADB" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">
  > 	</schema>
  > 	<dataNode name="dn1" dataHost="localhost1" database="db1" />
  > 	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
  > 			  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
  > 		<heartbeat>select user()</heartbeat>
  > 		<!-- 主服务器负责写的操作 -->
  > 		<writeHost host="hostM1" url="localhost:3306" user="root"
  > 				   password="itheima">
  > 			<!-- 从服务器负责读的操作 -->
  > 			<readHost host="hostS2" url="192.168.23.132:3306" user="root" password="itheima" />
  > 		</writeHost>
  > 	</dataHost>
  > </mycat:schema>
  > ```
  >
  > 

- 上传 schema.xml  server.xml 到 ~目录下

- 移动 schema.xml  server.xml 到 mycat/conf 下

  > [root@192 ~] mv schema.xml  server.xml mycat/conf

- Mysql-Master 服务器 -- 重启mycat

  > [root@192 ~]# cd mycat/bin/
  > [root@192 bin]# ./mycat  restart
  >
  > [root@192 bin]# netstat -ant|grep 8066
  > tcp6       0      0 :::8066                 :::*                    LISTEN  



###### 01_读写分离_mycat操作

```mysql
-- 创建学生表
CREATE TABLE student(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(10)
);
-- 查询学生表
SELECT * FROM student;

-- 添加两条记录
INSERT INTO student VALUES (NULL,'张三'),(NULL,'李四');

-- 停止主从复制后，添加的数据只会保存到主服务器上。
INSERT INTO student VALUES (NULL,'王五');
```

###### 02_读写分离_主服务器操作

```mysql
-- 主服务器：查询学生表，可以看到数据
SELECT * FROM student;

```

###### 03_读写分离_从服务器操作

```mysql
-- 从服务器：查询学生表，可以看到数据(因为有主从复制)
SELECT * FROM student;

-- 从服务器：删除一条记录。(主服务器并没有删除，mycat中间件查询的结果是从服务器的数据)
DELETE FROM student WHERE id=2;
```



##### 

##### **06. 分库分表**

1): 将庞大的数据量拆分为不同的数据库和数据表进行存储！

2): 水平拆分 

>  根据表的数据逻辑关系，将同一表中的数据按照某种条件，拆分到多台数据库服务器上，也叫做横向拆分。 例如：一张1000万的大表，按照一模一样的结构，拆分成4个250万的小表，分别保存到4个数据库中。 

3):  垂直拆分 

> 根据业务的维度，将不同的表切分到不同的数据库之上，也叫做纵向拆分。 例如：所有的订单信息都保存到订单库中，所有的用户信息都保存到用户库中，同类型的表保存在同一个库中。



##### 07. 水平分表

- Mysql-Master 服务器 -- 修改mycat配置

- 07-Mysql（双元）\day04_mysql高级\资料\02mycat水平拆分配置文件

  >1) 修改主服务器中的server.xml：vi /root/mycat/conf/server.xml
  >
  >```xml
  ><property name="sequnceHandlerType">0</property>
  >```
  >
  >2) [root@192 bin]# vi /root/mycat/conf/sequence_conf.properties 
  >
  >```properties
  >#默认就是如下无需修改
  >GLOBAL.HISIDS=     #可以自定义关键字
  >GLOBAL.MINID=10001  #主键自增方式添加数据, 最小值
  >GLOBAL.MAXID=20000  #主键自增方式添加数据, 最大值
  >GLOBAL.CURID=10000  #当前主键值
  >```
  >
  >3）修改主服务器中的schema.xml：vi /root/mycat/conf/schema.xml 注意配置中的从服务器ip地址，我的是192.168.23.132
  >
  >```xml
  ><?xml version="1.0"?>
  ><!DOCTYPE mycat:schema SYSTEM "schema.dtd">
  ><mycat:schema xmlns:mycat="http://io.mycat/">
  >
  >	<schema name="HEIMADB" checkSQLschema="false" sqlMaxLimit="100">
  >		<table name="product" primaryKey="id" dataNode="dn1,dn2,dn3" rule="mod-long"/>
  >	</schema>
  >	
  >	<dataNode name="dn1" dataHost="localhost1" database="db1" />
  >	<dataNode name="dn2" dataHost="localhost1" database="db2" />
  >	<dataNode name="dn3" dataHost="localhost1" database="db3" />
  >	
  >	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
  >			  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
  >		<heartbeat>select user()</heartbeat>
  >		<!-- 主服务器负责写的操作 -->
  >		<writeHost host="hostM1" url="localhost:3306" user="root"
  >				   password="itheima">
  >			<!-- 从服务器负责读的操作 -->
  >			<readHost host="hostS2" url="192.168.23.132:3306" user="root" password="itheima" />
  >		</writeHost>
  >	</dataHost>
  ></mycat:schema>
  >
  >```
  >
  >

- 上传 schema.xml  server.xml rule.xml到 ~目录下

- 移动 schema.xml  server.xml  rule.xml到 mycat/conf 下

  > [root@192 ~] mv schema.xml  server.xml  rule.xml    mycat/conf

- Mysql-Master 服务器 -- 重启mycat

  > [root@192 ~]# cd mycat/bin/
  > [root@192 bin]# ./mycat  restart
  >
  > [root@192 bin]# netstat -ant|grep 8066
  > tcp6       0      0 :::8066                 :::*                    LISTEN  



###### 01_水平拆分_mycat操作

```mysql

-- 提前创建 db2 db3两个数据库
CREATE DATABASE db2;
CREATE DATABASE db3;

-- 创建product表
CREATE TABLE product(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20),
	price INT
);

-- 添加6条数据  mycat 生成主键固定写法: NEXT VALUE FOR MYCATSEQ_GLOBAL
INSERT INTO product(id,NAME,price) VALUES (NEXT VALUE FOR MYCATSEQ_GLOBAL,'苹果手机',6999);
INSERT INTO product(id,NAME,price) VALUES (NEXT VALUE FOR MYCATSEQ_GLOBAL,'华为手机',5999); 
INSERT INTO product(id,NAME,price) VALUES (NEXT VALUE FOR MYCATSEQ_GLOBAL,'三星手机',4999); 
INSERT INTO product(id,NAME,price) VALUES (NEXT VALUE FOR MYCATSEQ_GLOBAL,'小米手机',3999); 
INSERT INTO product(id,NAME,price) VALUES (NEXT VALUE FOR MYCATSEQ_GLOBAL,'中兴手机',2999); 
INSERT INTO product(id,NAME,price) VALUES (NEXT VALUE FOR MYCATSEQ_GLOBAL,'OOPO手机',1999); 

-- 查询product表
SELECT * FROM product; 
```

###### 02_水平拆分_主服务器操作

```mysql
-- 在不同数据库中查询product表
SELECT * FROM product;
```

###### 03_水平拆分_从服务器操作

```mysql
-- 在不同数据库中查询product表
SELECT * FROM product;
```

总结：

优点：

1. 通过水平查分可以将之前一张表中所有数据进行稀释，比如一张表中有1000W条数据，现在查分4个表出来，每张表都装有250W条数据，那么在检索数据时，可以提高效果；

**取模差分法**

 优点：

  1）可以防止热点数据（数据的倾斜）, 避免了某一表数据过多，其他表数据过少，取模可以实现数据的均等分配；

 缺点：

  1）当增加一个新的数据库db4时，之前db1 db2 db3中所有的老数据，都要通过自己写程序，将这些老数据对4取模，从新散落下去





##### 08. 垂直拆分

- Mysql-Master 服务器 -- 修改mycat配置

- 07-Mysql（双元）\day04_mysql高级\资料\03mycat垂直拆分配置文件

  > 修改主服务器的schema.xml：vi /root/mycat/conf/schema.xml  , 注意配置中的从服务器ip地址，我的是192.168.23.132
  >
  > ```xml
  > <?xml version="1.0"?>
  > <!DOCTYPE mycat:schema SYSTEM "schema.dtd">
  > <mycat:schema xmlns:mycat="http://io.mycat/">
  > 
  > 	<schema name="HEIMADB" checkSQLschema="false" sqlMaxLimit="100">
  > 		<table name="product" primaryKey="id" dataNode="dn1,dn2,dn3" rule="mod-long"/>
  > 		
  > 		<!-- 动物类数据表 -->
  > 		<table name="dog" primaryKey="id" autoIncrement="true" dataNode="dn4" />
  > 		<table name="cat" primaryKey="id" autoIncrement="true" dataNode="dn4" />
  >     
  >        <!-- 水果类数据表 -->
  > 		<table name="apple" primaryKey="id" autoIncrement="true" dataNode="dn5" />
  > 		<table name="banana" primaryKey="id" autoIncrement="true" dataNode="dn5" />
  > 	</schema>
  > 	
  > 	<dataNode name="dn1" dataHost="localhost1" database="db1" />
  > 	<dataNode name="dn2" dataHost="localhost1" database="db2" />
  > 	<dataNode name="dn3" dataHost="localhost1" database="db3" />
  > 	
  > 	<dataNode name="dn4" dataHost="localhost1" database="db4" />
  > 	<dataNode name="dn5" dataHost="localhost1" database="db5" />
  > 	
  > 	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
  > 			  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
  > 		<heartbeat>select user()</heartbeat>
  > 		<!-- 主服务器负责写的操作 -->
  > 		<writeHost host="hostM1" url="localhost:3306" user="root"
  > 				   password="itheima">
  > 			<!-- 从服务器负责读的操作 -->
  > 			<readHost host="hostS2" url="192.168.23.132:3306" user="root" password="itheima" />
  > 		</writeHost>
  > 	</dataHost>
  > </mycat:schema>
  > 
  > ```
  >
  > 

- 上传 schema.xml  到 ~目录下

- 移动 schema.xml  到 mycat/conf 下

  > [root@192 ~] mv schema.xml  mycat/conf

- Mysql-Master 服务器 -- 重启mycat

  > [root@192 ~]# cd mycat/bin/
  > [root@192 bin]# ./mycat  restart
  >
  > [root@192 bin]# netstat -ant|grep 8066
  > tcp6       0      0 :::8066                 :::*                    LISTEN  



###### 01_垂直拆分_mycat操作

```mysql
-- 提前创建 db4 db5两个数据库
CREATE DATABASE db4;
CREATE DATABASE db5;

-- 创建dog表
CREATE TABLE dog(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(10)
);
-- 添加数据 mycat 生成主键固定写法: NEXT VALUE FOR MYCATSEQ_GLOBAL
INSERT INTO dog(id,NAME) VALUES (NEXT VALUE FOR MYCATSEQ_GLOBAL,'哈士奇');
-- 查询dog表
SELECT * FROM dog;


-- 创建cat表
CREATE TABLE cat(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(10)
);
-- 添加数据
INSERT INTO cat(id,NAME) VALUES (NEXT VALUE FOR MYCATSEQ_GLOBAL,'波斯猫');
-- 查询cat表
SELECT * FROM cat;


-- 创建apple表
CREATE TABLE apple(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(10)
);
-- 添加数据
INSERT INTO apple(id,NAME) VALUES (NEXT VALUE FOR MYCATSEQ_GLOBAL,'红富士');
-- 查询apple表
SELECT * FROM apple;


-- 创建banana表
CREATE TABLE banana(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(10)
);
-- 添加数据
INSERT INTO banana(id,NAME) VALUES (NEXT VALUE FOR MYCATSEQ_GLOBAL,'香蕉');
-- 查询banana表
SELECT * FROM banana;
```

###### 02_垂直拆分_主服务器操作

```mysql
-- 查询dog表
SELECT * FROM dog;
-- 查询cat表
SELECT * FROM cat;


-- 查询apple表
SELECT * FROM apple;
-- 查询banana表
SELECT * FROM banana;
```

###### 03_垂直拆分_从服务器操作

```mysql
-- 查询dog表
SELECT * FROM dog;
-- 查询cat表
SELECT * FROM cat;


-- 查询apple表
SELECT * FROM apple;
-- 查询banana表
SELECT * FROM banana;
```

##### 09. (扩展)   总结-分库分表

1):  主从复制是mysql自己的功能

2):  mycat 可以实现 读写分离,  负载均衡,  高可用,  水平分库分表， 垂直分库分表



**一般: 在优化 数据库架构的时候，有这样的几点思考**

1）： 我们不会直接进行加机器，而是考虑在一台数据库中，是否有必要对表进行水平分表或者垂直分表（一对一）；

2）：如果一台计算机的CPU计算能力和磁盘的存储能力不够用的时候，我们会考虑是否提高计算机的物理硬件来解决问题

3）：如果单纯的提高一台计算机的物理硬件也满足不了庞大的系统流量和存储能力，才会考虑到增加机器，构建数据库集群来提升系统的整体并发和存储能力

4）：虽然集群可以提升一定的性能，保证服务的高可用等等，但是也会带来一定的维护成本，比如之前运维只需要维护一台mysql计算机，现在要维护给多的计算机

5）：在软件层次，还要结合业务和数据的特点，考虑如何选择分库分表策略，还要有长远的眼光做出提前计划（都是经验），分库分表策略乱用，随意用，将会导致整个系统架构越来越臃肿，维护越来越难，这甚至会直接导致项目无法运营下去；

6）：虽然用了mycat，我们可以很好的解决mysql集群中各种问题，但是是否有想过，mycat如果挂了，你的系统还都是通过mycat访问的mysql集群，这也将导致你的web(tomcat)系统直接崩溃掉

7）：所以在软件系统架构中，引进一个中间件，势必要考虑其高可用，这都会让一个简单的单体架构系统在维护成本上倍数的增长

8）：综上，软件系统架构绝对不是什么技术新就用什么，什么技术好就使用什么，我们要在系统当前阶段下，去进行技术选型，合适才是最好的，只有做好这点技术把控的架构师，对公司来说才是最好的架构师。

