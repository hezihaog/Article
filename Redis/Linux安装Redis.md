# Linux安装Redis

## 下载Redis压缩包

```
wget http://download.redis.io/releases/redis-3.0.0.tar.gz
```

## 将压缩包拷贝到/usr/local下

```
cp redis-3.0.0.tar.gz /usr/local
```

## 解压压缩包

```
tar -zxvf redis-3.0.0.tar.gz
```

## 进入解压目录`/usr/local/redis`，进行编译并安装

```
//进入解压目录
cd /usr/local/redis-3.0.0
//编译并安装
make PREFIX=/usr/local/redis install
```

## 成功后，Redis的安装目录为`/usr/local/redis`，开始配置Redis

Redis的解压目录下，有一个redis.conf文件，这个文件就是Redis的配置文件，我们拷贝它到Redis安装目录下`/usr/local/redis/bin`

```
//切换到安装目录下
cd /usr/local/redis
//拷贝配置文件
cp /usr/local/redis-3.0.0/redis.conf /usr/local/redis/bin
```

## 在安装目录下的`bin`目录，存放着Redis可运行命令

```

//进入
cd /usr/local/redis/bin
//目录下，进行ll，可以看到如下可运行命令
redis-benchmark //redis性能测试工具
redis-check-aof //AOF文件修复工具
redis-check-rdb //RDB文件修复工具
redis-cli //redis命令行客户端
redis.conf //redis配置文件
redis-sentinal //redis集群管理工具
redis-server //redis服务进程
```

## 前台启动Redis

在bin目录下运行`redis-server`命令，即可启动Redis，这种是前台启动，缺点是当窗口关闭，则停止Redis，一般我们会使用后台启动

```
//
./redis-server
```

## 后台启动Redis

后台启动需要修改Redis的`redis.conf`配置文件，找到`daemonize`，设置为yes，`:wq`保存并退出（默认为no）

```
vim /usr/local/redis/bin/redis.conf
//找到daemonize，设置为yes
daemonize yes
```

还需要启动Redis时，指定该配置文件进行启动

```
//切换到Redis安装目录
cd /usr/local/redis
//指定配置文件进行启动
./bin/redis-server ./redis.conf
```

## 连接Redis

bin目录下，有一个`redis-cli`，通过它可以连接Redis进行操作

```
/usr/local/redis/bin/redis-cli
```

## 关闭Redis

关闭Redis，则使用使用`redis-cli`，发送`SHUTDOWN`指令进行关闭

```
cd /usr/local/redis
./bin/redis-cli shutdown
```

也可以强制关闭Redis，但这样强制终止Redis进程，可能会导致`Redis数据丢失`，不推荐这样做，应该像上面通过`redis-cli`进行停止

```
pkill redis-server
```

## 开机启动Redis

编辑`/etc/rc.local`文件，添加启动命令和配置文件路径

```
//编辑文件
vim /etc/rc.local
//添加启动命令和配置文件路径，:wq退出并保存
/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis-conf
```