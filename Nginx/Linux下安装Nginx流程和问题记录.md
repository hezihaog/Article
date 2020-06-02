#### Linux下安装Nginx流程和问题记录

今天在Linux虚拟机上安装Nginx，用来做反向代理，安装过程中遇到一些问题，顺便记录一下。

#### 终端远程连接到Linux

我们需要在Linux下，对安装包进行解压，所以我们可以使用终端，远程登录，进行命令解压。

![2.终端新建远程连接.png](https://user-gold-cdn.xitu.io/2020/6/2/172758269d52c0e4?w=929&h=655&f=png&s=386296)

![3.终端远程连接Linux.png](https://user-gold-cdn.xitu.io/2020/6/2/17275829179c0338?w=424&h=423&f=png&s=95345)

![3.终端远程连接Linux2.png](https://user-gold-cdn.xitu.io/2020/6/2/1727582d33e3e649?w=422&h=422&f=png&s=26379)

- 检查Nginx是否未安装，我们可以通过以下命令查看。

正常会有2个进程，master进程和worker进程。如果没有就是未安装，缺少一个，也是安装得有问题。

master进程，监听80端口。worker进程，负责转发请求到配置的端口服务。

```
ps -ef | grep nginx
//输出
[root@center6 ~]# ps -ef | grep nginx
nobody    2854  6521  0 04:01 ?        00:00:04 nginx: worker process
root      3848  3827  0 07:59 pts/4    00:00:00 grep nginx
root      6521     1  0 02:41 ?        00:00:00 nginx: master process nginx
```

- 新建一个用户，将安装包放到用户目录下，例如我新建一个leyou用户，leyou的用户目录就是/home/leyou

```
useradd leyou
```

#### 将Nginx安装包上传到Linux上

Linux系统，我是安装在虚拟机上的，是CenterOS6。需要准备好Nginx安装包，我使用的是nginx-1.10.0.tar.gz。

- 新建FTP连接，注意默认选择的协议是FTP连接，我们要使用SFTP，才能连接上，输入登录的账户和密码，连接即可。

1.SFTP连接Linux.png
![](https://user-gold-cdn.xitu.io/2020/6/2/172758206cd6bd76?w=812&h=463&f=png&s=68314)

- 使用FileZilla软件，将Nginx安装包上传到对应的Linux上，使用SFTP远程登录到Linxus系统上。将文件拖拽到右边到右边的Linux上即可。

![2.上传Nginx安装包.png](https://user-gold-cdn.xitu.io/2020/6/2/1727584e35e45ba8?w=1200&h=737&f=png&s=167940)

- 使用tar命令解压安装包**（重要）**

```
tar xvf nginx-1.10.0.tar.gz
```

- 解压完毕，将压缩包删除

```
rm-rf nginx-1.10.0.tar.gz
```

- 进入nginx解压目录，进行编译、安装

```
cd nginx-1.10.0
```

- 安装前，先要配置一个nginx的安装目录**（重要）**，第一个--prefix指定的事文件存放目录，放在了opt/nginx下，第二个是脚本文件存放目录，在`/usr/bin/nginx`下

```
./configure --prefix=/opt/nginx --sbin-path=/usr/bin/nginx
```

- 配置完毕，进行编译和安装**（重要）**

```
make && make install
```

#### 报错：./configure: error: C compiler cc is not found

执行上面的`./configure --prefix=/opt/nginx --sbin-path=/usr/bin/nginx`命令，可能会执行出错，抛出./configure: error: C compiler cc is not found，意思是没有安装c的编译环境，就是gcc，我们需要安装它，需要执行以下命令，其实是安装4个东西，分别是gcc和它的3个依赖

1. gcc，功能：预处理、编译、连接、汇编
1. openssl，功能：用于网站加密通讯
2. pcre，功能：用于支持解析正则表达式
3. zlib，功能：用于对数据进行解压缩。网站之间通信时，数据先压缩再传输，通过消耗CPU的方式来节省网络带宽

我们可以分别安装，也可以合并为一条命令，直接全部安装

```
yum -y install gcc gcc-c++ autoconf automake make
```

如果你其实已经安装了gcc，但又依旧出现这个错误，可以尝试卸载再重新安装

```
# yum remove gcc -y
# yum install gcc -y 
```

安装好gcc的依赖后，需要重新执行一遍`./configure --prefix=/opt/nginx --sbin-path=/usr/bin/nginx`命令，否则可能会抛出下面这个错误：`make: *** 没有规则可以创建“default”需要的目标“build”`。

#### 报错：make: *** 没有规则可以创建“default”需要的目标“build”

如果安装gcc和依赖后，没有重新执行一遍`./configure --prefix=/opt/nginx --sbin-path=/usr/bin/nginx`命令，就执行`make && make install`，就会抛出`make: *** 没有规则可以创建“default”需要的目标“build”`的错误。操作完后，再make就好了。

#### 启动、停止和配置Nginx

- 启动Nginx

```
nginx
```

- 停止Nginx

```
nginx -s stop
```

- 重新加载Nginx配置文件

```
nginx -s reload
```

--

- 如果启动不成功，可能是防火墙没有关闭，使用以下命令进行关闭，有临时关闭和永久关闭，重新即可生效

```
//临时关闭
service iptables stop
//永久关闭
chkconfig iptables off
```

- 配置文件，在opt文件夹下，完整路径为：/opt/nginx/conf，找到nginx.conf文件，右边编辑，例如我添加2个server，就是域名转发配置，分别将2个域名转发到本地的不同端口的微服务

```

#user  nobody;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
   
    keepalive_timeout  65;

    gzip  on;
	server {
        listen       80;
        server_name  manage.leyou.com;

        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        location / {
			proxy_pass http://127.0.0.1:9001;
			proxy_connect_timeout 600;
			proxy_read_timeout 600;
        }
    }
	server {
        listen       80;
        server_name  api.leyou.com;

        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        location / {
			proxy_pass http://127.0.0.1:10010;
			proxy_connect_timeout 600;
			proxy_read_timeout 600;
        }
    }
}
```

- 最后让Nginx重新加载配置文件，即可生效配置

```
nginx -s reload
```