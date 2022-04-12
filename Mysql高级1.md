# Mysql高级1

### 今日目标

- 动手练一练，临时开启事物, 转账案例中，对多条update  SQL 进行管理，错误后 回顾，没错误提交事物。

- 能够说出事物的作用和特点。

  答：作用：保证数据的一致性，特点：事物中的多条对数据有变更的SQL，要么都执行（commit），要么都不执行(rollback)。



### 1. 存储过程&函数-(了解)

- 在mysql数据库端编写带有逻辑的代码, 可以在客户端直接调用, java端可以写更少的代码

- **存储过程没有返回值,  函数必须有返回值**

- 优点: 

  > 1.简化我们java端的业务逻辑
  >
  > 2.数据都在数据库中存储, 在数据库端处理业务, 理论上讲效率更高, 毕竟减少了客户端和mysql 服务端频繁交互

- 注意: 

  > 1.虽然存储过程可以避免客户端和mysql的频繁交互, 但是不应该过于依赖存储过程解决业务问题, 毕竟数据库就是用来存储数据的, 不应该针对系统的某块业务而耗费更多的数据库性能, 这可能会造成数据库整体性能变慢
  >
  > 2.存储过程会带来一定的学习成本, 一般java程序员接触不深, 很难进行迭代, 会延长软件开发周期, 毕竟存储过程相关的业务很少,  使用不够频繁
  >
  > 3.在企业中，存储过程应该由架构师来决定是否使用, 而不应该由经验不多的人，随意乱用；否则会带来系统难以维护

##### 01_存储过程_--数据准备

```mysql
-- 创建db8数据库
CREATE DATABASE db8;

-- 使用db8数据库
USE db8;

-- 创建学生表
CREATE TABLE student(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 学生id
	NAME VARCHAR(20),			-- 学生姓名
	age INT,				-- 学生年龄
	gender VARCHAR(5),			-- 学生性别
	score INT                               -- 学生成绩
);
-- 添加数据
INSERT INTO student VALUES (NULL,'张三',23,'男',95),(NULL,'李四',24,'男',98),
(NULL,'王五',25,'女',100),(NULL,'赵六',26,'女',90);


-- 按照性别进行分组，查询每组学生的总成绩。按照总成绩的升序排序
SELECT gender,SUM(score) getSum FROM student GROUP BY gender ORDER BY getSum ASC;
```

##### 02_存储过程--创建存储过程

```mysql
/*
	创建存储过程

	-- 修改分隔符为$
	DELIMITER $

	-- 标准语法
	CREATE PROCEDURE 存储过程名称(参数...)
	BEGIN
		sql语句;
	END$

	-- 修改分隔符为分号
	DELIMITER ;
*/
-- 创建stu_group()存储过程，封装 分组查询总成绩，并按照总成绩升序排序的功能
DELIMITER $

CREATE PROCEDURE stu_group()
BEGIN
	SELECT gender,SUM(score) getSum FROM student GROUP BY gender ORDER BY getSum ASC;
		
END$

DELIMITER ;
```

##### 03_存储过程_--调用存储过程

```mysql
/*
	调用存储过程
	标准语法
		CALL 存储过程名称(实际参数);
*/
-- 调用stu_group存储过程
CALL stu_group();
```

##### 04_存储过程--查看存储过程

```mysql
/*
	查询数据库中所有的存储过程
	标准语法
		SELECT name FROM mysql.proc WHERE db='数据库名称';
*/
-- 查看db8数据库中所有的存储过程名字
SELECT NAME FROM mysql.proc WHERE db='db8';


/*
	查看存储过程的状态信息
	标准语法
		SHOW PROCEDURE STATUS;
*/
-- 查看存储过程的状态信息
SHOW PROCEDURE STATUS;


/*
	查看存储过程的创建语句
	标准语法
		SHOW CREATE PROCEDURE 存储过程名称;
*/
-- 查询stu_group存储过程的创建语句
SHOW CREATE PROCEDURE stu_group;
```

##### 05_存储过程_--删除存储过程

```mysql
/*
	删除存储过程
	标准语法
		DROP PROCEDURE [IF EXISTS] 存储过程名称;
*/
-- 删除stu_group存储过程
DROP PROCEDURE IF EXISTS stu_group;
```

##### 06_存储过程--变量

```mysql
/*
	定义变量
	标准语法
		DECLARE 变量名 数据类型 [DEFAULT 默认值];
*/
-- 定义一个int类型变量，并赋默认值为10
DELIMITER $

CREATE PROCEDURE pro_test1()
BEGIN
	-- 定义变量 int num = 10;
	DECLARE num INT DEFAULT 10;
	-- 查询变量
	SELECT num;
END$

DELIMITER ;

-- 调用pro_test1存储过程
CALL pro_test1();


/*
	变量赋值-方式一
	标准语法
		SET 变量名 = 变量值;
*/
-- 定义一个varchar类型变量并赋值
DELIMITER $

CREATE PROCEDURE pro_test2()
BEGIN
	-- 定义变量
	DECLARE NAME VARCHAR(10);
	-- 为变量赋值
	SET NAME = '存储过程';
	-- 查询变量
	SELECT NAME;
END$

DELIMITER ;

-- 调用pro_test2存储过程
CALL pro_test2();



/*
	变量赋值-方式二
	标准语法
		SELECT 列名 INTO 变量名 FROM 表名 [WHERE 条件];
*/
-- 定义两个int变量，用于存储男女同学的总分数
DELIMITER $

CREATE PROCEDURE pro_test3()
BEGIN
	-- 定义变量
	DECLARE men,women INT;
	-- 查询男同学的总分数，为men变量赋值
	SELECT SUM(score) INTO men FROM student WHERE gender='男';
	-- 查询女同学的总分数，为women变量赋值
	SELECT SUM(score) INTO women FROM student WHERE gender='女';
	-- 查询变量
	SELECT men,women;
END$

DELIMITER ;

-- 调用pro_test3存储过程
CALL pro_test3();
```

##### 07_存储过程_--if语句

```mysql
/*
	存储过程-if语句
	标准语法
		IF 判断条件1 THEN 执行的sql语句1;
		[ELSEIF 判断条件2 THEN 执行的sql语句2;]
		...
		[ELSE 执行的sql语句n;]
		END IF;
*/

/*
	定义一个int变量，用于存储班级总成绩
	定义一个varchar变量，用于存储分数描述
	根据总成绩判断：
		380分及以上   学习优秀
		320 ~ 380     学习不错
		320以下       学习一般
*/
DELIMITER $

CREATE PROCEDURE pro_test4()
BEGIN
	-- 定义保存总成绩的变量
	DECLARE total INT;
	-- 定义保存分数描述的变量
	DECLARE description VARCHAR(10);
	-- 为total赋值
	SELECT SUM(score) INTO total FROM student;
	-- 判断总成绩
	IF total >= 380 THEN
		SET description = '学习优秀';
	ELSEIF total >= 320 AND total < 380 THEN
		SET description = '学习不错';
	ELSE
		SET description = '学习一般';
	END IF;
	
	-- 查询总成绩和描述信息
	SELECT total,description;
END$

DELIMITER ;

-- 调用pro_test4存储过程
CALL pro_test4();
```

##### 08_存储过程_--输入参数

- 多个参数用逗号隔开即可

```mysql
/*
	输入参数
	
	DELIMITER $

	-- 标准语法
	CREATE PROCEDURE 存储过程名称(IN 参数名 数据类型)
	BEGIN
		执行的sql语句;
	END$

	DELIMITER ;
*/
/*
	输入总成绩变量，代表学生总成绩
	定义一个varchar变量，用于存储分数描述
	根据总成绩判断：
		380分及以上  学习优秀
		320 ~ 380    学习不错
		320以下      学习一般
*/
DELIMITER $

CREATE PROCEDURE pro_test5(IN total INT)
BEGIN
	-- 定义分数描述变量
	DECLARE description VARCHAR(10);
	-- 判断总成绩
	IF total >= 380 THEN
		SET description = '学习优秀';
	ELSEIF total >= 320 AND total < 380 THEN
		SET description = '学习不错';
	ELSE
		SET description = '学习一般';
	END IF;
	
	-- 查询总成绩和描述信息
	SELECT total,description;
END$

DELIMITER ;

-- 调用pro_test5存储过程
CALL pro_test5(310);

CALL pro_test5((SELECT SUM(score) FROM student));
```

##### 09_存储过程--_输出参数

- 多个输出参数用逗号隔开

```mysql
/*
	输出参数
	
	DELIMITER $

	-- 标准语法
	CREATE PROCEDURE 存储过程名称(OUT 参数名 数据类型)
	BEGIN
		执行的sql语句;
	END$

	DELIMITER ;
*/
/*
	输入总成绩变量，代表学生总成绩
	输出分数描述变量，代表学生总成绩的描述
	根据总成绩判断：
		380分及以上  学习优秀
		320 ~ 380    学习不错
		320以下      学习一般
*/
DELIMITER $

CREATE PROCEDURE pro_test6(IN total INT,OUT description VARCHAR(10), OUT stu_name VARCHAR(10))
BEGIN
    -- 给stu_name赋值
    SET stu_name = '传智黑马-java学生';
    
	-- 判断总成绩
	IF total >= 380 THEN
		SET description = '学习优秀';
	ELSEIF total >= 320 AND total < 380 THEN
		SET description = '学习不错';
	ELSE
		SET description = '学习一般';
	END IF;
	
END$

DELIMITER ;

-- 调用pro_test6存储过程
CALL pro_test6(390,@description,@stu_name);

-- 查询总成绩描述信息变量
SELECT @description,@stu_name;
```

##### 10_存储过程_--case语句-(了解)

- 功能与if类似，了解即可

```mysql
/*
	case语句
	标准语法
		CASE
		WHEN 判断条件1 THEN 执行sql语句1;
		[WHEN 判断条件2 THEN 执行sql语句2;]
		...
		[ELSE 执行sql语句n;]
		END CASE;
*/
/*
	输入总成绩变量，代表学生总成绩
	定义一个varchar变量，用于存储分数描述
	根据总成绩判断：
		380分及以上  学习优秀
		320 ~ 380    学习不错
		320以下      学习一般
*/
DELIMITER $

CREATE PROCEDURE pro_test7(IN total INT)
BEGIN
	-- 定义总成绩描述信息变量
	DECLARE description VARCHAR(10);
	-- 判断总成绩
	CASE
	WHEN total >= 380 THEN
		SET description = '学习优秀';
	WHEN total >= 320 AND total < 380 THEN
		SET description = '学习不错';
	ELSE
		SET description = '学习一般';
	END CASE;
	
	-- 查询总成绩和描述信息
	SELECT total,description;
END$

DELIMITER ;

-- 调用pro_test7存储过程
CALL pro_test7(310);
```

##### 11_存储过程_--while循环

```mysql
/*
	while循环
	标准语法
		初始化语句;
		WHILE 条件判断语句 DO
			循环体语句;
			条件控制语句;
		END WHILE;
*/
-- 计算1~100之间的偶数和
DELIMITER $

CREATE PROCEDURE pro_test8()
BEGIN
	-- 定义求和变量
	DECLARE result INT DEFAULT 0;
	-- 定义初始化变量
	DECLARE num INT DEFAULT 1;
	-- while循环
	WHILE num <= 100 DO
		-- 判断num是否是偶数
		IF num%2=0 THEN
			-- 让num和result进行累加
			SET result = result + num;
		END IF;
		
		-- 让num进行自增
		SET num = num + 1;
	END WHILE;
	
	-- 查询求和结果
	SELECT result;
END$

DELIMITER ;

-- 调用pro_test8存储过程
CALL pro_test8();
```

##### 12存储--函数-(了解)

```mysql
/*
	创建存储函数
	标准语法
		CREATE FUNCTION 函数名称([参数 数据类型])
		RETURNS 返回值类型
		BEGIN
			执行的sql语句;
			RETURN 结果;
		END$
*/
-- 定义存储函数，获取学生表中成绩大于95分的学生数量
DELIMITER $

CREATE FUNCTION fun_test1()
RETURNS INT
BEGIN
	-- 定义一个统计变量
	DECLARE result INT;
	
	-- 查询成绩大于95分的数量
	SELECT COUNT(*) INTO result FROM student WHERE score > 95;
	
	-- 将结果返回
	RETURN result;
END$

DELIMITER ;


/*
	调用函数
	标准语法
		SELECT 函数名称(实际参数);
*/
-- 调用函数
SELECT fun_test1();


/*
	删除函数
	标准语法
		DROP FUNCTION 函数名称;
*/
-- 删除函数
DROP FUNCTION fun_test1;
```



### 2. Mysql--触发器-(了解)

##### 01_触发器--_触发器的数据准备

```mysql
-- 创建db9数据库
CREATE DATABASE db9;

-- 使用db9数据库
USE db9;

-- 创建账户表account
CREATE TABLE account(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 账户id
	NAME VARCHAR(20),			-- 姓名
	money DOUBLE				-- 余额
);
-- 添加数据
INSERT INTO account VALUES (NULL,'张三',1000),(NULL,'李四',2000);


-- 创建日志表account_log
CREATE TABLE account_log(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 日志id
	operation VARCHAR(20),			-- 操作类型 (insert update delete)
	operation_time DATETIME,		-- 操作时间
	operation_id INT,			-- 操作表的id
	operation_params VARCHAR(200)       	-- 操作参数
);
```

##### 02_触发器_--INSERT型触发器

- CONCAT() 函数字符串拼接

```mysql
/*
	创建触发器
	标准语法
		DELIMITER $

		CREATE TRIGGER 触发器名称
		BEFORE|AFTER INSERT|UPDATE|DELETE
		ON 表名
		FOR EACH ROW
		BEGIN
			触发器要执行的功能;
		END$

		DELIMITER ;
*/
-- 创建INSERT型触发器。用于对account表新增数据进行日志的记录
DELIMITER $

CREATE TRIGGER account_insert
AFTER INSERT
ON account
FOR EACH ROW
BEGIN
	INSERT INTO account_log VALUES 
	(NULL,'INSERT',NOW(),new.id,CONCAT('插入后{id=',new.id,',name=',new.name,',money=',new.money,'}'));
	
END$

DELIMITER ;

-- 向account表来添加一条记录
INSERT INTO account VALUES (NULL,'王五',3000);

-- 查询account表
SELECT * FROM account;

-- 查询account_log表
SELECT * FROM account_log;
```

##### 03_触发器_--UPDATE型触发器

```mysql
/*
	创建触发器
	标准语法
		DELIMITER $

		CREATE TRIGGER 触发器名称
		BEFORE|AFTER INSERT|UPDATE|DELETE
		ON 表名
		FOR EACH ROW
		BEGIN
			触发器要执行的功能;
		END$

		DELIMITER ;
*/
-- 创建UPDATE型触发器。用于对account表修改数据进行日志的记录
DELIMITER $

CREATE TRIGGER account_update
AFTER UPDATE
ON account
FOR EACH ROW
BEGIN
	INSERT INTO account_log VALUES 
	(NULL,'UPDATE',NOW(),new.id,CONCAT('修改前{id=',old.id,',name=',old.name,',money=',old.money,'}','修改后{id=',new.id,',name=',new.name,',money=',new.money,'}'));
END$

DELIMITER ;

-- 修改account表中李四的金额为2500
UPDATE account SET money=2500 WHERE NAME='李四';

-- 查询account表
SELECT * FROM account;

-- 查询account_log表
SELECT * FROM account_log;
```

##### 04_触发器_--DELETE型触发器

```mysql
/*
	创建触发器
	标准语法
		DELIMITER $

		CREATE TRIGGER 触发器名称
		BEFORE|AFTER INSERT|UPDATE|DELETE
		ON 表名
		FOR EACH ROW
		BEGIN
			触发器要执行的功能;
		END$

		DELIMITER ;
*/
-- 创建DELETE型触发器。用于对account表删除数据进行日志的记录
DELIMITER $

CREATE TRIGGER account_delete
AFTER DELETE
ON account
FOR EACH ROW
BEGIN
	INSERT INTO account_log VALUES 
	(NULL,'DELETE',NOW(),old.id,CONCAT('删除前{id=',old.id,',name=',old.name,',money=',old.money,'}'));
END$

DELIMITER ;

-- 删除account表中王五
DELETE FROM account WHERE NAME='王五';

-- 查询account表
SELECT * FROM account;

-- 查询account_log表
SELECT * FROM account_log;
```

##### 05触发器--查看&删除

```mysql
/*
	查看触发器
	标准语法
		SHOW TRIGGERS;
*/
-- 查看触发器
SHOW TRIGGERS;



/*
	删除触发器
	标准语法
		DROP TRIGGER 触发器名称;
*/
-- 删除触发器
DROP TRIGGER account_delete;
```



### 3. Mysql--事物

- 事物是保证数据库数据安全的一种手段

- 在执行多条对表中数据存在改动的SQL时, 一定要加事物, 取保多条SQL要么都执行, 要么都不执行

- 实现步骤: 

  > 开启事物    start transaction    
  >
  > 提交 事物   commit 
  >
  >  回滚事物    rollback

##### 01_事务_--事务的数据准备

```mysql
-- 创建db10数据库
CREATE DATABASE db10;

-- 使用db10数据库
USE db10;

-- 创建账户表
CREATE TABLE account(
	id INT PRIMARY KEY AUTO_INCREMENT,	-- 账户id
	NAME VARCHAR(20),			-- 账户名称
	money DOUBLE				-- 账户余额
);
-- 添加数据
INSERT INTO account VALUES (NULL,'张三',1000),(NULL,'李四',1000);
```

##### <font color=red>02_事务--未管理事务-演示-(重点)</font>

```mysql
-- 张三给李四转账500元
-- 1.张三账户-500
UPDATE account SET money=money-500 WHERE NAME='张三';

出错了...

-- 2.李四账户+500
UPDATE account SET money=money+500 WHERE NAME='李四';
```

##### <font color=red>03_事务--_管理事务-(重点)</font>

```mysql
/*
	开启事务：START TRANSACTION;
	回滚事务：ROLLBACK;
	提交事务：COMMIT;
*/
-- 张三给李四转账500元
-- 1.开启事务
START TRANSACTION;

-- 2.执行转账的操作
UPDATE account SET money=money-500 WHERE NAME='张三';
出错了...
UPDATE account SET money=money+500 WHERE NAME='李四';

-- 3.结束事务
-- 3.1回滚事务
ROLLBACK;

-- 3.2提交事务
COMMIT;
```

##### 04_事务--_事务的提交方式

```mysql
/*
	事务的提交方式
		查询事务提交方式：SELECT @@AUTOCOMMIT;  -- 1代表自动提交    0代表手动提交
		修改事务提交方式：SET @@AUTOCOMMIT=数字;
*/
-- 查询事务的提交方式
SELECT @@autocommit;

UPDATE account SET money=1000 WHERE NAME='张三';

COMMIT;


-- 修改事务的提交方式
SET @@autocommit=0;

```

##### 05. 事务的四大特性

- 原子性(atomicity)

  > 原子性是指事务包含的所有操作要么全部成功，要么全部失败回滚  因此事务的操作如果成功就必须要完全应用到数据库，如果操作失败则不能对数据库有任何影响

-  一致性(consistency)

  >  一致性是指事务必须使数据库从一个一致性状态变换到另一个一致性状态，也就是说一个事务执行之前和执行之后都必须处于一致性状态
  >
  > 白话讲： 就是数据要按照我们预期的样子去发生变化

- 隔离性(isolcation)

  > 隔离性是当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务 , 不能被其他事务的操作所干扰，多个并发事务之间要相互隔离

-  持久性(durability)

  > 持久性是指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的， 即便是在数据库系统遇到故障的情况下也不会丢失提交事务的操作



### 4. 事务隔离级别

##### 01. 不同隔离级别带来的问题如下: 

|         事务隔离级别         | 脏读 | 不可重复读 | 幻读 |
| :--------------------------: | :--: | :--------: | :--: |
| 读未提交（read-uncommitted） |  是  |     是     |  是  |
| 不可重复读（read-committed） |  否  |     是     |  是  |
| 可重复读（repeatable-read）  |  否  |     否     |  是  |
|    串行化（serializable）    |  否  |     否     |  否  |

**脏读:**

-  事物A读到了事物B未提交的数据，如果事物B回滚了，对A事物来说查到的数据就是不正确的，这种现象我们称之为：“脏读”。
-  解决：RU隔离级别存在脏读问题，修改隔离级别为: RC 或以上。

**不可重复读**:  

- 事物A读到了事物B修改后提交的数据，导致事物A两次执行一模一样的查询SQL得到的数据是不一样的，这种现象称之为： “不可重复读”。
- 解决：RU隔离级别存在不可重复读问题, 修改隔离级别为：RR 或 以上。

**幻读**

- 事物A查询 用户赵六, 没有查到，想要新增用户赵六时，mysql提示错误：“该用户已存在”，执行删除赵六操作时，mysql提示成功了。导致原因：“事物B在事物A插入之前，提前将赵六插入到了表中，进而导致事物A出现了幻觉”，这种现象称之为：“幻读”。

- 解决：RR这种mysql默认的隔离界别只存在幻读问题，一般我们业务系统可以无视此影响，当然也可以修改隔离级别为：serializable。强烈不推荐修改隔离界别，因为:serializable 存在锁表的操作，会十分的影响性能，导致数据库的并发不高。

  

##### 02_事务_--查询和修改事务的隔离级别

```mysql
/*
	事务的隔离级别
		查询隔离级别：SELECT @@TX_ISOLATION;
		修改隔离级别：SET GLOBAL TRANSACTION ISOLATION LEVEL 级别字符串;
*/
-- 查询事务隔离级别
SELECT @@tx_isolation;

-- 修改事务隔离级别
SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

##### 03_事务_--脏读的问题演示和解决_窗口1

```mysql
/*
	脏读的问题演示和解决
	脏读：一个事务中读取到了其他事务未提交的数据
*/
-- 设置事务隔离级别为read uncommitted
SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- 开启事务
START TRANSACTION;

-- 转账
UPDATE account SET money = money-500 WHERE NAME='张三';
UPDATE account SET money = money+500 WHERE NAME='李四';

-- 查询account表
SELECT * FROM account;

-- 回滚
ROLLBACK;
```

##### 04_事务_--脏读的问题演示和解决_窗口2

```mysql
-- 查询事务隔离级别
SELECT @@tx_isolation;

-- 开启事务
START TRANSACTION;

-- 查询account表
SELECT * FROM account;
```

##### 05_事务_--不可重复读的问题演示和解决_窗口1

```mysql
/*
	不可重复读的问题演示和解决
	不可重复读：一个事务中读取到了其他事务已提交的数据
*/
-- 设置事务隔离级别为read committed
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;


-- 开启事务
START TRANSACTION;

-- 转账
UPDATE account SET money = money-500 WHERE NAME='张三';
UPDATE account SET money = money+500 WHERE NAME='李四';

-- 查询account表
SELECT * FROM account;

-- 提交事务
COMMIT;
```

##### 06_事务_--不可重复读的问题演示和解决_窗口2

```mysql
-- 查询隔离级别
SELECT @@tx_isolation;

-- 开启事务
START TRANSACTION;

-- 查询account表
SELECT * FROM account;

-- 提交事务
COMMIT;
```

##### 07_事务_--幻读的问题演示和解决_窗口1

```mysql
/*
	幻读的问题演示和解决
	幻读：
	查询某记录是否存在，不存在
	准备插入此记录，但执行插入时发现此记录已存在，无法插入
	或某记录不存在执行删除，却发现删除成功
*/
-- 设置隔离级别为repeatable read
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET GLOBAL TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- 开启事务
START TRANSACTION;

-- 添加记录
INSERT INTO account VALUES (4,'赵六',2000);

-- 查询account表
SELECT * FROM account;

-- 提交事务
COMMIT;
```

##### 08_事务_--幻读的问题演示和解决_窗口2

```mysql
-- 查询隔离级别
SELECT @@tx_isolation;

-- 开启事务
START TRANSACTION;

-- 查询account表
SELECT * FROM account;

-- 添加
INSERT INTO account VALUES (3,'王五',2000);
```



##### 09. (扩展)-如何解决Mysql 默认的Repeatable read隔离级别产生的幻读问题

- RR 级别下存在幻读的可能，但很多场景下我们的业务sql并不会存在幻读的风险。

- SERIALIZABLE 的一刀切虽然事务绝对安全，但性能会有很多不必要的损失。不推荐使用！

- 我们可以在查询时给改行数据加上 排他锁（写锁）, 因为该锁是悲观锁，加锁后，其他事物将不可对查询数据进行任何操作，所以也就不存在其他事物insert新数据导致当前事物出现幻觉，事务安全与性能兼备，这也是 RR 作为mysql默认隔是个事务离级别的原因，所以需要正确的理解幻读。

- 错误理解幻读:

  >  说幻读是 事务A 执行两次 select 操作得到不同的数据集，即 select 1 得到 10 条记录，select 2 得到 11 条记录。这其实并不是幻读，这是不可重复读的一种，只会在 R-U R-C 级别下出现，而在 mysql 默认的 RR 隔离级别是不会出现的。 

- 正确理解幻读:

  >  幻读，并不是说两次读取获取的结果集不同，幻读侧重的方面是某一次的 select 操作得到的结果所表征的数据状态无法支撑后续的业务操作。更为具体一些：select 某记录是否存在，不存在，准备插入此记录，但执行 insert 时发现此记录已存在，无法插入，此时就发生了幻读。 

**通过排他锁解决幻读--示例代码:** 

RR级别下只要对 SELECT 操作也手动加行（X）锁即可类似 SERIALIZABLE 级别（它会对 SELECT 隐式加锁），即大家熟知的：

```mysql
#使用悲观锁, 这里需要用悲观锁中的 X锁， 不能用 LOCK IN SHARE MODE, 因为拿到 S锁 后我们没办法做写操作
SELECT `id` FROM `users` FOR UPDATE;
```