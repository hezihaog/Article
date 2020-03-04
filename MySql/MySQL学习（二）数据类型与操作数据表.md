#### MySQL学习（二）数据类型与操作数据表

有人说，做安卓开发久了，数据库SQL和前端知识比较薄弱（也有例外哈，不绝对），我就对数据库SQL这方面知识比较薄弱。本篇是根据**慕课网**[与MySQL的零距离接触](https://www.imooc.com/learn/122)学习而写的笔记，感谢老师录制的视频，讲得很清楚。

## MySQL中的数据类型

#### 整形

数据类型| 字节 |
:-: | :-: |
TINYINT| 1 |
SMALLINT| 2 |
MEDIUMINT| 3 |
INT| 4 |
BIGINT| 8 |

#### 浮点型

数据类型| 解释 |
:-: | :-: |
FLOAT[(M,D)]| 单精度浮点数，M是数字总位数，D是小数点后的位数，根据硬件允许的限制来保存值，单精度浮点数精确到大于7位小数位 |
DOUBLE[(M,D)]|双精度浮点数|

#### 日期时间型

数据类型| 存储字节 | 解释 |
:-: | :-: | :-: |
YEAR| 1 | 4位数年份：例如1996 |
TIME| 3 | 时间：00:00:00~23:59:59 |
DATE| 3 | 年月日，不包括时间，1000-1-1~9999-12-31 |
DATATIME| 8 | 1000-1-1零点~9999-12-31二十三点59-59 |
TIMESTAMP| 4 |时间戳,1970-1-1零点~2037 |

#### 字符型

数据类型| 解释 |
:-: | :-: |
CHAR(M) | 定长，不足时自动补空格，M个字节，0 <= M <= 255 |
VARCHAR(M) | 不定长 |
TINYTEXT|  |
TEXT|  |
MEDIUMTEXT|  |
LONGTEXT|  |
ENUM| 1或2个字节，取决于枚举值	的个数（最多65,535个值） |
SET| 1,2,3,4或8个字节，取决于set成员的数量（最多64个成员） |

## 数据表操作

使用数据表，必须先打开某个数据库才能进行操作

- USE 打开数据库

```
//1、先看看有哪些数据库
SHOW DATABASES;
//输出
+--------------------+
| Database           |
+--------------------+
| information_schema |
| exercises          |
| mysql              |
| performance_schema |
| sys                |
| t2                 |
| test               |

//2、打开t2数据库
USE t2;
//输出
Database changed

//3、检验，显示当前选择的数据库名
SELECT DATABASE();
+------------+
| DATABASE() |
+------------+
| t2         |
+------------+
```

#### 创建数据表

- 格式

```
CREATE TABLE [IF NOT EXISTS] table_name {
	colume_name data_type,
	...
}
```

- 新建用户表

```
CREATE TABLE tb1( 
	username VARCHAR(20), 
	age TINYINT UNSIGNED, 
	salary FLOAT(8,2) UNSIGNED
);
```

#### 查看数据表

- 格式

```
SHOW TABLES [FROM db_name] [LIKE `pattern` | WHERE expr]
```

- 显示当前数据库中所有的数据表

```
SHOW TABLES;
//输出
+--------------+
| Tables_in_t2 |
+--------------+
| tb1          |
+--------------+
```

- 使用可选参数，查看另外的数据库中的所有表

```
//FROM关键字后跟查询的数据库名，例如我们查询MySQL自带的mysql数据库。
SHOW TABLES FROM mysql;
//输出mysql数据库中的24张表
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| engine_cost               |
| event                     |
| func                      |
| general_log               |
| gtid_executed             |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| innodb_index_stats        |
| innodb_table_stats        |
| ndb_binlog_index          |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| server_cost               |
| servers                   |
| slave_master_info         |
| slave_relay_log_info      |
| slave_worker_info         |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
```

#### 查看数据表结构

- 格式

```
SHOW COLUMNS FROM tab_name
```

- 查询tb1表的结构

```
SHOW COLUMNS FROM tb1;
//输出
+----------+---------------------+------+-----+---------+-------+
| Field    | Type                | Null | Key | Default | Extra |
+----------+---------------------+------+-----+---------+-------+
| username | varchar(20)         | YES  |     | NULL    |       |
| age      | tinyint(3) unsigned | YES  |     | NULL    |       |
| salary   | float(8,2) unsigned | YES  |     | NULL    |       |
+----------+---------------------+------+-----+---------+-------+
```

#### 记录的插入与查找

- 插入

	- 格式，字段可以省略，但是省略就必须全部字段都有值，如果想局部字段有值，那么就需要写明哪些字段需要插入。

	```
	INSERT [INTO] tab_name [(col_name)] VALUES(val, ...)
	```
	
	- 插入一条数据（全字段）
	
	```
	//插入用户，用户名Tom，年龄25，工资7863.5
	INSERT tb1 VALUES('Tom', 25, 7863.5);
	```
	
	- 插入一条数据（局部字段）

	```
	//局部字段插入，用户名john，工资4500.69
	//年龄没有插入，所以为NULL
	INSERT tb1(username, salary) VALUES('John', 4500.69);
	```

- 查找（最简单的，复杂的后面再讲）

	- 格式

	```
	SELECT expre,... FROM tb_name
	```
	
	- 查询指定表的所有记录

	```
	//*表示所有字段
	SELECT * FROM tb1;
	//输出
	+----------+------+---------+
	| username | age  | salary  |
	+----------+------+---------+
	| Tom      |   25 | 7863.50 |
	| John     | NULL | 4500.69 |
	+----------+------+---------+
	```

#### MySQL空值与非空

- NULL，字段可以为空
- NOT NULL，字段不可以为空

- 演示

	1. 新建一张表
	
	```
	CREATE TABLE tb2(
	username VARCHAR(20) NOT NULL, 
	age TINYINT UNSIGNED NULL
	);
	```
	
	2. 查询数据表结构

	```
	SHOW COLUMNS FROM tb2;
	//输出
	+----------+---------------------+------+-----+---------+-------+
	| Field    | Type                | Null | Key | Default | Extra |
	+----------+---------------------+------+-----+---------+-------+
	| username | varchar(20)         | NO   |     | NULL    |       |
	| age      | tinyint(3) unsigned | YES  |     | NULL    |       |
	+----------+---------------------+------+-----+---------+-------+
	```
	
	3. （测试可空）故意age值设置为NULL

	```
	INSERT tb2 VALUES('Wally', NULL);
	//输出，插入成功
	Query OK, 1 row affected (0.01 sec)
	//查询所有字段
	SELECT * FROM tb2;
	//输出
	+----------+------+
	| username | age  |
	+----------+------+
	| Wally    | NULL |
	+----------+------+
	```
	
	4. （测试非空）故意给username赋值为NULL

	```
	//username为null
	INSERT tb2 VALUES(NULL, 26);
	//报错输出，提示username不能为null
	ERROR 1048 (23000): Column 'username' cannot be null
	```

#### MySQL自动编号和主键约束

- 自动编号

	1. 自动编号，且必须与主键组合使用
	2. 默认情况下，起始值为1，每次的增量1

- 主键约束

	1. 每张数据表只能存在一个主键
	2. 主键保证记录的唯一性
	3. 主键自动为NOT NULL
	4. AUTO_INCREMENT必须和主键一起使用，但是主键不一定要和AUTO_INCREMENT一起使用

- 示例1（主键和自动编码）

	1. 创建一张表，声明id主键和自动编码

	```
	CREATE TABLE tb3(
    id SMALLINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(30) NOT NULL
    );
	```
	
	2. 查看数据表结构（可见id为主键，并且自动设置为非空）

	```
	SHOW COLUMNS FROM tb3;
	+----------+----------------------+------+-----+---------+----------------+
	| Field    | Type                 | Null | Key | Default | Extra          |
	+----------+----------------------+------+-----+---------+----------------+
	| id       | smallint(5) unsigned | NO   | PRI | NULL    | auto_increment |
	| username | varchar(30)          | NO   |     | NULL    |                |
	+----------+----------------------+------+-----+---------+----------------+
	```
	
	3. 插入4条记录（id主键不用插入值，会自动编号并插入）

	```
	INSERT tb3(username) VALUES('Tom');
	INSERT tb3(username) VALUES('John');
	INSERT tb3(username) VALUES('Rose');
	INSERT tb3(username) VALUES('Dimitar');
	```
	
	4. 查看记录

	```
	SELECT * FROM tb3;
	//输出
	+----+----------+
	| id | username |
	+----+----------+
	|  1 | Tom      |
	|  2 | John     |
	|  3 | Rose     |
	|  4 | Dimitar  |
	+----+----------+
	```
	
- 示例2（只有主键），主键不能重复

	1. 不设置自动编号

	```
	CREATE TABLE tb4(
    id SMALLINT UNSIGNED PRIMARY KEY,
    username VARCHAR(20) NOT NULL
    );
	```
	
	2. 插入2条数据，需要手动指定id

	```
	INSERT tb4 VALUE(4, 'Tom');
	INSERT tb4 VALUE(22, 'John');
	```
	
	3. 查看表记录

	```
	SELECT * FROM tb4;
	//输出
	+----+----------+
	| id | username |
	+----+----------+
	|  4 | Tom      |
	| 22 | John     |
	+----+----------+
	```
	
	4. 插入重复记录

	```
	INSERT tb4 VALUE(22, 'Rose');
	//输出报错信息，插入了重复主键
	ERROR 1062 (23000): Duplicate entry '22' for key 'PRIMARY'
	```

#### 唯一约束

1. 唯一约束可以保证记录的唯一性
2. 唯一约束的字段可以为NULL值
3. 每张数据表可以存在多个唯一约束

- 实例

	1. 创建一张新表（主键+自动编号，另外一个字段设置唯一约束）

	```
	CREATE TABLE tb5(
    id SMALLINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(20) NOT NULL UNIQUE KEY, 	 age TINYINTUNSIGNED
    );
	```
	
	 2. 查看数据表结构（用户名唯一）

    ```
    SHOW COLUMNS FROM tb5;
    //输出
	+----------+----------------------+------+-----+---------+----------------+
	| Field    | Type                 | Null | Key | Default | Extra          |
	+----------+----------------------+------+-----+---------+----------------+
	| id       | smallint(5) unsigned | NO   | PRI | NULL    | auto_increment |
	| username | varchar(20)          | NO   | UNI | NULL    |                |
	| age      | tinyint(3) unsigned  | YES  |     | NULL    |                |
	+----------+----------------------+------+-----+---------+----------------+
    ```
    
    3. 插入数据

    ```
    INSERT tb5(username, age) VALUES('Tom', 22);
    //输出，插入成功
    Query OK, 1 row affected (0.00 sec)
    //再次插入
    INSERT tb5(username, age) VALUES('Tom', 22);
    //输出报错信息
    ERROR 1062 (23000): Duplicate entry 'Tom' for key 'username'
    ```

#### 默认约束

- 默认值，当插入记录时，如果没有明确字段赋值，则自动赋予默认值

	1. 创建一张数据表，性别默认为3，为保密性别

	```
	CREATE TABLE tb6(
    id SMALLINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(20) NOT NULL UNIQUE KEY, 
    sex ENUM('1', '2', '3') DEFAULT '3'
    );
	```
	
	2. 查看数据表结构（sex字段的Default属性有了默认值）

	```
	SHOW COLUMNS FROM tb6;
	//输出
	+----------+----------------------+------+-----+---------+----------------+
	| Field    | Type                 | Null | Key | Default | Extra          |
	+----------+----------------------+------+-----+---------+----------------+
	| id       | smallint(5) unsigned | NO   | PRI | NULL    | auto_increment |
	| username | varchar(20)          | NO   | UNI | NULL    |                |
	| sex      | enum('1','2','3')    | YES  |     | 3       |                |
	+----------+----------------------+------+-----+---------+----------------+
	```
	
	3. 插入一条数据（只插入username字段）

	```
	INSERT tb6(username) VALUES('Tom');
	//输出
	Query OK, 1 row affected (0.01 sec)
	```
	
	4. 查询数据表的所有数据（发现记录中的sex字段我们没有赋值，但被自动赋值为默认值3了）

	```
	SELECT * FROM tb6;
	//输出
	+----+----------+------+
	| id | username | sex  |
	+----+----------+------+
	|  1 | Tom      | 3    |
	+----+----------+------+
	```