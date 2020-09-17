# Linux下搭建MQTT环境

Linux版本：centos7

## 安装依赖

```
yum install gcc-c++
yum install cmake
yum install openssl-devel
```

## 下载MQTT安装包，并解压

新建一个`software`目录，下载`mosquitto`安装包，解压，解压后，先不安装，先安装3个拓展插件

```
mkdir software
cd software
wget http://mosquitto.org/files/source/mosquitto-1.4.10.tar.gz
tar -xzvf mosquitto-1.4.10.tar.gz
```

## 安装3个拓展插件

### 安装`c-areas`，支持异步DNS查找的库

```
wget http://c-ares.haxx.se/download/c-ares-1.10.0.tar.gz
tar xvf c-ares-1.10.0.tar.gz
cd c-ares-1.10.0
./configure
make
sudo make install
```
### 安装`lib-uuid`，（支持为每个连接客户端生成唯一uuid）

```
yum install libuuid-devel
```

### 安装`libwebsockets`，（支持需使用websocket的应用）

```
wget https://github.com/warmcat/libwebsockets/archive/v1.3-chrome37-firefox30.tar.gz
tar zxvf v1.3-chrome37-firefox30.tar.gz
cd libwebsockets-1.3-chrome37-firefox30
mkdir build
cd build
cmake .. -DLIB_SUFFIX=64
make install
```

## 修改mosquitto配置文件

进入到刚才解压的`mosquitto`目录，将WITH_SRV:=yes和WITH_UUID:=yes，都用#号注释掉，没有的话，就不用注释了

```
cd mosquitto-1.4.10
vim config.mk
```

## 编译安装

```
make
sudo make install
```

## 启动和测试

### 添加用户和用户组

```
sudo groupadd mosquitto
sudo useradd -g mosquitto mosquitto
```

### 配置mosquitto

```
mv /etc/mosquitto/mosquitto.conf.example /etc/mosquitto/mosquitto.conf
```

### 启动

启动后，默认端口为`1883`

```
mosquitto -c /etc/mosquitto/mosquitto.conf -d
```

### 测试

先订阅一个Topic，例如`hello`

```
mosquitto_sub -t hello
```

打开另外一个窗口，发布Topic消息

```
mosquitto_pub -t hello -h localhost -m "hello world"
```

## 其他问题

启动时，如果提示找不到`libmosquitto.so.1`，则使用以下命令修复

```
sudo ln -s /usr/local/lib/libmosquitto.so.1 /usr/lib/libmosquitto.so.1
sudo ldconfig
```

如果是其他so文件不找不到，也是改动这句话，例如找不到`xxx.so.2`，则改成再运行即可

````
sudo ln -s /usr/local/lib/xxx.so.2 /usr/lib/xxx.so.2
sudo ldconfig
````