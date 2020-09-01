# Mac下MySQL允许用户远程访问数据库

默认Mac安装完MySQL是不允许远程访问的，一般我们都需要远程访问，所以需要进行配置。

## 步骤

### 登录MySQL

用root账户登录，回车输入密码进行登录

```
mysql -uroot -p
```

### 登录后，选择数据库

```
use mysql;
```

### 更新域属性，'%'表示允许外部访问

```
update user set host='%' where user ='root';
```

### 执行以上语句之后再执行(刷新配置)

```
FLUSH PRIVILEGES;
```

### 再执行授权语句

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION;
```

配置完，再测试远程连接，就发现可以连接上了