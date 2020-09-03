# mysql报错：Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggre

## 出错原因

MySQL 5.7.5 及以上功能依赖检测功能，而我使用的mysql是8.0版本。如果启用了ONLY_FULL_GROUP_BY SQL 模式（默认情况下），MySQL将拒绝选择列表，HAVING 条件或 ORDER BY 列表的查询引用在 GROUP BY 子句中既未命名的非集合列，也不在功能上依赖于它们。

## 解决方法

### 临时生效

本方法设置后，重启失效！

#### 登录MySQL，查询模式值

```
//登录
mysql -uroot -p
//查询
select @@global.sql_mode

//输出
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

#### 去掉ONLY_FULL_GROUP_BY，重新设置值

```
set @@global.sql_mode
=`STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION`;

//或者这样
SET GLOBAL sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
```

### 永久生效

### 查看mysql配置文件位置

```
[root@localhost ~]# ps -ef | grep mysql
mysql      838     1  0 02:21 ?        00:00:00 /usr/sbin/mysqld --defaults-file=/etc/my.cnf
root      2035  1706  0 02:29 pts/0    00:00:00 grep --color=auto mysql
```

### 打开配置文件

```
vim /etc/my.cnf
```

文件内容中，添加下面2行

```
[mysqld]
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

:wq，保存并写入

```
:wq
```

### 重启MySQL

如果报命令不存在，去设置里面重启MySQL

```
systemctl restart mysql
```