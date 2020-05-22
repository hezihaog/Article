#### MySQL学习（一）初涉数据库

有人说，做安卓开发久了，数据库SQL和前端知识比较薄弱（也有例外哈，不绝对），我就对数据库SQL这方面知识比较薄弱。本篇是根据**慕课网**[与MySQL的零距离接触](https://www.imooc.com/learn/122)学习而写的笔记，感谢老师录制的视频，讲得很清楚。

## MySQL服务的启动和停止

- MacOS，这里只总结MacOS下的命令，Windows和Linux请自行搜索。

#### 启动服务

```
sudo /usr/local/mysql/support-files/mysql.server start
```

#### 停止服务

```
sudo /usr/local/mysql/support-files/mysql.server stop
```

#### 重启服务

```
sudo /usr/local/mysql/support-files/mysql.server restart
```

## 使用MySQL

#### 查看版本

```
//注意最后的V是大写
mysql -V
```

#### 登录MySQL

- 登录命令的基本格式

```
mysql -u用户名 -p密码
//例如
mysql -uzihe -pzihe123
```

- 不想将密码显示到命令行，则-p后不写密码，直接回车，后续会继续让你输入密码

```
mysql -u用户名 -p
//例如
mysql -uzihe -p
//后续输入...
zihe123
```

- 端口号-P参数（大写P），MySQL默认端口号为3306，如果修改过，则需要填写，不写则默认为请求3306

```
mysql -uzihe -pzihe123 -P3307
```

- 连接地址-h参数，如果想连接的MySQL是远程的，则需要使用-h参数。不写默认连接本地MySQL数据库，本地为127.0.0.1（本地回传地址）。

```
//连接本地数据库地址，远程则修改为对应的地址即可
mysql -uzihe -pzihe123 -h127.0.0.1
```

#### 退出MySQL

- 方式一：mysql > exit;

```
//输入
exit;
//输出
Bye
```

- 方式二：mysql > quit;

```
//输入
quit;
//输出
Bye
```

- 方式三：mysql > \q

```
//输入
\q;
//输出
Bye
```

#### 修改MySQL提示符

MySQL提示符，指的是什么呢？就是登陆MySQL后，命令行左边的默认提示文字，例如默认就为mysql > ，那么如果我们想将提示符修改为有意义的文字，我们可以使用命令修改它。

- 未登录状态下

	- prompt参数，登陆命令后追加--prompt参数，--prompt后填写你所想修改为的提示符
	
	```
	mysql -uzihe -pzihe123 --prompt zihe > 
	//修改后显示：
	zihe > 
	```
	
- 已登录状态下

	- prompt命令 你所想修改为的提示符

	```
	zihe > prompt mysql > 
	//返回提示
	PROMPT set to 'mysql > '
	//修改后显示
	mysql > 
	```
	
- 常用提示符占位符（相当于默认变量）

参数 |描述 |
:-: | :-: |
\D | 完整日期 |
\d | 当前数据库 |
\h | 服务器名称 |
\u | 当前用户 |

```
//例如我们修改为：用户名@服务器名称\当前打开的数据库
prompt \u@\h \d > 
//修改后
root@localhost (none) > 
//none是因为我们没有打开数据库，如果我们打开test数据库，则显示
root@localhost test > 
```

## MySQL常用命令

- 显示当前版本号

```
SELECT VERSION();
//输出
+-----------+
| version() |
+-----------+
| 5.7.19    |
+-----------+
```

- 显示当前日期

```
SELECT NOW();
//输出
+---------------------+
| NOW()               |
+---------------------+
| 2019-05-02 10:43:51 |
+---------------------+
```

- 显示当前用户

```
SELECT USER();
+----------------+
| USER()         |
+----------------+
| root@localhost |
+----------------+
```

## MySQL语句规范

- 关键字和函数名称全部大写
- 数据库名称、表名称、字段名称全部小写
- SQL语句必须以分号结尾

#### 一问一答

- 问：为什么关键字和函数名称要全部大写
	- 答：因为要将关键字和函数名和库名、表名、字段名做区分，所以规范规定全部大写（如果不准守也是可以识别的）。

- 问：SQL语句如果不加分号会怎么样？
	- 答：不加回车，MySQL会一直等待输入结束;号才执行。

	```
	mysql> SELECT USER()
    -> 
    -> 
    -> ;
    +----------------+
	| USER()         |
	+----------------+
	| root@localhost |
	+----------------+
	```
	
## 操作数据库

#### 创建数据库

- 语法

{}花括号内的参数为必须项，[]中括号代表可选项，|竖线表示或者的意思。

```
CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name [DEFAULT] CHARACTER SET [=] charset_name
```

- 创建一个数据库名为t1的数据库

```
CREATE DATABASE t1;
//输出Query OK，代表查询成功或者执行命令成功
Query OK, 1 row affected (0.01 sec)
```

- 如果创建的表已存在，执行创建操作语句会报错

```
CREATE DATABASE t1;
//输出
ERROR 1007 (HY000): Can't create database 't1'; database exists
```

- 如果想创建时判断如果存在则不创建，不存在则创建，则添加IF NOT EXISTS参数即可。（不会报错，而是显示警告Warning）

```
CREATE DATABASE IF NOT EXISTS t1;
//输出
Query OK, 1 row affected, 1 warning (0.00 sec)
```

- 查询警告信息

```
SHOW WARNINGS;
//输出
+-------+------+---------------------------------------------+
| Level | Code | Message                                     |
+-------+------+---------------------------------------------+
| Note  | 1007 | Can't create database 't1'; database exists |
+-------+------+---------------------------------------------+
1 row in set (0.00 sec)
```

- 创建时指定数据库的默认编码

使用CHARACTER SET参数指定编码方式。CHARACTER SET前面的DEFAULT可以不写。

```
CREATE DATABASE IF NOT EXISTS t2 CHARACTER SET gbk;
//再次查询数据库创建语句，进行确认
SHOW CREATE DATABASE t2;
//输出
+----------+------------------------------------------------------------+
| Database | Create Database                                            |
+----------+------------------------------------------------------------+
| t2       | CREATE DATABASE `t2` /*!40100 DEFAULT CHARACTER SET gbk */ |
+----------+------------------------------------------------------------+
1 row in set (0.00 sec)
```

#### 查看数据库所有的表

- 语法

```
SHOW {DATABASES | SCHEMAS} [LIKE 'pattern' | WHERE expr]
```

```
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
| t1                 |
| test               |
+--------------------+
```

#### 查看创建数据库时使用的语句

```
SHOW CREATE DATABASE t1;

//输出
+----------+-------------------------------------------------------------+
| Database | Create Database                                             |
+----------+-------------------------------------------------------------+
| t1       | CREATE DATABASE `t1` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+-------------------------------------------------------------+
1 row in set (0.00 sec)
```

#### 修改数据库

- 修改数据库的编码方式

语法格式：

```
ALERT {DATABASE | SCHEMA} [db_name] [DEFAULT] CHARACTER SET [=] charset_name
```

修改数据库的编码为utf8

```
ALERT DATABASE t2 CHARACTER SET = utf8;
//重新查看创建数据库的语句
SHOW CREATE DATABASE t2;
//输出
+----------+------------------------------------------------------------+
| Database | Create Database                                            |
+----------+------------------------------------------------------------+
| t2       | CREATE DATABASE `t2` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+------------------------------------------------------------+
1 row in set (0.00 sec)
```

#### 删除数据库

- 语法

```
DROP {DATABASE | SCHEMA} [IF EXISTS] db_name
```

- 删除指定数据库

```
DROP DATABASE t1;
//输出
Query OK, 0 rows affected (0.01 sec)
//重新查看所有数据库
+--------------------+
| Database           |
+--------------------+
| information_schema |
| exercises          |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
```

- 删除不存在的数据库

如果我们删除一个不存在的数据库，会报错

```
DROP DATABASE t1;
//输出，提示我们要删除的数据库不存在
ERROR 1008 (HY000): Can't drop database 't1'; database doesn't exist
```

- 要想删除前先判断是否存在，我们就需要加上IF EXISTS参数。如果删除的数据库不存在，不会报错，而是得到一个警告。

```
DROP DATABASE IF EXISTS t1;
//输出
Query OK, 0 rows affected, 1 warning (0.00 sec)
```