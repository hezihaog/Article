# Linux下Tomcat启动正常，但浏览器无法访问8080

## 环境

- Centos7
- Tomcat7
- JDK8

- 查看Java是否配置环境变量

```
echo $JAVA_HOME
```

环境变量和tomcat都启动成功，按浏览器访问8080端口，访问不了

其实是centos的防火墙导致我们访问不了8080端口。

## 配置防火墙

有2种方式，要么关闭防火墙，要么让防火墙开放8080端口

- 方式一：关闭防火墙

```
查看防火墙状态： systemctl status firewalld 
启动防火墙： systemctl start firewalld 
停止防火墙： systemctl stop firewalld 
永久停用： systemctl disable firewalld 
启用防火墙： systemctl enable firewalld
```

- 方式二：让防火墙开放8080端口

```
//查看防火墙是否开启，如果没有开启，先用上面开启防火墙的命令开启了先
firewall-cmd --permanent --zone=public --add-port=8080/tcp
//开放8080端口
firewall-cmd --zone=public --query-port=8080/tcp 
//重新加载防火墙
firewall-cmd --reload 
```

最后重启Tomcat，即可

如果还是不行，查看一下8080端口是否被占用！

## 端口占用

用以下命令查询8080端口

```
netstat –apn | grep 8080
```

如果有，则修改占用方使用的端口，或者让Tomcat用别的端口。

- 修改tomcat端口
    1. 进入到tomcat的安装目录
    2. 进入conf文件夹，vim 打开server.xml
    3. 将8080改为8090或其他端口