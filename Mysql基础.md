## Mysql基础

### 今日目标

- **创建 修改 和 删除** 数据库 表 字段（指定类型）可以在SQLyog中完成操作
- 对表中数据进行 **增删改查** 操作的 SQL 关键字 + 结构，加强练习，（企业就玩这些）

- 所有约束要知道其作用，记不住SQL 可以，但是要在sqlyog中完成约束的条件。（多加练习）

### 注意事项

- 聚合函数 一般不与 其他字段一起查询
- 聚合函数不能作为 where 条件进行过滤
- order by 排序，如果有 过滤条件 要写在 过滤条件的后面, 没有过滤条件，直接写在表名的后面即可。





### 介绍:  

- mysql 是一个数据库软件, db2    oralce   postgresql(pgsql)   sql server

  >（1）mysql软件中可以创建N多个数据库:  数据库用来存放表
  >
  >（2）每一个数据库中可以创建N多个表 :  表用来存放数据
  >
  >（3）每一个表可以创建N多个 字段(列) : 用来跟数据一一对应
  >
  >（4）每一个字段都有自己唯一对应的类型: int  bigint  char  date datetime varchar .....
  >
  >  (5)  mysql 软件 简称 DataBase （DB）--> DataBaseAdmin (DBA)



### 1. 操作数据库-(练一练)

```mysql
/*
	查询所有数据库
	标准语法：
		SHOW DATABASES;
*/
-- 查询所有数据库
SHOW DATABASES;


/*
	查询某个数据库的创建语句
	标准语法：
		SHOW CREATE DATABASE 数据库名称;
*/
-- 查询mysql数据库的创建语句
SHOW CREATE DATABASE mysql;


/*
	创建数据库
	标准语法：
		CREATE DATABASE 数据库名称;
*/
-- 创建db1数据库
CREATE DATABASE db1;


/*
	创建数据库，判断、如果不存在则创建
	标准语法：
		CREATE DATABASE IF NOT EXISTS 数据库名称;
*/
-- 创建数据库db2(判断，如果不存在则创建)
CREATE DATABASE IF NOT EXISTS db2;


/*
	创建数据库、并指定字符集
	标准语法：
		CREATE DATABASE 数据库名称 CHARACTER SET 字符集名称;
*/
-- 创建数据库db3、并指定字符集utf8
CREATE DATABASE db3 CHARACTER SET utf8;

-- 查看db3数据库的字符集
SHOW CREATE DATABASE db3;


-- 练习：创建db4数据库、如果不存在则创建，指定字符集为gbk
CREATE DATABASE IF NOT EXISTS db4 CHARACTER SET gbk;


-- 查看db4数据库的字符集
SHOW CREATE DATABASE db4;


/*
	修改数据库的字符集
	标准语法：
		ALTER DATABASE 数据库名称 CHARACTER SET 字符集名称;
*/
-- 修改数据库db4的字符集为utf8
ALTER DATABASE db4 CHARACTER SET utf8;

-- 查看db4数据库的字符集
SHOW CREATE DATABASE db4;


/*
	删除数据库
	标准语法：
		DROP DATABASE 数据库名称;
*/
-- 删除db1数据库
DROP DATABASE db1;

/*
	删除数据库，判断、如果存在则删除
	标准语法：
		DROP DATABASE IF EXISTS 数据库名称;
*/
-- 删除数据库db2，如果存在
DROP DATABASE IF EXISTS db2;


/*
	使用数据库
	标准语法：
		USE 数据库名称;
*/
-- 使用db4数据库
USE db4;

/*
	查询当前使用的数据库
	标准语法：
		SELECT DATABASE();
*/
-- 查询当前正在使用的数据库
SELECT DATABASE();

-- 使用mysql数据库
USE mysql;

/*
	查询所有数据表
	标准语法：
		SHOW TABLES;
*/
-- 查询库中所有的表
SHOW TABLES;

/*
	查询表结构
	标准语法：
		DESC 表名;
*/
-- 查询user表结构
DESC USER;

/*
	查询数据表的字符集
	标准语法：
		SHOW TABLE STATUS FROM 数据库名称 LIKE '表名';
*/
-- 查看mysql数据库中user表字符集
SHOW TABLE STATUS FROM mysql LIKE 'user';






/*
	创建数据表
	标准语法：
		CREATE TABLE 表名(
			列名 数据类型 约束,
			列名 数据类型 约束,
			...
			列名 数据类型 约束
		);
*/
-- 创建一个product商品表(商品编号、商品名称、商品价格、商品库存、上架时间) -----重点
CREATE TABLE  product(
	id INT,
	NAME VARCHAR(20),
	price DOUBLE,
	stock INT,
	insert_time DATE
);

-- 查看product表详细结构
DESC product;



/*
	修改表名
	标准语法：
		ALTER TABLE 旧表名 RENAME TO 新表名;
*/
-- 修改product表名为product2
ALTER TABLE product RENAME TO product2;



/*
	修改表的字符集
	标准语法：
		ALTER TABLE 表名 CHARACTER SET 字符集名称;
*/
-- 查看db3数据库中product2数据表字符集
SHOW TABLE STATUS FROM db3 LIKE 'product2';

-- 修改product2数据表字符集为gbk
ALTER TABLE product2 CHARACTER SET gbk;


/*
	给表添加列
	标准语法：
		ALTER TABLE 表名 ADD 列名 数据类型;
*/
-- 给product2表添加一列color
ALTER TABLE product2 ADD color VARCHAR(10);


/*
	修改表中列的数据类型
	标准语法：
		ALTER TABLE 表名 MODIFY 列名 数据类型;
*/
-- 将color数据类型修改为int
ALTER TABLE product2 MODIFY color INT;

-- 查看product2表详细信息
DESC product2;

/*
	修改表中列的名称和数据类型
	标准语法：
		ALTER TABLE 表名 CHANGE 旧列名 新列名 数据类型;
*/
-- 将color修改为address
ALTER TABLE product2 CHANGE color address VARCHAR(200);

-- 查看product2表详细信息


/*
	删除表中的列
	标准语法：
		ALTER TABLE 表名 DROP 列名;
*/
-- 删除address列
ALTER TABLE product2 DROP address;


/*
	删除表
	标准语法：
		DROP TABLE 表名;
*/
-- 删除product2表
DROP TABLE product2;

/*
	删除表，判断、如果存在则删除
	标准语法：
		DROP TABLE IF EXISTS 表名;
*/
-- 删除product2表，如果存在则删除
DROP TABLE IF EXISTS product2;

```

### 2. 表数据的增删改-(重点)

```mysql
/*
	给指定列添加数据
	标准语法：
		INSERT INTO 表名(列名1,列名2,...) VALUES (值1,值2,...);
*/
-- 向product表添加一条数据
INSERT INTO product (id,NAME,price,stock,insert_time) VALUES (1,'手机',1999.99,25,'2020-02-02');


-- 向product表添加指定列数据
INSERT INTO product (id,NAME,price) VALUES (2,'电脑',3999.99);

/*
	给全部列添加数据
	标准语法：
		INSERT INTO 表名 VALUES (值1,值2,值3,...);
*/
-- 默认给全部列添加数据
INSERT INTO product VALUES (3,'冰箱',1500,35,'2030-03-03');

/*
	批量添加所有列数据
	标准语法：
		INSERT INTO 表名 VALUES (值1,值2,值3,...),(值1,值2,值3,...),(值1,值2,值3,...);
*/
-- 批量添加数据
INSERT INTO product VALUES (4,'洗衣机',800,15,'2030-05-05'),(5,'微波炉',300,45,'2030-06-06');


/*
	修改表数据
	标准语法：
		UPDATE 表名 SET 列名1 = 值1,列名2 = 值2,... [where 条件];
*/
-- 修改手机的价格为3500
UPDATE product SET price=3500 WHERE NAME='手机';

-- 修改电脑的价格为1800、库存为36
UPDATE product SET price=1800,stock=36 WHERE NAME='电脑';


/*
	删除表数据
	标准语法：
		DELETE FROM 表名 [WHERE 条件];
*/
-- 删除product表中的微波炉信息
DELETE FROM product WHERE NAME='微波炉';

-- 删除product表中库存为10的商品信息
DELETE FROM product WHERE stock=10;
```

### 3. 表数据的查询-(重点)

#### 3.1_查询_数据准备

```mysql
-- 创建db1数据库
CREATE DATABASE db1;

-- 使用db1数据库
USE db1;

-- 创建数据表
CREATE TABLE product(
	id INT,			-- 商品编号
	NAME VARCHAR(20),	-- 商品名称
	price DOUBLE,		-- 商品价格
	brand VARCHAR(10),	-- 商品品牌
	stock INT,		-- 商品库存
	insert_time DATE        -- 添加时间
);

-- 添加数据
INSERT INTO product VALUES 
(1,'华为手机',3999,'华为',23,'2088-03-10'),
(2,'小米手机',2999,'小米',30,'2088-05-15'),
(3,'苹果手机',5999,'苹果',18,'2088-08-20'),
(4,'华为电脑',6999,'华为',14,'2088-06-16'),
(5,'小米电脑',4999,'小米',26,'2088-07-08'),
(6,'苹果电脑',8999,'苹果',15,'2088-10-25'),
(7,'联想电脑',7999,'联想',NULL,'2088-11-11');

注：如果插入中文字 插不进去报错的话  先展示show create table 表名;---看表列字段字符编码是utf8
    如果不是 就将 所有列字段 alter table 表名 convert to character set utf8; 设置一下
```

#### 3.2_查询_查询全部

```mysql
/*
	分页查询
	标准语法：
		SELECT 列名 FROM 表名 
		[WHERE 条件] 
		[GROUP BY 分组列名]
		[HAVING 分组后条件过滤] 
		[ORDER BY 排序列名 排序方式] 
		LIMIT 当前页数,每页显示的条数;
	
	LIMIT 当前页数,每页显示的条数;
	公式：当前页数 = (当前页数-1) * 每页显示的条数
*/

#######################################################################
/*
	查询全部数据
	标准语法：
		SELECT * FROM 表名;
*/
-- 查询product表所有数据
SELECT * FROM product;


/*
	查询指定列
	标准语法：
		SELECT 列名1,列名2,... FROM 表名;
*/
-- 查询名称、价格、品牌
SELECT NAME,price,brand FROM product;


/*
	去除重复查询
	标准语法：
		SELECT DISTINCT 列名1,列名2,... FROM 表名;
*/
-- 查询品牌
SELECT brand FROM product;
-- 查询品牌，去除重复
SELECT DISTINCT brand FROM product;



/*
	计算列的值
	标准语法：
		SELECT 列名1 运算符(+ - * /) 列名2 FROM 表名;
		
	如果某一列为null，可以进行替换
	ifnull(表达式1,表达式2)
	表达式1：想替换的列
	表达式2：想替换的值
*/
-- 查询商品名称和库存，库存数量在原有基础上加10
SELECT NAME,stock+10 FROM product;

-- 查询商品名称和库存，库存数量在原有基础上加10。进行null值判断
SELECT NAME,IFNULL(stock,0)+10 FROM product;


/*
	起别名
	标准语法：
		SELECT 列名1,列名2,... AS 别名 FROM 表名;
*/
-- 查询商品名称和库存，库存数量在原有基础上加10。进行null值判断。起别名为getSum
SELECT NAME,IFNULL(stock,0)+10 AS getSum FROM product;
SELECT NAME,IFNULL(stock,0)+10 getSum FROM product;
```

#### 3.3_查询_条件查询

```mysql
/*
	条件查询
	标准语法：
		SELECT 列名列表 FROM 表名 WHERE 条件;
*/
-- 查询库存大于20的商品信息
SELECT * FROM product WHERE stock > 20;


-- 查询品牌为华为的商品信息
SELECT * FROM product WHERE brand='华为';

-- 查询金额在4000 ~ 6000之间的商品信息
SELECT * FROM product WHERE price >= 4000 AND price <= 6000;
SELECT * FROM product WHERE price BETWEEN 4000 AND 6000;


-- 查询库存为14、30、23的商品信息
SELECT * FROM product WHERE stock=14 OR stock=30 OR stock=23;
SELECT * FROM product WHERE stock IN(14,30,23);

-- 查询库存为null的商品信息
SELECT * FROM product WHERE stock IS NULL;

-- 查询库存不为null的商品信息
SELECT * FROM product WHERE stock IS NOT NULL;


-- 查询名称以小米为开头的商品信息
SELECT * FROM product WHERE NAME LIKE '小米%';

-- 查询名称第二个字是为的商品信息
SELECT * FROM product WHERE NAME LIKE '_为%';

-- 查询名称为四个字符的商品信息
SELECT * FROM product WHERE NAME LIKE '____';

-- 查询名称中包含电脑的商品信息
SELECT * FROM product WHERE NAME LIKE '%电脑%';

```

#### 3.4_查询_聚合函数

```mysql
/*
	聚合函数
	标准语法：
		SELECT 函数名(列名) FROM 表名 [WHERE 条件];
*/
-- 计算product表中总记录条数
SELECT COUNT(*) FROM product;

-- 获取最高价格
SELECT MAX(price) FROM product;


-- 获取最低库存
SELECT MIN(stock) FROM product;


-- 获取总库存数量
SELECT SUM(stock) FROM product;


-- 获取品牌为苹果的总库存数量
SELECT SUM(stock) FROM product WHERE brand='苹果';

-- 获取品牌为小米的平均商品价格
SELECT AVG(price) FROM product WHERE brand='小米';

```

#### 3.5_查询_排序查询

```mysql
/*
	排序查询
	标准语法：
		SELECT 列名 FROM 表名 [WHERE 条件] ORDER BY 列名1 排序方式1,列名2 排序方式2;
*/
-- 按照库存升序排序
SELECT * FROM product ORDER BY stock ASC;

-- 查询名称中包含手机的商品信息。按照金额降序排序
SELECT * FROM product WHERE NAME LIKE '%手机%' ORDER BY price DESC;

-- 按照金额升序排序，如果金额相同，按照库存降序排列
SELECT * FROM product ORDER BY price ASC,stock DESC;
```

#### 3.6_查询_分组查询

```mysql
/*
	分组查询
	标准语法：
		SELECT 列名 FROM 表名 [WHERE 条件] GROUP BY 分组列名 [HAVING 分组后条件过滤] [ORDER BY 排序列名 排序方式];
*/
-- 按照品牌分组，获取每组商品的总金额
SELECT brand,SUM(price) FROM product GROUP BY brand;

-- 对金额大于4000元的商品，按照品牌分组,获取每组商品的总金额
SELECT brand,SUM(price) FROM product WHERE price > 4000 GROUP BY brand;

-- 对金额大于4000元的商品，按照品牌分组，获取每组商品的总金额，只显示总金额大于7000元的
SELECT brand,SUM(price) as getSum FROM product WHERE price > 4000 GROUP BY brand HAVING getSum > 7000;

-- 对金额大于4000元的商品，按照品牌分组，获取每组商品的总金额，只显示总金额大于7000元的、并按照总金额的降序排列
SELECT brand,SUM(price) getSum FROM product 
WHERE price > 4000 
GROUP BY brand 
HAVING getSum > 7000 
ORDER BY getSum DESC;
```

#### 3.7_查询_分页查询

```mysql
/*
	分页查询
	标准语法：
		LIMIT n ; limit 1; 表示我只获取一条数据
		LIMIT m, n; 表示我从 m+1 开始查, 我只差n条
		LIMIT 10, 10; 从第11条开始查，往后查10条；
	
	前提: 每页展示rows=10条; 表中共有35条;
	第一页: 0, 10;
	第二页: 10, 10;
	第三页: 20, 10;
	
	LIMIT 当前页数,每页显示的条数;
	m = (current_page - 1) * n;
	--公式：当前页数 = (当前页数-1) * 每页显示的条数
	
*/
-- 每页显示3条数据

-- 第1页  当前页数=(1-1) * 3
SELECT * FROM product LIMIT 0,3;

-- 第2页  当前页数=(2-1) * 3
SELECT * FROM product LIMIT 3,3;

-- 第3页  当前页数=(3-1) * 3
SELECT * FROM product LIMIT 6,3;

```

### 4. 约束-(重点)

#### 4.1_约束_主键约束

```mysql
-- 创建学生表(编号、姓名、年龄)  编号设为主键
CREATE TABLE student(
	id INT PRIMARY KEY,
	NAME VARCHAR(30),
	age INT
);

-- 查询学生表的详细信息
DESC student;

-- 添加数据
INSERT INTO student VALUES (1,'张三',23);
INSERT INTO student VALUES (2,'李四',24);

-- 删除主键
ALTER TABLE student DROP PRIMARY KEY;


-- 建表后单独添加主键约束
ALTER TABLE student MODIFY id INT PRIMARY KEY;
```

#### 4.2_约束_主键自增约束

```mysql
-- 创建学生表(编号、姓名、年龄)  编号设为主键自增
CREATE TABLE student(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(30),
	age INT
);

-- 查询学生表的详细信息
DESC student;

-- 添加数据
INSERT INTO student VALUES (NULL,'张三',23),(NULL,'李四',24);

-- 删除自增约束
ALTER TABLE student MODIFY id INT;
INSERT INTO student VALUES (NULL,'张三',23);

-- 建表后单独添加自增约束
ALTER TABLE student MODIFY id INT AUTO_INCREMENT;
```

#### 4.3_约束_唯一约束

```mysql
-- 创建学生表(编号、姓名、年龄)  编号设为主键自增，年龄设为唯一
CREATE TABLE student(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(30),
	age INT UNIQUE
);

-- 查询学生表的详细信息
DESC student;

-- 添加数据
INSERT INTO student VALUES (NULL,'张三',23);
INSERT INTO student VALUES (NULL,'李四',23);

-- 删除唯一约束
ALTER TABLE student DROP INDEX age;

-- 建表后单独添加唯一约束
ALTER TABLE student MODIFY age INT UNIQUE;
```

#### 4.4_约束_非空约束

```mysql
-- 创建学生表(编号、姓名、年龄)  编号设为主键自增，姓名设为非空，年龄设为唯一
CREATE TABLE student(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(30) NOT NULL,
	age INT UNIQUE
);

-- 查询学生表的详细信息
DESC student;

-- 添加数据
INSERT INTO student VALUES (NULL,'张三',23);

-- 删除非空约束
ALTER TABLE student MODIFY NAME VARCHAR(30);
INSERT INTO student VALUES (NULL,NULL,25);

-- 建表后单独添加非空约束
ALTER TABLE student MODIFY NAME VARCHAR(30) NOT NULL;
```

