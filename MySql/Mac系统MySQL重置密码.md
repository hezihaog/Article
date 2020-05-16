#### Mac系统MySQL重置密码

Mac下，每次安装MySQL，root账号的密码都是随机的，一般我们都会去重置密码，本篇记录一下，重置密码的步骤。

#### 步骤一：停止MySQL服务

```
sudo /usr/local/mysql/support-files/mysql.server stop
```

#### 步骤二：禁止MySQL验证功能

- 进入MySQL目录，MySQL的目录每个人安装完后，可能不一样，具体看个人的安装路径来做修改，例如我安装完，安装目录就是mysql-5.7.16-osx10.11-x86_64

```
cd /usr/local/mysql/bin/
```

- 获取管理员权限

```
sudo su
```

- （重点）禁止MySQL的验证功能，回车后，会重启MySQL服务，这时不要关闭终端，在MySQL目录下新开一个终端

```
mysqld_safe --skip-grant-tables
```

#### 步骤三：重置密码

- 查询MySQL版本

```
//查询版本，可使用这个命令，注意最后一个字符V是大写
mysql -V
//会输出以下信息，我的版本是5.7.16
mysql  Ver 14.14 Distrib 5.7.16, for osx10.11 (x86_64) using  EditLine wrapper
```

- 进入MySQL命令操作

```
mysql
```

- 刷新权限

```
FLUSH PRIVILEGES;
```

- （重点）修改密码，注意有版本区别！

```
//8.+版本，以下使用这个
set password for 'root'@'localhost' = password('新密码');
//以上使用这个
alter user 'root'@'localhost' identified by '新密码';
```

- 修改完毕，退出

```
exit
```

- 用新密码重新登录即可

```
mysql -uroot -p新密码
```