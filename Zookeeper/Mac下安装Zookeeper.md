# Mac下安装Zookeeper

[Zookeeper官方](http://zookeeper.apache.org/)

[Zookeeper下载地址](http://zookeeper.apache.org/releases.html#download)，我选择3.4.14版本

## 配置环境变量

解压压缩包到指定目录，我选择放在User目录下

```
export ZOOKEEPER_HOME=/Users/wally/zookeeper-3.4.14
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

- 更新环境变量

```
source .bash_profile
```

## 配置Zookeeper

- 配置Zookeeper的配置文件

进入Zookeeper的目录，找到conf子目录，默认会有一个配置文件模板zoo_sample.cfg，复制一份，重命名为zoo.cfg。


1. 编辑zoo.cfg，将dataDir属性的值改为../data，就是将数据目录配置到了Zookeeper根目录下的data目录。
2. 在Zookeeper根目录下新建一个data目录，作为Zookeeper的数据存放目录。

- clientPort，为Zookeeper要使用的端口号

```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=../data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

## Zookeeper常用命令

去到bin目录，终端下使用zkServer.sh

```
//启动
zkServer.sh start
//停止
zkServer.sh stop
//查看状态
zkServer.sh status
```

使用Zookeeper的客户端cli连接Zookeeper

```
zkCli.sh  -server localhost
```