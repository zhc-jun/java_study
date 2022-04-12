# Mysql进阶

### 1. 约束-(了解)

##### 01_外键约束

```mysql
-- 创建db2数据库
CREATE DATABASE db2;

-- 使用db2数据库
USE db2;

/*
	外键约束
	标准语法：
		CONSTRAINT 外键名 FOREIGN KEY (本表外键列名) REFERENCES 主表名(主表主键列名)
*/
-- 建表时添加外键约束
-- 创建user用户表
CREATE TABLE USER(
	id INT PRIMARY KEY AUTO_INCREMENT,    -- id
	NAME VARCHAR(20) NOT NULL             -- 姓名
);
-- 添加用户数据
INSERT INTO USER VALUES (NULL,'张三'),(NULL,'李四');


-- 创建orderlist订单表
CREATE TABLE orderlist(
	id INT PRIMARY KEY AUTO_INCREMENT,    -- id
	number VARCHAR(20) NOT NULL,          -- 订单编号
	uid INT,			      -- 外键列
	CONSTRAINT ou_fk1 FOREIGN KEY (uid) REFERENCES USER(id)
);
-- 添加订单数据
INSERT INTO orderlist VALUES (NULL,'hm001',1),(NULL,'hm002',1),
(NULL,'hm003',2),(NULL,'hm004',2);


-- 添加一个订单，但是没有真实用户。添加失败
INSERT INTO orderlist VALUES (NULL,'hm005',3);

-- 删除李四用户。删除失败
DELETE FROM USER WHERE NAME='李四';


/*
	删除外键约束
	标准语法：
		ALTER TABLE 表名 DROP FOREIGN KEY 外键名;
*/
-- 删除外键约束
ALTER TABLE orderlist DROP FOREIGN KEY ou_fk1;



/*
	建表后单独添加外键约束
	标准语法：
		ALTER TABLE 表名 ADD CONSTRAINT 外键名 FOREIGN KEY (本表外键列名) REFERENCES 主表名(主键列名);
*/
-- 添加外键约束
ALTER TABLE orderlist ADD CONSTRAINT ou_fk1 FOREIGN KEY (uid) REFERENCES USER(id);
```

##### 02_外键级联操作

```mysql
/*
	添加外键约束，同时添加级联更新  标准语法：
	ALTER TABLE 表名 ADD CONSTRAINT 外键名 FOREIGN KEY (本表外键列名) REFERENCES 主表名(主键列名) 
	ON UPDATE CASCADE;
	
	添加外键约束，同时添加级联删除  标准语法：
	ALTER TABLE 表名 ADD CONSTRAINT 外键名 FOREIGN KEY (本表外键列名) REFERENCES 主表名(主键列名) 
	ON DELETE CASCADE;
	
	添加外键约束，同时添加级联更新和级联删除  标准语法：
	ALTER TABLE 表名 ADD CONSTRAINT 外键名 FOREIGN KEY (本表外键列名) REFERENCES 主表名(主键列名) 
	ON UPDATE CASCADE ON DELETE CASCADE;
*/
-- 删除外键约束
ALTER TABLE orderlist DROP FOREIGN KEY ou_fk1;

-- 添加外键约束，同时添加级联更新和级联删除
ALTER TABLE orderlist ADD CONSTRAINT ou_fk1 FOREIGN KEY (uid) REFERENCES USER(id)
ON UPDATE CASCADE ON DELETE CASCADE;


-- 将李四这个用户的id修改为3,订单表中的uid也自动修改
UPDATE USER SET id=3 WHERE id=2;

-- 将李四这个用户删除,订单表中的该用户所属的订单也自动删除
DELETE FROM USER WHERE id=3;
```

### 2. 多表操作-(重要)

##### 01_表关系_一对一

```mysql
-- 创建db3数据库
CREATE DATABASE db3;

-- 使用db3数据库
USE db3;

-- 创建person表
CREATE TABLE person(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 主键id
	NAME VARCHAR(20)                        -- 姓名
);
-- 添加数据
INSERT INTO person VALUES (NULL,'张三'),(NULL,'李四');

-- 创建card表
CREATE TABLE card(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 主键id
	number VARCHAR(20) UNIQUE NOT NULL,	-- 身份证号
	pid INT UNIQUE,                     -- 外键列
	CONSTRAINT cp_fk1 FOREIGN KEY (pid) REFERENCES person(id)
);
-- 添加数据
INSERT INTO card VALUES (NULL,'12345',1),(NULL,'56789',2);
```

##### 02_表关系_一对多

```mysql
-- 创建user表
CREATE TABLE USER(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 主键id
	NAME VARCHAR(20)                        -- 姓名
);
-- 添加数据
INSERT INTO USER VALUES (NULL,'张三'),(NULL,'李四');

-- 创建orderlist表
CREATE TABLE orderlist(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 主键id
	number VARCHAR(20),                     -- 订单编号
	uid INT,				-- 外键列
	CONSTRAINT ou_fk1 FOREIGN KEY (uid) REFERENCES USER(id)
);
-- 添加数据
INSERT INTO orderlist VALUES (NULL,'hm001',1),(NULL,'hm002',1),(NULL,'hm003',2),(NULL,'hm004',2);



/*
	商品分类和商品
*/
-- 创建category表
CREATE TABLE category(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 主键id
	NAME VARCHAR(10)                        -- 分类名称
);
-- 添加数据
INSERT INTO category VALUES (NULL,'手机数码'),(NULL,'电脑办公');

-- 创建product表
CREATE TABLE product(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 主键id
	NAME VARCHAR(30),			-- 商品名称
	cid INT,				-- 外键列
	CONSTRAINT pc_fk1 FOREIGN KEY (cid) REFERENCES category(id)
);
-- 添加数据
INSERT INTO product VALUES (NULL,'华为P30',1),(NULL,'小米note3',1),
(NULL,'联想电脑',2),(NULL,'苹果电脑',2);
```

##### 03_表关系_多对多

```mysql
-- 创建student表
CREATE TABLE student(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 主键id
	NAME VARCHAR(20)			-- 学生姓名
);
-- 添加数据
INSERT INTO student VALUES (NULL,'张三'),(NULL,'李四');

-- 创建course表
CREATE TABLE course(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 主键id
	NAME VARCHAR(10)			-- 课程名称
);
-- 添加数据
INSERT INTO course VALUES (NULL,'语文'),(NULL,'数学');


-- 创建中间表
CREATE TABLE stu_course(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 主键id
	sid INT,  -- 用于和student表中的id进行外键关联
	cid INT,  -- 用于和course表中的id进行外键关联
	CONSTRAINT sc_fk1 FOREIGN KEY (sid) REFERENCES student(id), -- 添加外键约束
	CONSTRAINT sc_fk2 FOREIGN KEY (cid) REFERENCES course(id)   -- 添加外键约束
);
-- 添加数据
INSERT INTO stu_course VALUES (NULL,1,1),(NULL,1,2),(NULL,2,1),(NULL,2,2);
```

##### 04_多表查询_数据准备

```mysql
-- 创建db4数据库
CREATE DATABASE db4;
-- 使用db4数据库
USE db4;

-- 创建user表
CREATE TABLE USER(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 用户id
	NAME VARCHAR(20),			-- 用户姓名
	age INT                                 -- 用户年龄
);
-- 添加数据
INSERT INTO USER VALUES (1,'张三',23);
INSERT INTO USER VALUES (2,'李四',24);
INSERT INTO USER VALUES (3,'王五',25);
INSERT INTO USER VALUES (4,'赵六',26);


-- 订单表
CREATE TABLE orderlist(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 订单id
	number VARCHAR(30),			-- 订单编号
	uid INT,    -- 外键字段
	CONSTRAINT ou_fk1 FOREIGN KEY (uid) REFERENCES USER(id)
);
-- 添加数据
INSERT INTO orderlist VALUES (1,'hm001',1);
INSERT INTO orderlist VALUES (2,'hm002',1);
INSERT INTO orderlist VALUES (3,'hm003',2);
INSERT INTO orderlist VALUES (4,'hm004',2);
INSERT INTO orderlist VALUES (5,'hm005',3);
INSERT INTO orderlist VALUES (6,'hm006',3);
INSERT INTO orderlist VALUES (7,'hm007',NULL);


-- 商品分类表
CREATE TABLE category(
	id INT PRIMARY KEY AUTO_INCREMENT,  -- 商品分类id
	NAME VARCHAR(10)                    -- 商品分类名称
);
-- 添加数据
INSERT INTO category VALUES (1,'手机数码');
INSERT INTO category VALUES (2,'电脑办公');
INSERT INTO category VALUES (3,'烟酒茶糖');
INSERT INTO category VALUES (4,'鞋靴箱包');


-- 商品表
CREATE TABLE product(
	id INT PRIMARY KEY AUTO_INCREMENT,   -- 商品id
	NAME VARCHAR(30),                    -- 商品名称
	cid INT, -- 外键字段
	CONSTRAINT cp_fk1 FOREIGN KEY (cid) REFERENCES category(id)
);
-- 添加数据
INSERT INTO product VALUES (1,'华为手机',1);
INSERT INTO product VALUES (2,'小米手机',1);
INSERT INTO product VALUES (3,'联想电脑',2);
INSERT INTO product VALUES (4,'苹果电脑',2);
INSERT INTO product VALUES (5,'中华香烟',3);
INSERT INTO product VALUES (6,'玉溪香烟',3);
INSERT INTO product VALUES (7,'计生用品',NULL);


-- 中间表
CREATE TABLE us_pro(
	upid INT PRIMARY KEY AUTO_INCREMENT,  -- 中间表id
	uid INT, -- 外键字段。需要和用户表的主键产生关联
	pid INT, -- 外键字段。需要和商品表的主键产生关联
	CONSTRAINT up_fk1 FOREIGN KEY (uid) REFERENCES USER(id),
	CONSTRAINT up_fk2 FOREIGN KEY (pid) REFERENCES product(id)
);
-- 添加数据
INSERT INTO us_pro VALUES (NULL,1,1);
INSERT INTO us_pro VALUES (NULL,1,2);
INSERT INTO us_pro VALUES (NULL,1,3);
INSERT INTO us_pro VALUES (NULL,1,4);
INSERT INTO us_pro VALUES (NULL,1,5);
INSERT INTO us_pro VALUES (NULL,1,6);
INSERT INTO us_pro VALUES (NULL,1,7);
INSERT INTO us_pro VALUES (NULL,2,1);
INSERT INTO us_pro VALUES (NULL,2,2);
INSERT INTO us_pro VALUES (NULL,2,3);
INSERT INTO us_pro VALUES (NULL,2,4);
INSERT INTO us_pro VALUES (NULL,2,5);
INSERT INTO us_pro VALUES (NULL,2,6);
INSERT INTO us_pro VALUES (NULL,2,7);
INSERT INTO us_pro VALUES (NULL,3,1);
INSERT INTO us_pro VALUES (NULL,3,2);
INSERT INTO us_pro VALUES (NULL,3,3);
INSERT INTO us_pro VALUES (NULL,3,4);
INSERT INTO us_pro VALUES (NULL,3,5);
INSERT INTO us_pro VALUES (NULL,3,6);
INSERT INTO us_pro VALUES (NULL,3,7);
INSERT INTO us_pro VALUES (NULL,4,1);
INSERT INTO us_pro VALUES (NULL,4,2);
INSERT INTO us_pro VALUES (NULL,4,3);
INSERT INTO us_pro VALUES (NULL,4,4);
INSERT INTO us_pro VALUES (NULL,4,5);
INSERT INTO us_pro VALUES (NULL,4,6);
INSERT INTO us_pro VALUES (NULL,4,7);
```

##### 05_多表查询_内连接查询

```mysql
/*
	显示内连接
	标准语法：
		SELECT 列名 FROM 表名1 [INNER] JOIN 表名2 ON 关联条件;
*/
-- 查询用户信息和对应的订单信息
SELECT * FROM USER INNER JOIN orderlist ON orderlist.uid = user.id;


-- 查询用户信息和对应的订单信息，起别名
SELECT * FROM USER u INNER JOIN orderlist o ON o.uid=u.id;


-- 查询用户姓名，年龄。和订单编号
SELECT
	u.name,		-- 用户姓名
	u.age,		-- 用户年龄
	o.number	-- 订单编号
FROM
	USER u          -- 用户表
INNER JOIN
	orderlist o     -- 订单表
ON
	o.uid=u.id;



/*
	隐式内连接
	标准语法：
		SELECT 列名 FROM 表名1,表名2 WHERE 关联条件;
*/
-- 查询用户姓名，年龄。和订单编号
SELECT
	u.name,		-- 用户姓名
	u.age,		-- 用户年龄
	o.number	-- 订单编号
FROM
	USER u,		-- 用户表
	orderlist o     -- 订单表
WHERE
	o.uid=u.id;

```

##### 06_多表查询_外连接查询

-  on 的过滤条件是限制从属表的, where 的顾虑条件是针对关联后整个查询结果的;
- on 不能包含主表的限制条件, 加了也不起作用;
- inner join 在 on 和 where 后设置过滤条件效果是一样的

```mysql
/*
	左外连接
	标准语法：
		SELECT 列名 FROM 表名1 LEFT [OUTER] JOIN 表名2 ON 条件;
*/
-- 查询所有用户信息，以及用户对应的订单信息
SELECT
	u.*,
	o.number
FROM
	USER u
LEFT OUTER JOIN
	orderlist o
ON
	o.uid=u.id;
	

/*
	右外连接
	标准语法：
		SELECT 列名 FROM 表名1 RIGHT [OUTER] JOIN 表名2 ON 条件;
*/
-- 查询所有订单信息，以及订单所属的用户信息
SELECT
	o.*,
	u.name
FROM
	USER u
RIGHT OUTER JOIN
	orderlist o
ON
	o.uid=u.id;
```

##### 07_多表查询_子查询

```mysql
/*
	结果是单行单列的
	标准语法：
		SELECT 列名 FROM 表名 WHERE 列名=(SELECT 列名 FROM 表名 [WHERE 条件]);
*/
-- 查询年龄最高的用户姓名
SELECT MAX(age) FROM USER;
SELECT NAME,age FROM USER WHERE age=(SELECT MAX(age) FROM USER);


/*
	结果是多行单列的
	标准语法：
		SELECT 列名 FROM 表名 WHERE 列名 [NOT] IN (SELECT 列名 FROM 表名 [WHERE 条件]); 
*/
-- 查询张三和李四的订单信息
SELECT * FROM orderlist WHERE uid IN (1,2);
SELECT id FROM USER WHERE NAME IN ('张三','李四');
SELECT * FROM orderlist WHERE uid IN (SELECT id FROM USER WHERE NAME IN ('张三','李四'));


/*
	结果是多行多列的
	标准语法：
		SELECT 列名 FROM 表名 [别名],(SELECT 列名 FROM 表名 [WHERE 条件]) [别名] [WHERE 条件];
*/
-- 查询订单表中id大于4的订单信息和所属用户信息
SELECT * FROM orderlist WHERE id > 4;
SELECT
	u.name,
	o.number
FROM
	USER u,
	(SELECT * FROM orderlist WHERE id > 4) o
WHERE
	o.uid=u.id;
```

##### 08_多表查询_自关联查询

```mysql
-- 创建员工表
CREATE TABLE employee(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 员工编号
	NAME VARCHAR(20),			-- 员工姓名
	mgr INT,				-- 上级编号
	salary DOUBLE				-- 员工工资
);
-- 添加数据
INSERT INTO employee VALUES (1001,'孙悟空',1005,9000.00),
(1002,'猪八戒',1005,8000.00),
(1003,'沙和尚',1005,8500.00),
(1004,'小白龙',1005,7900.00),
(1005,'唐僧',NULL,15000.00),
(1006,'武松',1009,7600.00),
(1007,'李逵',1009,7400.00),
(1008,'林冲',1009,8100.00),
(1009,'宋江',NULL,16000.00);


-- 查询所有员工的姓名及其直接上级的姓名，没有上级的员工也需要查询
/*
分析
	员工信息 employee表
	条件：employee.mgr = employee.id
	查询左表的全部数据，和左右两张表有交集部分数据，左外连接
	OUTER 可以省略不写
*/
SELECT
	e1.id,
	e1.name,
	e1.mgr,
	e2.id,
	e2.name
FROM
	employee e1
LEFT OUTER JOIN
	employee e2
ON
	e1.mgr = e2.id;
```

##### 09_多表查询_多表查询练习

```mysql
-- 1.查询用户的编号、姓名、年龄。订单编号
/*
分析
	用户的编号、姓名、年龄  user表      订单编号 orderlist表
	条件：user.id=orderlist.uid
*/
SELECT
	u.id,
	u.name,
	u.age,
	o.number
FROM
	USER u,
	orderlist o
	
WHERE
	u.id=o.uid;



-- 2.查询所有的用户。用户的编号、姓名、年龄。订单编号
/*
分析
	用户的编号、姓名、年龄  user表    订单编号 orderlist表
	条件：user.id=orderlist.uid
	查询所有的用户，左外连接
*/
SELECT
	u.id,
	u.name,
	u.age,
	o.number
FROM
	USER u
LEFT OUTER JOIN
	orderlist o
ON
	u.id=o.uid;



-- 3.查询所有的订单。用户的编号、姓名、年龄。订单编号
/*
分析
	用户的编号、姓名、年龄 user表    订单编号 orderlist表
	条件：user.id=orderlist.uid
	查询所有的订单，右外连接
*/
-- 方式一
SELECT
    o.number,
	u.id,
	u.name,
	u.age
	
FROM
	orderlist o
LEFT OUTER JOIN
	USER u
ON
	u.id=o.uid;
	
-- 方式二	
SELECT
	u.id,
	u.name,
	u.age,
	o.number
FROM
	USER u
RIGHT OUTER JOIN
	orderlist o
ON
	u.id=o.uid;
	
	
	
-- 4.查询用户年龄大于23岁的信息。显示用户的编号、姓名、年龄。订单编号
/*
分析
	用户的编号、姓名、年龄 user表    订单编号 orderlist表
	条件：user.id=orderlist.uid AND user.age > 23
*/
SELECT
	u.id,
	u.name,
	u.age,
	o.number
FROM
	USER u,
	orderlist o
WHERE
	u.id=o.uid
	AND
	u.age > 23;


-- 5.查询张三和李四用户的信息。显示用户的编号、姓名、年龄。订单编号
/*
分析
	用户的编号、姓名、年龄 user表   订单编号 orderlist表
	条件：user.id=orderlist.uid AND user.name IN ('张三','李四')
*/
-- 方式一
SELECT
	u.id,
	u.name,
	u.age,
	o.number
FROM
	USER u,
	orderlist o
WHERE
	u.id=o.uid
	AND
	u.name IN ('张三','李四');
	
-- 方式二	注意() 号一定要有
SELECT
	u.id,
	u.name,
	u.age,
	o.number
FROM
	USER u,
	orderlist o
WHERE
	u.id=o.uid
	AND
	(u.name = '张三' OR u.name = '李四');


-- 6.查询商品分类的编号、分类名称。分类下的商品名称
/*
分析
	商品分类的编号、分类名称 category表    商品名称 product表
	条件：category.id=product.cid
*/
SELECT
	c.id,
	c.name,
	p.name
FROM
	category c,
	product p
WHERE
	c.id=p.cid;


-- 7.查询所有的商品分类。商品分类的编号、分类名称。分类下的商品名称
/*
分析
	商品分类的编号、分类名称 category表    商品名称 product表
	条件：category.id=product.cid
	查询所有的商品分类，左外连接
*/
SELECT
	c.id,
	c.name,
	p.name
FROM
	category c
LEFT OUTER JOIN
	product p
ON
	c.id=p.cid;


-- 8.查询所有的商品信息。商品分类的编号、分类名称。分类下的商品名称
/*
分析
	商品分类的编号、分类名称  category表   商品名称 product表
	条件：category.id=product.cid
	查询所有的商品信息，右外连接
*/
SELECT
	c.id,
	c.name,
	p.name
FROM
	category c
RIGHT OUTER JOIN
	product p
ON
	c.id=p.cid;



-- 9.查询所有的用户和该用户能查看的所有的商品。显示用户的编号、姓名、年龄。商品名称
/*
分析
	用户的编号、姓名、年龄 user表   商品名称 product表    中间表 us_pro
	条件：us_pro.uid=user.id AND us_pro.pid=product.id
*/
SELECT
	u.id,
	u.name,
	u.age,
	p.name
FROM
	USER u,
	product p,
	us_pro up
WHERE
	up.uid=u.id
	AND
	up.pid=p.id;


-- 10.查询张三和李四这两个用户可以看到的商品。显示用户的编号、姓名、年龄。商品名称
/*
分析
	用户的编号、姓名、年龄 user表   商品名称 product表   中间表 us_pro
	条件：us_pro.uid=user.id AND us_pro.pid=product.id AND user.name IN ('张三','李四') 
*/
SELECT
	u.id,
	u.name,
	u.age,
	p.name
FROM
	USER u,
	product p,
	us_pro up
WHERE
	up.uid=u.id
	AND
	up.pid=p.id
	AND
	u.name IN ('张三','李四');
```

### 3. 视图-(了解)

##### 01_视图_数据准备

```mysql
-- 创建db5数据库
CREATE DATABASE db5;

-- 使用db5数据库
USE db5;

-- 创建country表
CREATE TABLE country(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 国家id
	NAME VARCHAR(30)                        -- 国家名称
);
-- 添加数据
INSERT INTO country VALUES (NULL,'中国'),(NULL,'美国'),(NULL,'俄罗斯');

-- 创建city表
CREATE TABLE city(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 城市id
	NAME VARCHAR(30),			-- 城市名称
	cid INT, -- 外键列。关联country表的主键列id
	CONSTRAINT cc_fk1 FOREIGN KEY (cid) REFERENCES country(id)  -- 添加外键约束
);
-- 添加数据
INSERT INTO city VALUES (NULL,'北京',1),(NULL,'上海',1),(NULL,'纽约',2),(NULL,'莫斯科',3);
```

##### 02_视图_视图的创建和查询

```mysql
/*
	创建视图&修改
	标准语法
		CREATE VIEW 视图名称 [(列名列表)] AS 查询语句;
*/
-- 创建city_country视图，保存城市和国家的信息(使用指定列名)
CREATE VIEW city_country (city_id,city_name,country_name) AS
SELECT
	c1.id,
	c1.name,
	c2.name
FROM
	city c1,
	country c2
WHERE
	c1.cid=c2.id;
	
-- 修改视图背后的查询SQL
CREATE OR REPLACE VIEW city_country (city_id,city_name,country_name) AS
SELECT
	*
FROM
	city


	
/*
	查询视图
	标准语法
		SELECT * FROM 视图名称;
*/
-- 查询视图
SELECT * FROM city_country;
```

##### 03_视图_视图的修改和删除

```mysql
/*
	修改视图数据
	标准语法
		UPDATE 视图名称 SET 列名=值 WHERE 条件;
	
	修改视图结构
	标准语法
		ALTER VIEW 视图名称 (列名列表) AS 查询语句; 
*/
-- 修改视图数据,将北京修改为深圳。(注意：修改视图数据后，源表中的数据也会随之修改)
SELECT * FROM city_country;
UPDATE city_country SET city_name='深圳' WHERE city_name='北京';


-- 将视图中的country_name修改为name
ALTER VIEW city_country (city_id,city_name,NAME) AS
SELECT
	c1.id,
	c1.name,
	c2.name
FROM
	city c1,
	country c2
WHERE
	c1.cid=c2.id;


/*
	删除视图
	标准语法
		DROP VIEW [IF EXISTS] 视图名称;
*/
-- 删除city_country视图
DROP VIEW IF EXISTS city_country;
```

##### <font color=red>04 视图注意事项</font>

-  视图发明之初就是对SELECT语句进行包装的, 无需考虑 UPDATE  DELETE INSERT语句

- 视图是一个**虚拟表**，其内容由查询定义。同真实的表一样，视图包含一系列带有名称的列和行数据。但是，**视图并不在数据库中以存储的数据值集形式存在。行和列数据来自由定义视图的查询所引用的表，并且在使用视图时动态生成** 

-  使用视图，基表中的数据就有了一定的安全性, 这里特指真实表数据对开发不可见, 我们可以将真实表中重要的字段信息在SELECT操作中过滤掉后，用户在通过视图查询时, 将看不到这些字段，所以具备一定的安全；

-  因为视图是虚拟的，物理上是不存在的，，视图是动态的数据的集合，数据是随着真实表表的更新而更新。同时，用户对视图，不可以随意的更改虚拟表中数据(修改，删除)，因为它会同步更新真实表，不做此操作, 才可以真正保证数据的安全性。 

- **优点:**  

  > 可以定制用户数据，聚焦特定的数据 ;  
  >
  > 使用视图，可以简化开发人员对数据操作。但是会增加数据库管理员(DBA)的运维工作
  >
  > 可以让真实表中的数据具备一定的安全性; 前提: **使用得当** 

- **缺点:** 

  > 性能差,  操作视图会比直接操作基础表要慢 
  >
  > 修改限制， 当用户试图修改试图的某些信息时，数据库必须把它转化为对基本表的某些信息的修改，对于简单的试图来说，这是很方便的，但是，对于比较复杂的试图，可能是不可修改的 

**总结:**  没有特定的业务，一般我们不需要使用视图,  工作中将由架构师根据具体的业务和数据的特点来决定视图是否使用视图；一般有这样的几点思考:  

> 是否有必要使用视图
>
> 是否会影响后期的代码迭代
>
> 是否可以很好的把控，把控不当开发人员会基于视图更新特点,造成真实表数据的不安全；真实表数据的更改极其危险的
>
> 视图背后的真实表不可频繁变更表信息, 如字段名和表名, 真实表结构的改变会导致视图背后的SQL也做出调整；
>
> ......